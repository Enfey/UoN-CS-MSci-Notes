# Shared libraries
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
From now, we no longer refer to 'static shared libraries'. The advent of PIE executables and ASLR nullified many of the issues surrounding address space management for shared libraries. Between the formers and the paging system, one is able to load a shared library at an arbitrary address, where the loader and the runtime linker are responsible for handling any relocation automatically. This generally amounts to assigning a random load-base and filling in the `.got` and lazily (or eagerly) filling in the `.plt` for a given shared object. This is relatively mechanical. There is some small runtime overhead for this scheme as discussed in the [[Chapter 8|prior chapter]].


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
		Elfxx_Half    vn_version;
		Elfxx_Half    vn_cnt;
		Elfxx_Word    vn_file;
		Elfxx_Word    vn_aux;
		Elfxx_Word    vn_next;
	} Elfxx_Verneed;
  	```
	`vn_version`
		Version of structure. This value is currently set to `1`.
	`vn_cnt`
		Number of associated `verneed` array entries
	`vn_file`
		Offset to the file name string of the library string, often in `.dynstr`
	`vn_aux`
		Offset to a corresponding entry in `Elfxx_Vernaux` array, which gives rich version definition.
	`vn_next`
		Offset to the next `Elfxx_verneed` entry, in bytes.
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
Generally, the following sections are generally created. Certain sections will be elaborated on more in [[Chapter 10]] and beyond, otherwise, we are already familiar.
- `.text`
- `.rodata`
- `.bss`
- `.dynsym`
- `.dynstr`
- `.rel/a.dyn`
- `.plt`
- `.got` (and `.got.plt`)
- `.hash` or `.gnu.hash`
	Discussed in chapter 10
- `.dynamic`
	We will discuss this here as it is extremely relevant.
	If an object file participates in dynamic linking (most shared libraries will), then it will contain the  `.dynamic`, and as a ready .so its program header table will contain the `.dynamic` section in a segment with type  `PT_DYNAMIC`. A synthetic symbol `_DYNAMIC` is emitted by the static linker to label the section, whose value is the virtual address of .dynamic relative to the DSO's own load-base, relocating via the load-time delta and load-time. The section contains an array of the following structures:
  ```C
  typedef struct {
	Elf32_Sword	d_tag;
   	union {
   		Elf32_Word	d_val;
   		Elf32_Addr	d_ptr;
	} d_un;
} Elf32_Dyn;

extern Elf32_Dyn	_DYNAMIC[];
```
	For each object with this type, `d_tag` controls the interpretation of `d_un`.
	`d_val`
		These objects represent integer values with various interpretations.
	`d_ptr`
		These objects represent program virtual addresses. The dynamic linker computes actual addresses based on the original file value and the memory base address. For consistency, files do not contain relocation entries to correct addresses in the dynamic structure.
	![](Pasted%20image%2020251202211040.png)
	For `dt_tag`. If a tag is marked mandatory, the dynamic linking array for an ABI-conforming file must have an entry of that type.
	`DT_NULL` marks the end of the `_DYNAMIC` array.
	`DT_NEEDED`
	`DT_SONAME`
	`DT_FLAGS` holds flag values specific to the object being loaded. 
		`DF_ORIGIN` - signifies the object being loaded may make reference to the $ORIGIN substitution string (look at this)
		`DF_SYMBOLIC`
		`DF_TEXTREL`
		`DF_BINDNOW`
		`DF_STATIC_TLS`
- `.init`, `.fini`, `.init_array`, `.fini_array`, `.preinit_array`.
- `.tdata`/`.tbss`
- `.gnu.version`, `.gnu.version_d`, `.gnu.version_r`.

#### Execution View

### Linking with shared libs and Running Program

### The malloc hack and other shared lib problems
