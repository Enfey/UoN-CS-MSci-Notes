
### Component overview 
- 2 main components, and 1 planned out
- All planned out according to my own notes, with inspiration/implementations that are derived elsewhere cited in the comments. 
- Features an AArch64 + ARM32 mmap-based ELF parser as both a library and CLI tool, named Lens
- Features a static linker (meld) accepting ET_REL objects and static archives to produce statically linked executables (both ET_DYN and ET_EXEC), able to link against musl, a standard library implementation, supports RELRO, symbol versioning with versioning scripts, many synthetic symbols, .got, .got.plt, .plt, .dynamic, dynamic reloc generation, relaxation for MOVT* relocations, archive support, essentially everything other than debug support, thumb support, and TLS support is provided in this static linker. 
- Features the concepts for a userspace loader which could theoretically unmap the majority of itself from memory; i discussed this prior and the challenges of such an implementation (which i am not sure exists), but did not get round to an implementation unfortunately.
### Capabilities
- Focusing on meld:
	- Musl libc linking
		- Crt Objects(which expect a number of synthetic symbols) + archive extraction
	- Symbol resolution
		- GST hash table with the regular strong/weak semantics
	- Archive support
		- Fixpoint intra-archive extraction and group semantics for inter-archive circular dependencies
	- ARM32 Relocations
		- 15+ types, focusing entirely on the core subset that appear, handling all static relocs, and emitting dynamic relocs e.g., for .got.plt entries.
	- Relaxation
		- MOVT becomes a NOP when the upper 16 bits are zero
	- Long branch veneers
		- Derive intermediate target, patch at original reloc where overflow happened should reloc type have this enabled, and then jump to veneer which is in range which will load the full address.
	- GOT/PLT generation
	- .dynamic generation
		- All required DT_* tags with some optional included e.g., .gnu.hash as this is synthesised
	- GNU hash
		- Bloom filter + reordering of dynsym according to hash values to support memory locality.
	- Symbol versioning
		- .gnu.version_d, .gnu.version with version script parsing
	- RELRO
		- Partial and Full, though afaik passing partial as an option is more or less redundant in the long run because musl's dynamic linker will resolve the PLT at startup and ignore DT_BINDNOW entirely.

### Tests
Run tests, show the tests, prepare some commands and test files additionally?
### Lens
- Zero-copy, as in we mmap the elf file and point directly into that memory, subsequent code just maintains the base of the elf file it refers to, rather than allocating buffers and copying from the kernel's page cache.
	- File has to stay mapped for the lifetime of handle, but this is fine; pages are obviously faulted in when needed.
- We maintain an IR for elf files, with a discriminated union(as a library, lens only cares for e32, accessors are quite branch heavy, dispatching internally, for word specific object content, but the branch predictor learns the union type quickly).
	- Contains base, size, file path, class, type, machine, pointers to key structures e.g., strtab, shstrtab, and store info on the structure derived at parse-time for easy access e.g., shnum, phnum, symtab count
- The reason for internal dispatch is due to no function overloading.
- Two main open functions
	- elf_open(path) - lens owns the memory, munmap on close, opens .o files on disk
	- elf_open_mem(data, size) - caller owns the memory, used for parsing already mmap'd archive members.
	- Both dispatch to elf_parse_internal with a flag that just determines whether the memory should be unmapped on error.
- Defensive programming occurs, e.g., bounds checking before dereferencing, overflow checking before multiplication via __built_in_mul_overflow, prevent ELF file poisoning from breaking the linker
- Functions return error codes; there is a global SET_ERR macro functions should use to attach an error message and error code (which is an enum) to the context structure e.g., elf, meld_ctx. Functions then also return this error code, if failed, then we can determine display the attached info on the context object to the user.
- Lots of pretty printing functions for use as a CLI tool rather than a library, supports AArch64 and ARM32, generally analogous to readelf -a, and supports all common relocation types, not types, section types etc as according to the processor supplement. 


## Meld
### Input description
- Deals with input objects; either .o files or archives. 
- Key IR
	- meld_input
		- Maintains a lens handle to elf structure
		- Array of meld_sec_state
		- Pointer to parsed relocations
		- Symbol mapping of length symtab_count
			- **sym_map** array of pointers to definitions (meld_symbol_t) for symbols within that input file; the pointer may be to the locals array within that input IR, or it may be a pointer to a GST entry, depending on the symbol semantics.
		- Maintains a pointer to the parent archive(and offset of where this member appears), arguably wasteful as these 8 bytes + alignment included on all input structures, should've been made into a union with a single bit discriminator to save some bytes.
	- meld_sec_state
		- Small structure, one per section for that object, describing the output section that input section coalesces into, and its offset within that output section (set later).

