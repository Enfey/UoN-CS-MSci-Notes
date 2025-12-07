# Shared libraries and Dynamic Linking
> A shared library (often called a Dynamic Link Library or Dynamic Shared Object) is a PIE executable that is loaded and linked at runtime, intended to be shared amongst multiple programs simultaneously, rather than being copied into the executable at static-link time. 

Shared libraries necessitate `PIE`s; for ELF this amounts to an `ET_DYN` object, the reason for this is that it ensures the library can be mapped into the address space at any address without requiring drastic load-time relocation. 

Shared libraries further export symbols that are resolved to their final memory addresses at runtime, sometimes lazily at runtime via `.plt` for function calls, and generally via `.got`. This resolution mechanism is known as **dynamic linking**, and is expanded upon in [[Chapter 10]].

The primary benefit acquired from the use of shared libraries is memory optimisation; whereby the kernel guarantees that the text segment resides in one shared copy on physical RAM. Writable data segments are mapped to processes **Copy-on-Write**(CoW) to preserve separation of process states.
### Binding Time Issues
Shared libraries introduce unique issues that don't exist with fully static, conventionally linked executables.

The most basic error is a missing library. Since the code isn't inside the executable, if the OS cannot find the library at runtime, the program crashes immediately due to failing resolution. The library must be present and compatible when the application runs.

A more subtle issue arises when the library is present but has changed since the program was originally linked. With traditional static linking, the program embeds the library code and resolves symbol addresses at build time, so later changes to these libraries don't matter. But with shared libraries, the program depends on library code that lives outside the executable and is loaded at runtime. Early systems used 'static shared libraries', where symbol addresses were still baked into the executable. Even small structural changes in the library would inevitably shift addresses, meaning addresses could be invalidated. Only tiny updates were permitted, and anything larger either required relinking, or keeping multiple versions of the library installed.

Modern platforms have mostly solves this issue. Shared libraries are loaded at runtime, and symbol addresses are resolved entirely at runtime via the `.got` and `.plt`. This allows much more freedom for updating libraries as references are symbolic and not absolute and can easily be rectified at runtime.
### Address space management
From now, we no longer refer to 'static shared libraries'. The advent of PIE executables and [ASLR](ASLR.md) nullified many of the issues surrounding address space management for shared libraries. Between the formers and the paging system, one is able to load a shared library at an arbitrary address, where the loader and the runtime linker are responsible for handling any relocation automatically. This generally amounts to assigning a random load-base and filling in the `.got` and lazily (or eagerly) filling in the `.plt` for a given shared object. This is relatively mechanical. There is some small runtime overhead for this scheme as discussed in the [[Chapter 8|prior chapter]].


### Library and Symbol Versioning
Versioning in libraries is still prevalent. On old UNIX systems, names were often formulated like so:
```C
libc_s.4.2.2
```
- **4** $\to$ major version: ABI incompatible changes, programs liked with 4.x cannot run with 3.x
- **2** $\to$ minor version: new features, backwards-compatible
- **2** $\to$ patch: bug fixes, security patches; fully backwards-compatible
The linker searched directories for the best match, attempting to find the most compatible version. 

Modern shared libraries essentially have three names.
1. **soname**
	This follows the naming scheme:
		$lib<library name>.so.<version number>.$ 
	This encodes the ABI version, and is used to determine compatibility.
2. **real name**
	Provides the full file name and the exact name of the shared object, including the path. 
3. **link-name**
	The soname without any version numbering.
The processes of linking against such libraries and ensuring correct versioning are detailed more in depth in [[Chapter 10]].

