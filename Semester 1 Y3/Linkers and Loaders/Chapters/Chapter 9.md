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
When loading a sharable object, version definition data from the loaded object is analysed to assure that it meets the version requirements of the calling object. The dynamic loader retrieves the entries in the caller's `Elfxx_Verneed` array and attempts to find matching definition information in the loaded `Elfxx_Verdef` table.

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
	
	I am led to believe that assemblers only emit primitive relocations that the linker then has to 'promote/specialise' when it has more information e.g., cannot find matching function in static libs and other objects, must be provided by shared lib, so emits `PLT` relocation at the offset the function appears in `.got.plt`, and adds that relocation to the `.rel/a.plt` section and adds auxilary information in the `.dynamic` section. 
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
Within a string provided by dynamic array entries with the `DT_NEEDED` or `DT_RUNPATH` tags and in pathnames passed as parameters to the [dlopen()](https://man7.org/linux/man-pages/man3/dlopen.3.html), a dollar sign (`$`) introduces substitution sequence. This sequence consists of `$` immediately followed by either longest *name* sequence or a name contained within left and right braces (`{`) and (`}`). A name is a sequence of bytes that start with either a letter or an underscore followed by zero or more letters, digits, and underscores. If a dollar sign is not immediately followed by a name  or brace-enclosed name, the behaviour of the dynamic linker is unspecified. 

If the name is `ORIGIN` then the substitution sequence is replaced by the dynamic linker with the absolute pathname of the directory in which the object containing the substitution sequence originated. Moreover, the pathname will contain no symbolic links or the use of `.` or `..` components. Otherwise, when the name is not `ORIGIN`, the behaviour of the dynamic linker is unspecified. 

When the dynamic linker loads an object that uses `$ORIGIN`, it must calculate the pathname of the directory containing the object. This calculation can be computationally expensive, so calculation should be avoided for objects that do not use `$ORIGIN`. If an object calls `dlopen()` with a string containing `$ORIGIN` and does not use `$ORIGIN` in one of its dynamic array entries, the dynamic linker may not have calculated the pathname for the object until `dlopen()` actually occurs. Since the application may have changed its current working directory before the `dlopen()` call, the calculation may not yield the correct result. To avoid this possibility, an object may signal its intention to reference `$ORIGIN` by setting the `DF_ORIGIN` flag. An implementation may reject an attempt to use `$ORIGIN` within a `dlopen()` call from an object that has not set `DF_ORIGIN` and did not use `$ORIGIN` within its dynamic array. 

For security, origin substitution sequences for [setuid](https://man7.org/linux/man-pages/man2/setuid.2.html) and [setgid](https://man7.org/linux/man-pages/man2/setgid.2.html) programs are not permitted. For such sequences that appear within strings specified by `DT_RUNPATH` (library search path) dynamic array entries, the specific search path containing the `$ORIGIN` seqeunce is ignored, but other search paths in the same string are processed. `$ORIGIN` sequences within a `DT_NEEDED` entry are treated as errors. - - `$ORIGIN` within `DT_NEEDED` or passed as `dlopen()` path may be treated as an error for privileged programs (depends on implementation and platform security policy).
### Execution View
We now transition to the runtime view of an ELF shared library/PIE executable. 

#### Program Headers
$S \in P$
	Where $S$ is a **section**,
	and P is a **program header**,
The above is truly a proposition; the inclusion of a section in a segment depends on the specific toolchain. Below I will discuss the typical and logical.

##### Program Header Types
This is an extension of [chapter 3's segment types table](Chapter%203.md#Segment%20Types).

| Name              | Value        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PT_NULL`         | `0x0`        | Array element is unused. Permits program header table to have ignored entries.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `PT_LOAD`         | `0x1`        | The array element specifies a loadable segment, described by `p_filesz` and `p_memsz`. Bytes that are to be mapped to the beginning of the memory segment. `.bss` described by `p_memsz > p_filesz`. `PT_LOAD` informs where to `mmap()` file bytes and how many pages to reserve. There are usually multiple `PT_LOAD` segments.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `PT_DYNAMIC`      | `0x2`        | The array element specifies dynamic linking information. Mechanism dynamic loader uses to locate the `.dynamic` array; essentially a pointer to the `.dynamic` array, which is included inside a `PT_LOAD` segment.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `PT_INTERP`       | `0x3`        | The array element specifies the location and size of a NUL-terminated path name to invoke as an interpreter; may not occur more than once. If present, much precede any loadable segment entry. During an `exec**()` call the system retrieves a path name from `PT_INTERP` and creates the initial process image from the interpreter's file segments, transferring responsibility to the interpreter to receive control from system and provide environment for the object. The interpreter receives control in one of two ways. First, it may receive a file descriptor to read the executable file, positioned at the beginning, and can map the executable's segments into memory. Second, depending on file format, the system may load the executable file into memory directly instead of giving the interpreter an open file descriptor.  The interpreter should be a shared object to avoid potential VMA collisions. Record [`AT_BASE`](https://man7.org/linux/man-pages/man3/getauxval.3.html#AT_BASE). |
| `PT_NOTE`         | `0x4`        | The array element specifies the location and size of auxilary information. See the [*note section*](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html#note_section) for more detail.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `PT_SHLIB`        | `0x5`        | This segment type is reserved but has unspecified semantics. Was reserved for shared library information in older specifications. Programs that contain an array element of this type do not conform to the ABI.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `PT_PHDR`         | `0x6`        | The array element, if present, specifies the location and size of the program header itself, both in file and in the memory image of the program. This segment type may not occur more than once in a file, moreover, it may occur only if the program header table is part of the memory image of the program. If present, it must precede any loadable segment entry.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `PT_TLS`          | `0x7`        | The array element specifies the *Thread-Local Storage* template region. Implementations need not support this program table entry. See [Thread-local Storage](Thread-local%20Storage.md) for more detail.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `PT_GNU_EH_FRAME` | `0x6474e550` | The array element specifies the location and size of exception handling information as defined by the [`.eh_frame_hdr`](https://refspecs.linuxfoundation.org/LSB_1.3.0/gLSB/gLSB/ehframehdr.html) section.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `PT_GNU_STACK`    | `0x6474e551` | The `p_flags` of this member indicate whether the application requests an executable stack (`PF_X`) or not. The kernel uses this to mark the process stack non-executable. The absence of this header indicates the stack will be executable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `PT_GNU_RELRO`    | `0x6474e552` | See [RELRO](RELRO.md) for more detail.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `PT_GNU_PROPERTY` | `0x6474e553` | Points to a blob of 'property notes' contained within the `.note.gnu.property` section. This section contains precisely one ELF [note](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html#note_section) with the descriptor portion containing a sequence of 'property' entries, each with the following layout:<br><br>`uint32_t pr_type;`<br>`uint32_t pr_datasz;`<br>`uint8_t  pr_data[pr_datasz];`<br>`#padding`<br><br>Property types are defined per architecture, but contain the same container format described above. See [ARM specifics for linking](ARM%20specifics%20for%20linking.md) for more detail.                                                                                                                                                                                                                                                                                                                                                                                  |

It should be noted that $ARM$, other vendors, and alternate operating systems supply their own $PT$ types. See [this](../ARMv7/ARM%20Specifics%20For%20Linking.md#Program%20header%20types) for more details regarding $ARM$.
##### Program Header Typical Contents
| Name              | Contents                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PT_NULL`         | **Contents**: Nothing. <br>**Toolchain behaviour**: Can be emitted to reserve entries; ignored.<br>**Runtime**: no mapping, no `auxv`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `PT_LOAD`         | **Contents**: File-backed bytes to be mapped into memory. Typically multiple `PT_LOAD` entries exist (e.g., `text` segment and `data` segment).<br>**Typical contents**: `.text`, `.plt`, `.rodata`, `.got`, `.got.plt`, `.dynamic`(often inside data segment), `.gnu.hash`/`.hash`, `.data`, `.init_array`, `.dynsym`, `.dynstr`, `.eh_frame`, etc.<br>**p_filesz/p_memsz semantics**: `p_filesz` bytes are mapped from the file, an additional `p_memsz - p_filesz` bytes describe the `.bss` section,  typically residing at the end of the data segment. <br>**Alignment & mapping**: see [here](#mapping%20rules%20and%20formulas) for more detail.<br>**Protections**: `p_flags` $\to$ page protections. Loader may temporarily relax protections in case of text relocations and then restore. There is also the case of [RELRO](RELRO.md).<br>**Runtime Role**               |
| `PT_DYNAMIC`      | **Typical Contents** : describes the `.dynamic` table location/size via `p_vaddr`, `p_offset`, `p_filesz`. It does not itself supply those bytes, which are instead present in some `PT_LOAD` segment.<br>**Runtime Role**: dynamic linker reads `AT_PHDR` to find `PT_DYNAMIC`, computes `runtime_DT = load_bias + p_vaddr` and walks the `ElfXX_Dyn` structures to find `DT_*` targets using `load_bias + d_un.d_ptr`.                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `PT_INTERP`       | **Contents**: The array element specifies the location and size of a NUL-terminated path name to invoke as an interpreter. Stored at `p_offset` with `p_filesz` covering the string.<br>**Loader behaviour**: Reads `PT_INTERP` and maps the interpreter into the process; sets `AT_BASE` `auxv` to interpreter base. Then transfer control to interpreter's entry point, supplied as `AT_ENTRY`, this is typically the dynamic linker, which is then capable of doing `mmap()` for the main executable. This is standard kernel behaviour, but for our implementation of a dynamic linker and loader, this may change.<br>**Ordering rule**: If present, appears before `PT_LOAD` entries in `phdr` array.                                                                                                                                                                          |
| `PT_NOTE`         | **Contents**: Contains ELF [notes](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html#note_section).<br>**Typical contents**: .note.gnu.build-id, .note.ABI-tag<br>**Alignment**: Notes use 4-byte alignment for the descriptor, and 4 byte alignment for next note entry (32-bit object).<br>**Runtime**: loader can read notes for ABI checks; debuggers/tools use build-id.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `PT_SHLIB`        | **Contents**: reserved, no defined semantics in modern ABIs.<br>**Toolchain behaviour**: modern linkers/loaders ignore.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `PT_PHDR`         | **Contents:** Specifies the location and size of the program header table itself, both in file and in the memory image of the program. `p_vaddr` points to the address where the `Elf_Phdr` array is mapped in the process image.<br>**Auxv relation**: Set `AT_PHDR = load_bias + p_vaddr` (or compute `phdr` address from first `PT_LOAD` if `PT_PHDR` absent). `AT_PHENT = sizeof(Elf_Phdr)`, `AT_PHNUM = phnum`<br>**Runtime**: used by `dl_iterate_phdr` debuggers, and dynamic linker/loader to find other headers.                                                                                                                                                                                                                                                                                                                                                            |
| `PT_TLS`          | See [Thread-local Storage](Thread-local%20Storage.md) for more detail                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `PT_GNU_STACK`    | **Contents**: no file bytes, usually `p_filesz` == 0; `p_flags` indicate whether the stack should be executable.<br>**Effect:** If present and `PF_X` is clear, the kernel marks stack non-executable. Absence often implies executable stack historically, but most modern linkers will emit this explicitly.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `PT_GNU_RELRO`    | See [RELRO](RELRO.md) for more detail                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `PT_GNU_EH_FRAME` | **Contents**: see [here](https://maskray.me/blog/2020-11-08-stack-unwinding) for more detail.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `PT_GNU_PROPERTY` | **Contents**: Points to `.note.gnu.property` section. That note must be present in a loadable mapping. The note of type `NT_GNU_PROPERTY_TYPE_0` contains a descriptor portion with a sequence of one or more `pr_property` entries, each of which describes a distinct property of the object:<br><br>`struct pr_property {`<br>    `uint32_t pr_type;`<br>	`uint32_t pr_datasz;`<br>	`uint8_t  pr_data[pr_datasz];`<br>	`// Padding to 4 or 8 bytes, 32/64 bit`<br>`}`<br><br>Property types are defined per architecture, but contain the same container format described above. See [ARM specifics for linking](ARM%20specifics%20for%20linking.md) for more detail.<br><br>`Runtime usage`: loader reads these early to select mapping semantics, and honour required features. Unknown properties are ignored unless they are mandatory according to the processor supplement. |

#### Mapping Rules and Formulas
$S \in P  \iff  [ S.sh\_addr, S.sh\_addr + S.sh\_size )  \Subset   [P.p\_vaddr, P.p\_vaddr + P.p\_memsz )$
##### Notation
- `PAGESZ`
	Page size in bytes, `4096` for our purposes.
- `p_offset, p_vaddr, p_filesz, p_memsz`
	Fields acquired from the program header.
- `file_map_offset`
	Page-aligned file offset to pass to `mmap()`.
- `seg_page_vaddr`
	Page-aligned link-time VA to map to (add `chosen_base` at runtime). `p_vaddr` is only required to be congruent with `p_offset`, where `[p_offset, p_vaddr] % PAGESZ == 0` and may not be page aligned depending on the content segments.
- `delta`
	`delta = p_vaddr & (PAGESZ - 1)`, determines where segment data begins within the first mapped page.
- `map_filesz`
	Number of bytes to `mmap()` from the file.
- `map_memsz`
	Number of bytes to reserve in memory for the whole segment. 
- `load_bias`
	The VMA chosen for the object in memory, as ordained by the loader.
- `runtime_map_addr`
	`runtime_map_addr = load_bias + seg_page_vaddr` = address of the first page at which to map the segment.
- `runtime_seg_content`
	`runtime_seg_content = runtime_map_addr + delta`.

##### Formulas
- `file_map_offset = p_offset & ~(PAGESZ-1)`
	`-1` yields all low bits needed to clear to get the page boundary. We perform a logical not to clear these bits, and then perform a logical and with the file offset to clear the bits in `p_offset` to obtain a **page aligned file offset** rounded down to a page boundary. It should be noted that yes, the page will contain bytes from the previous segment up to `runtime_seg_content`. If multiple `PT_LOAD`s request permissions for the same page, take the union of their `P_FLAGS`.
- `seg_page_vaddr = p_vaddr & ~(PAGESZ-1)`
	Must map memory at a page boundary. Round down link-time `p_vaddr` to the page start.
- `delta = p_vaddr & (PAGESZ-1)`/`delta = p_offset & (PAGESZ-1)
	Yields the byte offset within the first page where the segment data actually begins. May need to compute for `p_offset` to determine where the segment's file bytes begin in the first `mmap()`'ed file page. From now on, these will be referred to as `delta_v` and `delta_o`, respectively.
- `map_filesz = round_up(delta_o + p_filesz, PAGESZ)`
	The number of bytes to map from the file is the sum of:
	1. `delta_o` - the offset of the segment start within the first page of the file-mapped region.
	2. `p_filesz` - the actual size of the segment's data in the file. 
	This sum is rounded to the next page boundary to satisfy the `mmap()` page alignment requirement. Without adding `delta_o`, the first few bytes of the segment could be missed because `mmap()` starts at the page boundary of file_map_offset, which may be before the segment start.
- `map_memsz = round_up(delta_v + p_memsz, PAGESZ)`
	Total number of bytes to reserve in memory for the segment (including zero initialised bytes) is the sum of:
	1. `delta_v` - the offset of the segment start within the first memory page
	2. `p_memsz` - the segment size in memory (may exceed `p_filesz` to define `.bss`)
	This sum is rounded to the next page boundary to ensure the mapping is page-aligned.
- `runtime_map_addr = load_bias + seg_page_vaddr`
	The address at which the page-aligned segment is mapped in runtime memory.
	- `seg_page_vaddr` is the page-aligned link-time VA of the segment.
	- `load_bias` is the runtime base address chosen by the loader for this `ET_DYN` object.
	Adding them computes the runtime address of the first virtual page of the segment. The segment's true content starts at `runtime_map_addr` + `delta_v`, or alternatively, `load_bias + p_vaddr`.
- **Zero fill bytes**
	After mapping the file bytes, any remaining bytes in the last page of the segment that extend beyond `p_filesz` but are within `p_memsz` must be zeroed.
		Formulaically: zero bytes in `[runtime_map_addr + delta_v + p_filesz, runtime_map_addr + delta_v + p_memsz)`.

##### `mmap()` considerations
[`mmap()`](https://man7.org/linux/man-pages/man2/mmap.2.html) expects file offsets and memory addresses to be page aligned, which is why above there is a considerable amount of rounding, and the use of a delta in the formulas to locate the true start of the segment.

`prot` is set according to segment flags; for partially overlapping permissions, `prot` is set according to the union of the overlapping permissions. 

The [flags](https://man7.org/linux/man-pages/man2/mmap.2.html#:~:text=not%20be%20accessed.-,The%20flags%20argument,-The%20flags%20argument)argument determines whether updates to the mapping are visible to other processors mapping the same region. We typically map segments as `MAP_PRIVATE`, and map `.bss` as `MAP_ANONYMOUS`, and use `MAP_FIXED` to clobber existing ranges, though typically this is only for userspace loaders for non-relocatable ELF executables.


### Linking with Shared Libraries and Running Programs
Below is the totality of the previous chapters! We discuss the *entire pipeline,* going from a number of arbitrary `ET_REL` files to an `ET_DYN` executable, loading this in via a userspace loader, and then describing how shared libraries are linked into the process image at load-time. 


- Take in ET_RELs, Storage allocation, Symbol resolution and library search, static relocation application, relocation emittance, creation of needed sections like .dynamic and setting corresponding values, userspace loading of ET_DYN, and dynamic linking of said ET_DYN. Drag in all files.