#### Input implementation
- Input_create()
	- Synthesise input IR, given path to input object, utilise lens for validation, calloc per-section state, calloc sym map, return input
- Input_create_from_archive()
	- Synthesise input IR, given base address of some archive, using elf_open_mem, set ar pointer and member off, done in archive portion, rest is the same.
- Input_add_local
	- Helper; given sym, just add to input local array (first appearance of dynamic array realloc scheme, cap increased by 2, begins at 32, and increases). Maintain O(1) for end insertion
- Input_parse_symbols
	- Iterate over .symtab for an input object, and instantiate meld_symbol_t entries, storing locals in inp->locals, and inserting globals into GST, setting symmap accordingly. Utilises some functions from meld_symbol to do this. In the case of sections, special, instantiate local value, update st_value later when layout performed, and give no name, and mark flag as section symbol, and add to local. General case, match on symbol binding, either add to local and set symmap pointer for index i, or add to GST and set symmap point for index i to the gst entry. Symbols destroyed on error, for GST, if incoming symbol was merged in, then we destroy it here.
- Input_update_symbol_values
	- Utilises sec_state, iterate symmap, deal with symbols just from this input, take shndx of symbol, determine the sec_state for that section, and use sections output address + sym_st_value which is relative to that section at input time, to divine the final full address. 
### Symbols/GST
- Symbol IR
	- Symbol name
	- Name hash (FNV-1a, fast, strong, found from just googling, and has good avalanche characteristics) 
	- Stores all ELF symbol contents
	- State structure, which marks undef, def, or defined in shared library, designating runtime resolution
	- GOT offset, PLT_offset
	- Veneer_addr, should symbol be unreachable
		- Can use this to make original branch retarget the veneer_addr instead of patching ST_VALUE at reloc processing time. 
	- Dynsym index, 0 = not in. 
	- Defining input file
	- Archive containing symbol
	- SONAME if from shared library.
		- Assists in determining whether need runtime resolution.
	- Pointer to resolving symbol
		- GST entry, i think i added this based on my original design, and then never used it as symmap was more intuitive.
	- Pointer to next symbol(if inserted into GST)
#### Symbols/GST implementation
- GST implementation
	- Hash table with FNV-1a hash with separate chaining(simpler deletion) via pointer on symbol structure.
		- FNV-1a, takes string, offset basis, and XOR byte of string into low 8 bits, and multiplies edited offset basis with prime to propagate the effect of the byte upward.
	- Support for rehashing; calloc new buckets x2, and remod(bucket count = power of 2, can do fast mod via & count -1) name hashes with new_count to yield new bucket. Free old buckets.
- Symbol resolution + GST insertion
	- gst_insert(gst, symbol, result symbol)
		- Collision
			- Conflict resolution, strong vs weak semantics, prefer local def over runtime provided one. Defined cannot be overriden(unless original def was weak), defined override shared, undef overriden by anything except undef.
			- If winner was incoming symbol, then preserve next in chain, memcpy fields, restore the chain, and then transfer ownership.
		- No collision
			- Head insertion into designated bucket, increment symbol count, and undef count, if sym_state is undef.
		- Set output
- GST iterate
	- Takes a function pointer(has symbol pointer and additional info as args) and some additional info, applies function to each GST entry.
- Symbol from elf + symbol create shared
	- Utility, one creates sym IR from elf file, just extract + pack attributes, set some defaults, set hash, set input object, set state based on st_shndx.
	- Create shared does same, sets all defaults, but sets SHN_UNDEF, strdup's soname, and state is SYM_SHARED.