On ELF-based systems, a shared library can attach a version tag to each exported symbol. All ELF objects may provide or depend on versioned symbols. This allows incompatible changes to be made without increasing the major version number of the library at a minimum and at the extreme, symbol versioning can completely replace traditional naming schemes using major and minor version numbers. Symbol versioning is implemented by 3 section types. Internally, this results in the creation of a number of special ELF sections:
- `.gnu.version` 
	Has a section type of `SHT_GNU_versym` and shall contain the **Symbol Version Table.** This section shall have the same number of entries as the Dynamic Symbol Table in `.dynsym`. This section contains an array of elements of type `Elfxx_Half`. Each `Elfxx_Half` entry  specifies the version defined for or required by the corresponding symbol in the Dynamic Symbol Table. The values in the Symbol Version Table are specified to the object in which they are located. These values are identifiers that are provided by the `vna_other` member of the `Elfxx_Vernaux` structure or the `vd_ndx` member of the `Elfxx_Verdef` structure. The values 0 and 1 are reserved. 
	- 0 - local symbol, not visible outside the object
	- 1 - global symbol defined in this object
	- Other values - reference a version string defined elsewhere in one of the other Symbol Version sections. The value itself is not the version associated with the symbol. 
- `.gnu.version_d`
	Has a section type of `SHT_GNU_verdef` and shall contain **symbol version definitions**. The number of entries in this section shall be contained in the `DT_VERDEFNUM` entry of the `.dynamic` section. The `sh_link` member of the section header shall point to the section that contains the strings referenced by this section. This section contains an array of `Elfxx_Verdef` structures, optionally followed by an array of `Elfxx_Verdaux` structures. Section describes version namespaces exported, mostly used by shared libs being linked against.
  ```C
	typedef struct {
		Elfxx_Half    vd_version;
		Elfxx_Half    vd_flags;
		Elfxx_Half    vd_ndx;
		Elfxx_Half    vd_cnt;
		Elfxx_Word    vd_hash;
		Elfxx_Word    vd_aux;
		Elfxx_Word    vd_next;
	} Elfxx_Verdef;
  	```
	`vd_version` 
		Version revision. This field shall be set to `1`.
	`vd_flags
		Version information flag bit mask.
	`vd_ndx`
		Index used by entries in .gnu.version to refer to this version definition. This is how a symbol is tagged with this version.
	`vd_cnt`
		Number of associated verdaux array entries
	`vd_hash`
		Hash of the version name string
	`vd_aux`
		Offset in bytes to a corresponding entry in an array of `Elfxx_Verdaux` structures.
	`vd_next`
		Offset to the next `Verdef` entry, in bytes
  ```C
	typedef struct {
		Elfxx_Word    vda_name;
		Elfxx_Word    vda_next;
	} Elfxx_Verdaux;
  	```
	`vda_name`
		Offset into `.dynstr` which contains the version name string
	`vda_next`
		Offset to next aux entry, in bytes.
- `.gnu.version_r`
	Has section type of `SHT_GNU_verneed` shall contain **required symbol version definitions from other shared libraries**. The number of entries in this section shall be contained in the `DT_VERNEEDNUM` entry of  `.dynamic`. The `sh_link` member of the section header shall point to the section that contains the strings referenced by this section.  The section shall contain an array of `Elfxx_Verneed` structures, optionally followed by an array of `Elfxx_Vernaux` structures. Used by both consumer objects and shared libs.
	```C
	typedef struct {
		Elfxx_Word    vna_hash;
		Elfxx_Half    vna_flags;
		Elfxx_Half    vna_other;
		Elfxx_Word    vna_name;
		Elfxx_Word    vna_next;
	} Elfxx_Vernaux;
	```
	`vna_hash`
		Dependency name hash value
	`vna_flags`
		Dependency information flag bitmask.
	`vna_other`
		Object file version identifier used in the `.gnu.version` symbol version array.
	`vna_name`
		Offset to the dependency name string, in bytes
	`vna_next`
		Offset to the next `Elfxx_Vernaux` entry.
#### Startup sequence
When loading a shareable object, the system shall analyse version definition data from the loaded object to assure that it meets the versions requirements of the calling object. The step is referred to as **definition testing**. The dynamic loader shall retrieve the entries in the caller's `Elfxx_Verneed` array, and attempt to find matching definition information in the loaded `Elfxx_Verdef` table.

Each object and dependency shall be tested in turn. If a symbol definition is missing and the `vna_flag` bit for `VER_FLG_WEAK` is not set, the loader shall return an error and exit. If the `vna_flags` bit for `VER_FLG_WEAK` is set, the loader shall issue a warning and continue operation.
#### Symbol resolution
Where symbol versioning is active, symbol resolution becomes stricter. 
Definition testing is thus extended beyond a simple match of symbol name strings; the version of the reference must also equal the name of the definition. A number of steps are taken to achieve this. 

The static linker constructs the `.gnu.version` and `.gnu.version_r` and entailing structures when building the consumer object and seeing a required versioned symbol from a shared/static library. If the consumer object further exports versioned symbols, then the `gnu.version_d` is instantiated, with entailing structures.

The linker uses the same index from `.dynsym`, parallel to `SHT_GNU_versym` (`.gnu.version`). This index permits the acquirement of name data via `.dynstr`, and acquirement of the version requirement string from `vn_name` (offset) within the `Elfxx_Verneed` array, likewise, the corresponding definition string is retrieved from the `Elfxx_Verdef` table (via `Elfxx_Verdaux` `vda_name` offset into `.dynstr` (or `sh_link`)) in the library being linked against. 

Additionally, if the high-order bit (bit 15) of the version symbol is set, in the section `.gnu.version` (containing `Elfxx_Half` array) for that symbol, then the static linker shall ignore the symbol's presence in the object during the build process. Designed to prevent new applications from linking against obsolete versions of a function. 

When an object with a reference and an object with a reference and an object with a definition are being linked, the following rules shall govern the result:

1. The object with reference and the object with the definitions both use versioning. All described matching is processed in this case. A fatal error shall be triggered when no matching definition can be found in the object whose name is one referenced by the `vn_name` element in the `Elfxx_Verneed` array.
2. The object with the reference does not export any versions, while the object with the definitions does. In this instance, the linker attempts to resolve safely by restricting the search to the 'base definition'. These are definitions with index numbers 1 or 2, which represent the default, public face of the definition. 
3. The object with the reference uses versioning, but the object with the definitions specifies none. A matching symbol shall be accepted in this case. A fatal error will occur in a unique case. When the consumer object is constructed, records exact file name of needed library into 
4. Neither the object with the reference nor the object with the definitions use versioning. The behaviour in this instance shall default to pre-existing name resolution rules. 

### Startup sequence for shared libs
When loading a sharable object, version definition data from the loaded object is analyzed to assure that it meets the version requirements of the calling object. The dynamic loader retrieves the entries in the caller's `Elfxx_Verneed` array and attempts to find matching definition information in the loaded `Elfxx_Verdef` table.

Each object and dependency is tested in turn. If a symbol definition is missing, the loader returns an error. A warning is issued instead of a hard error when the `vna_flags` bit for `VER_FLG_WEAK` is set in the `Elfxx_Vernaux` entry.

When the versions referenced by undefined symbols in the loaded object are found, version availability is certified. The test completes without error and the object is made available.
### Shared library structure and Creation
`ET_DYN` is the `e_type` specified for **shared objects** (`.so`) and for **position-independent executables** (PIEs). An `ET_DYN` file is *relocatable at load time* - the loader maps its `PT_LOAD` segments to some arbitrary base address and applies relocations specified in the `rel.dyn` and `rela.dyn` sections, as emitted by the static linker. The typical static linker strategy is to link the object to load at address zero, emit relocations that the runtime loader will apply, set up the dynamic tables (`.dynamic/.dynsym/.dynstr.rel/a.*, .gnu.hash or .hash, .got, .got.plt, .plt` etc). At runtime, ASLR and its entropy determine the actual base of the `ET_DYN` object. 

#### Linking View
Generally, the following sections are generally created. Certain sections will be elaborated on more in [[Chapter 10]] and beyond, otherwise, we are already familiar. $Arm$ and other manufacturers often supply their own sections, some of which relate to dynamic linking and shared libraries. See [ARM specifics for linking](ARM%20specifics%20for%20linking.md) for more information. 
- `.text`
- `.rodata`
- `.bss`
- `.dynsym` and `.dynstr`
	These form the symbol database the dynamic linker uses to resolve symbols at runtime. `.dynsym` is a compact symbol table for dynamic linking and `.dynstr` holds symbol names referenced by `.dynsym`'s `st_name` field.
	
	The static linker is responsible for collecting exported symbols, these are symbols with `STB_GLOBAL` or `STB_WEAK` binding and `STV_DEFAULT` or `STV_PROTECTED`(non-interposable) visibility, which are needed by other DSOs or a consumer and are placed into `.dynsym`. Much of this is accomplished in generic static-link pipelines via the aggregation of symbols into a '*Global Symbol Table*'. The linker is further responsible for deciding an ordering of symbols, as GNU hash expects symbols sorted for the chain part. `st_name` is pointed to `.dynstr` and `st_value` is set to the virtual address relative to the object base, or 0 for undefined symbols. The `st_info` and `st_other` fields are set accordingly.
	
	We do not have the restriction of static-link only pipelines whereby a symbol with binding `STB_GLOBAL` without a corresponding definition would trigger a link-time error - DSOs expect many of their symbols to be resolved at runtime from other DSOs, so it is perfectly fine for `.dynsym` to contain undefined symbols. A relocation entry is simply emitted for it, where this will be a `.got/.got.plt` relocation. 
- `.rel/a.dyn`
	Contains the catenated general dynamic relocations. That is those that are data relocations, GOT entry relocations, TLS relocations, and all those not pertaining to dynamic PLT relocations; those have their own section prescribed.
- `.rel/a.plt`
	Contains the relocations associated solely with the procedure linkage table. That is, the relocations that point to .got.plt entries. The reason for this separation is that the dynamic linker can ignore them during process initialisation if lazy binding is enabled. If eager binding is enabled, then `.rel/a.dyn` and `.rel/a.plt` may merge. The static linker is responsible for instantiating this table by inspecting relocation types, and adding relocation entries to this table which relate to the `PLT`, on ARM this is `R_ARM_JUMP_SLOT` for dynamic relocations. 
	
	I am led to believe that assemblers only emit primitive relocations that the linker then has to 'promote/specialise' when it has more information e.g., cannot find matching function in static libs and other objects, must be provided by shared lib, so emits `PLT` relocation at the offset the function appears in, and adds that relocation to the `.rel/a.plt` section and adds auxilary information in the `.dynamic` section. 
- `.plt`
	The static linker is responsible for constructing a PLT area. PLT0 is emitted; the resolver entry that will eventually point to the dynamic linker's resolution routine entry point. This will be relocated, as `PLT_0` points to a `.got.plt` entry. `PLT_0` implementation is architecture specific but always present. We emit one `PLT` entry per external function needing lazy binding; that is stub code that jumps indirectly using its `.got.plt` slot. The stub is position-independent and typically identical across similar symbols except for the `.got.plt` slot referenced. We emit relocations, one per `PLT/GOT` entry. Here, this relocation targets the `.got.plt` slot. On $ARM$, this relocation type is typically `R_ARM_JUMP_SLOT`. We emit `.dynsym`/`.dynstr` entries for referenced symbols, and write the corresponding `DT_JMPREL`, `DT_PLTRELSZ`, `DT_PLTREL` entries in `.dynamic` so the loader knows where `.rel/a.plt` lives, its size, and the type of relocations it holds. It should be noted that when the linker sees a PLT generating relocation, it checks whether the referenced symbol is pre-emptible/interposable, if it is not, then the relocation can be resolved at link-time, and there will be no PLT entry for that symbol. It should further be noted that entries `.got.plt[1]` and `.got.plt[2]` are reserved, for a pointer to `link_map` and a base/offset for computing relocation index, respectively.
	
	Because of the 1-1 mapping between `.plt` and `.got.plt`,the relocation index is known and can be embedded directly in the `PLT` stub to refer to the correct relocation entry in `.rel/a.plt`. This is embedded either as an immediate constant or loaded into a register, such that when control passes from `.plt` -> `.got.plt` -> `.plt[0]` -> `resolver` that the correct symbol can have its address resolved and placed into the corresponding .`got.plt` entry.
	As mentioned in a previous chapter, the interpretation of `.plt` is architecture-specific. See [ARM specifics for linking](ARM%20specifics%20for%20linking.md) for more information. 
- `.got` (and `.got.plt`)
	During the link-step, all input relocations are processed. The `.got` creation proceeds as follows:
	1. **Collect `.got`-related relocations**
		These include:
		- **R_ARM_GOT_PREL** - compute pc-relative offset of a symbol's `.got` entry, $.GOT_{sym} + A - P$ 
		- **R_ARM_GOT32** - rewrite word to contain absolute virtual address of `.got` slot for $sym$, $.GOT_{sym} + A$
		- **R_ARM_GOTPC** - compute the pc-relative offset of the `.got` base address itself $.GOT_{base} + A - P$ 
		or their architectural equivalents. Each relocation indicates 'this instruction expects symbol $S$ to be accessed via the `.got`.
	2. **Allocate `.got` entries**
		The linker creates the .`.got` section and assigns each unique symbol referenced through the `.got` its own slot. The slot holds a pointer which the loader will fill with the symbol's runtime address. ARM uses separate relocations for `.got`-relative addresses (`.got`OFF) vs actual `.got` entries.
	3. **Emit dynamic relocation entries**
		Generate relocation entries in the output binary's dynamic relocation sections. For every unique external symbol that received a `.got` slot, the static linker performs the following:
		1. **Select relocation type**
			For `.got` entries that hold the absolute address of a data symbol, this is typically **R_ARM_GLOB_DAT**. For entries related to function calls via the PLT, its typically **R_ARM_JUMP_SLOT**, and this entry will belong in a runtime writable subsection called `.got.plt`
		2. **Select offset**
			The linker records the virtual address of the newly allocated `.got` slot as the `r_offset` field in the dynamic relocation entry to indicate where the final address should be written. 
		3. **Set the symbol index**
			Sets the symbol index of the symbol $S$ into the `r_info` field to tell what address to look up. 
- `.hash` or `.gnu.hash`
	A hash table of Elf32_Word objects exists to support *dynamic symbol lookup*. Doing a plain linear search over potentially thousands of symbols, especially for relocation resolution at load-time, would be slow. A hash table is constructed at static link-time and stored in a serialised format within the object to aid lookup. A loader uses one of the aforementioned, or can use both for compatibility.
	`.hash`
		Sequence of 32 bit words.
		`u32 nbucket` - number of buckets (chosen by linker, heuristic).
		`u32 nchain` - number of chain entries (equal to number of entries in `.dynsym`).
		`u32 buckets[nbucket]` - entry contains symbol index of first symbol in that bucket, if 0 lookup fails. 
		`u32 chain[nchain]` - entry contains, for each symbol index $i$, $chains[i]$ = next symbol index in same bucket, linked-list.
			1. **The [ELF hash function](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.dynamic.html#hash) generates an integer**
				`x = elf_hash("sym_name");
			2. **Compute bucket index**
				`bucket_index = [x % nbucket];
			3. **Lookup** 
				`y = buckets[bucket_index]`
				`y` is an index into `.dynsym` **and** the `chain[nchain]` array. Should `y == STN_UNDEF(0)`, the bucket is empty. If the bucket does not contain desired result, follow chains array until match is found.
		Limitations such as poor performance on "false" lookups i.e., lookups where the symbol does not exist requires a linear traversal of the chain for that bucket until the end is hit motivated the creation of a more modern serialised format to aid symbol lookup. 
	`.gnu.hash`
		Section with type `SHT_GNU_HASH` containing a more efficient hash table format than `.hash`, referenced by `DT_GNU_HASH`. One of the most prevalent augmentations is the inclusion of a [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter), a space-efficient probabilistic data structure that is used to test set membership. It can be used to guarantee that a symbol is not present in a file. This is used prior to hash lookup to determine if the symbol is found in the object or not. `.gnu.hash` further stores hash values (or at least metadata derived from them) alongside bucket/chain entries to avoid redundant string comparisons where possible. Also, `.dynsym`is sorted into hash order, such that memory access tends to be adjacent and monotonically increasing to exploit memory locality for improved cache behaviour. For formatting reasons and the slight complexity, the discussion of the algorithms surrounding `.gnu.hash` will be deferred to [GNU Hash](GNU%20Hash.md).
- `.dynamic`  
  We will discuss this here as it is extremely relevant.  
  If an object file participates in dynamic linking, then it will contain the `.dynamic`, and as a ready `.so` its program header table will contain the `.dynamic` section in a segment with type `PT_DYNAMIC`. A synthetic symbol `_DYNAMIC` is emitted by the static linker to label the section, whose value is the virtual address of `.dynamic` relative to the DSO's link-time base, relocating via the load-time delta at load-time. The `.dynamic` section/segment is used almost entirely by the dynamic linker. The section contains an array of the following structures:
    ```c
    typedef struct {
        Elf32_Sword d_tag;
        union {
            Elf32_Word d_val;
            Elf32_Addr d_ptr;
        } d_un;
    } Elf32_Dyn;

    extern Elf32_Dyn _DYNAMIC[];
    ```
  For each object with this type, `d_tag` controls the interpretation of `d_un`.  
  `d_val`
		These objects represent integer values with various interpretations.
	`d_ptr`
		These objects represent program virtual addresses. The dynamic linker computes actual addresses based on the original file value and the memory base address. For consistency, files do not contain relocation entries to correct addresses in the dynamic structure.
	
	For `dt_tag`. If a tag is marked mandatory, the dynamic linking array for an ABI-conforming file must have an entry of that type.
	![](Pasted%20image%2020251206223514.png)
	`DT_NULL`
		An entry with a `DT_NULL` tag marks the end of the `_DYNAMIC`array. 
	`DT_STRSZ`, `DT_SYMENT`, `DT_SYMTAB`, `DT_STRTAB`
		These tags describe the dynamic symbol table `.dynsym` and its string table `.dynstr`
		`DT_SYMTAB` 
			An entry with a `DT_SYMTAB` tag holds the address of `.dynsym` after it has been laid out. 
		`DT_SYMENT`
			An entry with a `DT_SYMENT` tag holds the size, in bytes, of a symbol table entry. Linker sets it to `sizeof(Elf32/64_Sym)`
		`DT_STRTAB`
			An entry with the `DT_STRTAB` tag holds the address of `.dynstr`, this holds symbol names, library names, and other strings the dynamic linker may need.
		`DT_STRSZ`
			An entry with the `DT_STRSZ` tag holds the size, in bytes, of `.dynstr` after it has been laid out.
	`DT_NEEDED`
		An entry with the `DT_NEEDED` tag holds the string table offset of a null-terminated string, yielding the name of a needed shared library. The offset is an index into the table recorded in the `DT_STRTAB` code. The dynamic array may contain multiple entries with this type. These entries' relative order is significant, though their relation to entries of other types is not. More details are described [further on](#Shared%20Object%20Dependencies).
	`DT_SONAME`
		An entry with the `DT_SONAME` tag holds the `.dynstr` offset of a null-terminated string, giving the name of the shared object. The offset is an index into whatever table is recorded in the `DT_STRTAB` entry. 
	`DT_RUNPATH`
		An entry with the `DT_RUNPATH` tag holds the string table offset of a NUL-terminated library search path string. The offset is an index into the table recorded in the `DT_STRTAB` entry.
	`DT_RELA`, `DT_RELASZ`, `DT_RELAENT`, `DT_REL`, `DT_RELSZ`, `DT_RELENT`
		These tags describe the general dynamic relocation section
		`.rel/a.dyn`. 
		`DT_RELA`
			An entry with the tag `DT_RELA` holds the address of the dynamic relocation table `.rela.dyn`. An object file may have multiple relocation sections that emit dynamic relocations of type `Elf32_Rela`. When building the relocation table, the link editor catenates those sections to form a single table such that the dynamic linker only sees one table. If this element is present, it must also have the following 2 sections.
		`DT_RELASZ`
			An entry with the tag `DT_RELASZ` holds the total size, in bytes, of the `DT_RELA` relocation table
		`DT_RELAENT` 
			An entry with the tag `DT_RELAENT` holds the size, in bytes, of the `DT_RELAENT` relocation entry.
		`DT_REL`
			An entry with the tag of `DT_REL` holds the address of the dynamic relocation table `.rel.dyn`. It is otherwise similar to `DT_RELA`. If this element is present, it must also have the following 2 sections.
		`DT_RELSZ`
			An entry with the tag `DT_RELSZ` holds the total size, in bytes, of the `DT_REL` relocation table.
		`DT_RELENT`
			An entry with the tag `DT_RELENT` holds the size, in bytes, of 
	`DT_JMPREL`, `DT_PLTRELSZ`, `DT_PLTREL`, `DT_PLTGOT`
		These tags describe the relocation entries associated solely with the procedure linkage table `.rel/a.plt`.
		`DT_JMPREL`
			An entry with the tag `DT_JMPREL` holds the address of relocation entries associated solely with the procedure linkage table. If this entry is present, the related entries of types `DT_PLTRELSZ` and `DT_PLTREL` must also be present. 
		`DT_PLTRELSZ`
			An entry with the tag `DT_PLTRELSZ` holds the total size, in bytes, of the relocation entries associated with the procedure linkage table.
		`DT_PLTREL`
			An entry with the tag `DT_PLTREL` specifies the type of relocation entry to which the procedure linkage table refers. The `d_val` member holds `DT_REL` or `DT_RELA`, as appropriate. All relocations in a procedure linkage table must use the same relocation type. 
		`DT_PLTGOT`
			An entry with the tag `DT_PLTGOT` holds an address associated with the procedure linkage table and/or the global offset table. Interpreted according to architecture.
	`DT_HASH`, `DT_GNU_HASH`
		These tags describe the hash table structures used to support *dynamic symbol lookup*.
		`DT_HASH`
			An entry with the tag `DT_HASH` holds the address of the symbol hash table. This hash table refers to the symbol table referenced by the `DT_SYMTAB` element
		`DT_GNU_HASH`
			An entry with the tag `DT_GNU_HASH` holds the address of the GNU symbol hash table. See [GNU Hash](GNU%20Hash.md) for more details.
	`DT_FLAGS`
		`DF_ORIGIN`
			A flag with the name `DF_ORIGIN`
		`DF_SYMBOLIC`
			If the flag with the name `DF_SYMBOLIC` with value `0x2` is set then the dynamic linker's symbol resolution algorithm for references within the library is changed. Instead of starting a symbol search with the executable file the dynamic linker starts from the shared object itself. If the shared object fails to supply the referenced symbol, the dynamic linker then searches the executable file, and other shared objects as usual.
		`DF_TEXTREL`
			If the flag with the name `DF_TEXTREL` with value `0x4` is set then one or more relocations might request modifications to a non-writable segment; the dynamic linker must prepare accordingly and provide temporary write permissions. This can prevent marking the text segment as immutable in some cases.
		`DF_BINDNOW`
			If the flag with the name `DF_BINDNOW` with value `0x8` is set then all `.got.plt` relocations are to be resolved at load-time eagerly, rather than lazily on first call. The presence of this entry takes precedence over all directives to use lazy binding for this object. 
		`DF_STATIC_TLS`
			If the flag with the name `DF_STATIC_TLS` with value `0x10` is set the dynamic linker is instructed to reject attempts to load this object dynamically. It indicates that this object requires **static** [Thread-local Storage](Thread-local%20Storage.md)(see for more details).
- `.init`, `.fini`, `.init_array`, `.fini_array`, `.preinit_array`.
	See [Initialisers and Finalisers](Initialisers%20and%20Finalisers.md)
- `.tdata`/`.tbss`
	See [Thread-local Storage](Thread-local%20Storage.md)
- `.gnu.version`, `.gnu.version_d`, `.gnu.version_r`.
	Described earlier.
#### Shared Object Dependencies
When a link editor processes an archive library, it extracts library members and copies them into the output object file. These statically linked services are available during execution without involving the dynamic linker. Shared objects must also provide services, and the dynamic linker must attach the proper shared object files to the process image for execution, mapped `RO` and `COW` for writable data pages.

 When the dynamic linker maps a consumer object into memory, it inspects the `DT_NEEDED` entries of the dynamic structure to identify the shared objects that are needed to supply the program's services. By repeatedly connecting referenced shared objects and their dependencies, the dynamic linker constructs a complete process image. When resolving symbolic references, the dynamic linker examines the symbol tables with a breadth-first search. That is, it first looks at the symbol table of the executable program itself, then at the symbol tables of the `DT_NEEDED` entries (in the order they appear in `.dynamic)`, and the `DT_NEEDED` entries of those libraries (second-level), in order, and so on and so forth. This ensures the first-found definition in that traversal is used for symbol interposition and definition, unless `DF_SYMBOLIC` or visibility rules alter precedence. Shared object files must be readable by the process; no other permissions are required.
	Even when a shared object is referenced multiple times in the dependency list, the dynamic linker will connect the object only once to the process. 

##### Names and Path Searching
Names in the dependency list are either copies of `DT_SONAME` strings or the path names of shared objects used to construct the full process image. For example, if the link editor builds an executable file using one shared object's references with a `DT_SONAME` entry of `lib1` and another shared object with the path name `/usr/lib/lib2`, the executable will contain `lib1` and `/usr/lib/lib2` in its `DT_NEEDED` entries. 

If a shared object name has one or more slash (`/`) characters anywhere in the name, the dynamic linker uses that string directly as the path name from which to retrieve a library name.

###### Resolving Library Names Without Direct Paths
Three facilities specify shared object path searching:

1.  `DT_RUNPATH`
	The dynamic array entry with `DT_RUNPATH` yields a string that holds a list of directories, delimited by colons (`:`). For example, the string `/home/dir/lib:/home/dir2/lib:` tells the dynamic linker to search first the directory `/home/dir/lib`, then `/home/dir2/lib` and then the current directory to find directories.
2. `LD_LIBRARY_PATH`
	A variable called `LD_LIBRARY PATH` in the process environment, passed in `envp` array (array of character pointers to NUL-terminated strings, which constitute the environment for the new process image). This may hold a list of directories, optionally followed by a semicolon, and another directory list. It informs the dynamic linker of paths to search that may deviate from system default library directories. The following values would be equivalent to the `DT_RUNPATH` example:
	`LD_LIBRARY_PATH=/home/dir/usr/lib:/home/dir2/usr/lib:`
	`LD_LIBRARY_PATH=/home/dir/usr/lib;/home/dir2/usr/lib:`
	`LD_LIBRARY_PATH=/home/dir/usr/lib:/home/dir2/usr/lib:;`
	Some programs such as the link editor may treat the lists before and after the semi-colon differently, however the dynamic linker does not.
3. **Default system library directories**
	If the other two groups of directories fail to locate the desired library, the dynamic linker searches the default directories, `/usr/lib` or other such directories may be specified by the ABI supplement for a given processor. 
When the dynamic linker is searching for shared objects, it is not a fata error if an ELF file with the wrong attributes is encountered in the search e.g., `e_ident[16]` fields are not as required. The dynamic linker exhaustively searches all candidate directories until compatible library is found or all options are exhausted.

For security reasons, the dynamic linker ignores `LD_LIBRARY_PATH` for [setuid](https://man7.org/linux/man-pages/man2/setuid.2.html)/[setgid](https://man7.org/linux/man-pages/man2/setgid.2.html) programs. These run with elevated privileges. Allowing `LD_LIBRARY_PATH` to influence library loading would let unprivileged users inject malicious libraries.
##### Substitution Sequences
When a string provided by dynamic array entries with the `DT_NEEDED` or `DT_RUNPATH` tags and in pathnames passed as parameters to the [dlopen()](https://man7.org/linux/man-pages/man3/dlopen.3.html), a dollar sign (`$`) introduces substitution sequence. This sequence consists of `$` immediately followed by either longest *name* sequence or a name contained within left and right braces (`{`) and (`}`). A name is a sequence of bytes that start with either a letter or an underscore followed by zero or more letters, digits, and underscores. If a dollar sign is not immediately followed by a name  or brace-enclosed name, the behaviour of the dynamic linker is unspecified. 

If the name is `ORIGIN` then the substitution sequence is by the dnamic linker with the absolute pathname of the directory in which the object containing the substitution sequence originated. Moreover, the pathname will contain no symbolic links or `.` or `..`. Otherwise, when the name is not origin, the behaviour of the dynamic 
### Execution View

### Linking with shared libs and Running Programs

### The malloc hack and other shared lib problems