- Some utils for debugging, dumping GST by printing symbol contents. 
### Archive support 
- Archive IR
	- AR_hdr, as ordained in ar format, only member name, size, and fmag(`\n) used.
	- ar_index_entry - name, name hash, member offset, used to create IR for symbol table.
	- format enum, didn't add bsd AR support, differ in terms of symtab member semantics(gnu always bigend, bsd depend on host endianness on which ar was created).
	- meld_archive
		- base, size, group_id for group semantics, long names table ptr, ar_index_entry array for symtab representation,  extraction tracking
			- Extraction tracking, bool array, member_offsets array, extracted[i] true if member already extracted.
- Group semantics supported

#### Archive implementation
- Utils to get member name from header, parse decimal, and get member name, both from long file name table, or / terminated.
- Utility to parse gnu_symtab
		- Must do bit packing to little-endian, calloc symbols array (symtab on ar), grab offset, name, and hash name, and advance.
- archive_open()
	- equivalent to elf_open for archives, just ensure valid format, mmap archive contents (justifying elf_open_mem), set ar attributes incl callocing dynamic attributes, parsing symtab, and finding long names table and setting attributes for that. 
- search archive range
	- Search range of archives given indexes (comes from overarching link context) for symbols that resolve undefs.
	- Collect all undefs from gst, make gst_iter_fn and a small IR for this which just holds undef symbols dynamic array
	- For each undef, search archives in range by looking up hash against ar symtab, and then do direct name comparison on match, then extract member that defines according to member offset.
	- For grouped, continue checker other archives, for ungrouped, break on first match.
	- Return how many inputs extracted.
- archive_resolve
	- Main func, fix point iteration. --start / --end-group.
	- Iterate all archives, if no group, then just iterate until fixpoint via search_archive_range(extracted > 0)
	- If group, then find extent of the group, search that range until fixpoint. 
### Sections 
- Term osec as output section that input sections coalesce into, referred to in input sec_state array
- IR
	- OSEC flags - derived from SHF_* flags, only subset handled, OSEC synthetic introduced, represents secs gen by linker.
	- meld_isec
		- Minimal, just stores input IR pointer, shndx, output_off, and pointer to next isec.
	- meld_osec
		- Result of coalescing
			- name
			- name_off (shstrtab)
			- shndx
			- flags
			- general attributes e.g., file off, vaddr, size, align, 
			- isecs, isecs tail for o(1) tail insertion
			- next osec pointer.
	- meld_sec_mgr
		- osec linked list + tail pointer, top level IR.
#### Implementation
- -ffunction-sections/-fdata-section compiler flag yields .text.foo, .text.bar, we elect to merge into text for simplicity and respecting the semantics taught, couldve maybe added tool to discard unused .text.* sections
	- Have array of section name bases, with precedence, with strict or non strict matching, given section name, can determine the section name we need to coalesce into from the name bases by matching either on prefix, or matching strictly. 
- find/createosec
	- Search based on name and flags, if match, then return
	- Otherwise instantiate with defaults, alignment = 1, lowest, as can only be raised(will depend on isecs), and do tail insertion.
- add_input
	- Adds input to appropriate output section, based on base_name and flags.
	- Calloc isec, add the minor attributes of shndx and input, and do tail insertion for that osec's isec's array. Also, update alignment. 
- Some helper functions
- Finalise structure
	- Needed to finalise section header fields e.g., sh_link sh_info, sh_entsize for example, after assigning indices or syntheising those sections, done later on. 
- Section_add_synthetic
	- Has synthetic flag, passed in type, etc. Has no isecs, and added to mgr osecs via tail insertion. 
- section layout
	- For each osec, iterate input sections, assign internal offsets via alignment and size for that isec(offsets start at zero).
	- Also set sec_state output offset for that inputs adjacent section to contain that offset.
	- Set size for osec, based on final offset.
### Relocs 
- Has own error enum, as many things can go wrong
- All info came from processor supplement (besides veneer generation, practically no info on this, gave it a go, but have no way to verify that it will work, but the semantics look mostly correct to myself.)
- meld_reloc
	- offset, sym_idx, sec_idx, addend, type
- I will discuss veneer implementation but it will not be practical to verify they work.
	- veneer_entry
		- maintains target symbol, veneer addr which becomes intermediary jump target, and offset within the veneer section, which is appended immediately after text. stored as linked list, and there is buffer in a mgr structure so can output this content after text. 
		- If overflow when patching address in and reloc type suitable for veneer, then create veneer_addr, and apply the same relocation to P, but with S being the veneer address instead. Add to veneer mgr (which also generates the veneer in the buffer, with respect to the target symbol, patched into literal pool that is loaded PC-relative without padding).
- reloc_parse_input
- reloc_apply_input
- relaxation
### Dynlink support
### Output
### Layout
### Synthetic 
### Link



