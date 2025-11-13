# Symbol Management
Symbol management is considered a linker's key function, encompassing symbol binding and resolution - the process in which a symbol reference in one file to a name in a second file is resolved to a machine address. 

Without some way to refer from one module to another, there wouldn't be much use for a linker's other capabilities.

It should be noted that the sheaf of tables that a compiler maintains for things like establishing context in type checking is mostly discarded. The compiler emits only externally visible symbols, potentially function-level symbols for debugging. ELF's `.symtab` is flat and global at the file level - there is no 'subtables' tied to lexical scopes.
## Binding and Name Resolution
Linkers handle a variety of kinds of symbols. Each input module includes a symbol table. The symbols include the following:
### Symbol types 
- Global symbols defined and perhaps referenced in the module
- Global symbols referenced but not defined in the module(externals)
- Segment/section names, considered to be global symbols defined to be at the beginning of the segment
- Nonglobal/function-level symbols, usually for debuggers and crash dump analysis, unneeded for linking process.
- Line number information, tell source language debuggers correspondence between source lines and object code. 

Linker reads all of symbol tables in input module and extracts necessary information such as size, value and info. Then builds the link-time symbol tables and uses those to guide the linking process. Depending on output format, linker may place some or all of the symbol information in the output file. 

Some formats such as ELF can have multiple symbol tables per file, ELF Shared libraries can have one symbol table with just info needed for dynamic linker, and separate, larger table useful for debugging and relinking. 
## Symbol table formats
Linker symbol tables generally similar to those in compilers, though usually simpler. Generally maintain a few distinct symbol tables; one lists the input files and library modules, keeping the per file information. A second table handles global symbols, the ones that the linker has to resolve among input files. A third table may handle intramodule debugging symbols, though more often that not the linker need not create a full-fledged symbol table for debug symbols, given it only needs to pass the debugging symbols through the input to the output file. 

The coalesced, in-memory symbol table for a linker is essentially a hash table with a separate chaining collision resolution mechanism. Each 'hash header' points to a linked list of symbols that hash to the same value. 

To locate a symbol in the table:
1. `hash(symname) = hashval`
2. `bucket = hashval % NBUCKET`
3. Traverse linked list `symhash[bucket]`
4. Compare names and other properties until a hash is found.

```C
struct sym *symhash[NBUCKET]; //array of length NBUCKET containg sym elements

struct sym {
	struct sym *next;
	int         fullhash;
	char        *symname;  // string derived from st_name
}
```

`strcmp` for each symbol in a bucket is expensive and modern languages like C++ produce long, mangled names making comparison operations expensives, even more so with exaggerated collisions. 

The optimisation is seen above. Store the pre-modulo hash value first, and for each symbol in the chain, compare its stored hash value to the computed hash, if it matches, then do `strcmp()`. 

## Module Tables
Linker needs to track every input module seen during linking run. This includes both those linked explicitly and those extracted from libraries. This is the role of the **module table**. For most files, the key information is in the headers, so the table most often just stores a copy of the header.

For our purposes, this is no longer used; this is the forerunner to the methodology employed by modern linkers. 

In short, when a linker runs, it keeps track of every such input module during the link. Each module table entry represents one input file. Each entry holds roughly the following kind of data:

| Field                       | Description                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------- |
| Header                      | A copy of the a.out file header, gives sizes and addresses of .text, .data, .bss, and tables. |
| Symbol Table Pointer        | Pointer to in-memory copy of the symbol table.                                                |
| String Table Pointer        | Pointer to the in-memory copy of symbol names.                                                |
| Relocation Table Pointer(s) | For .text and .data relocation entries.                                                       |
| Computed output offsets     | The positions of this module's text/data/bss in final output file after storage allocation.   |
| Flags                       | Indicate if the module was pulled from library, has unresolved symbols etc.                   |
| Next/previous pointers      | Chain modules together.                                                                       |


This table is used as follows:
First Pass:
	For each input file, read file header and symbol table
	Create new module table entry
	Copy the symbol table and string table into memory such that linker can utilise them
	It adjusts symbol entries such that each symbol name points to its currently in-memory string, (instead of a file offset)
Second pass:
	Walk through all module table entries
	Resolve definitions and references
	If an undefined symbol is found and can be satisfied by a library member, that member is read in - a new module table entry is created for it.
Third pass:
	Linker assigns output addresses to each module's sections
	Updates the computed offsets fields in each module table entry
Fourth pass:
	Applies relocations using data in the module table.
```
typedef struct Module {
    AOutHeader header;          // copy of a.out file header
    Symbol *symtab;            // pointer to in-memory symbol table
    char *strings;              // pointer to in-memory string table
    Reloc *text_relocs;         // relocations for .text
    Reloc *data_relocs;         // relocations for .data
    long text_offset;           // output offset for .text
    long data_offset;           // output offset for .data
    long bss_offset;            // output offset for .bss
    struct Module *next;        // next module in list
} Module;
```

### ELF linker equivalent
Of course, there is no single equivalent. But an internal data structure that suits the needs of modern ELF can be developed to act in a similar way to a module table. 

Initially, one parses the ELF header `ELF32_Ehdr`, and uses it to locate sections, symbol tables, and relocation entries in the file.

We use this to construct an entry into an ELF input representation semi-equivalent to a module table:
```C

#include <stdint.h>

typedef struct GlobalSymbol {
    char *name;
    Symbol *definition;          // points to resolved definition
    Symbol **references;         // optional list of refs
    uint32_t num_refs;
} GlobalSymbol;

typedef struct Relocation {
	uint64_t        offset;
	uint32_t        type;
	struct Symbol * symbol;
	int64_t          addend; //rela
} Relocation;

typedef struct Symbol {
	char *name;
	struct InputSection    *section;
	uint64_t value; //offset within section
	uint8_t size;
	uint8_t binding; 
	uint8_t type;                  
    uint8_t visibility;      
} Symbol;

typedef struct InputSection {
	char *name;
	uint8_t *data; //pointer to section contents 
	uint64_t size;
	uint64_t alignment;
	Relocation *relocs;
	uint32_t num_relocs;
	uint32_t flags;
    uint32_t type;
    
    uint64_t vaddr;
    uint64_t offset;
} InputSection;

typedef struct InputFile {
	char *filename;
	InputSection **sections;
	uint32_t num_sections;
	Symbol **symbols;
	uint32_t num_symbols;
} InputFile;
```
#### Parsing
1. Parse ELF File header
	Use ELF header `Elf32_Ehdr` to locate **section header table** and `.symtab`
2. Parse Sections
	For each section in the ELF file:
	- Allocate `InputSection` 
	- Fill in:
		- `name` -> from section header string table
		- `size, alignment` -> from section header
		- `flags`
		- `type`
		- `data` -> may be null for bss
	- If the section has relocations, parse them into Relocation structs:
		- `offset` -> where in section to apply the relocation
		- `symbol` -> will be resolved later
		- `type` -> type of relocation
		- `addend` -> `.rela{name}` only
3. Parse symbols
	For each entry in `.symtab`
	- Allocate a `Symbol`
	- Fill in
		- `name` -> symbol name from `.strtab`
		- `section` -> pointer to `InputSection` symbol belongs to(NULL if not defined)
		- `value` -> offset within section
		- `size, binding, type, visibility`
4. Create `InputFile` object
	Set `filename`
	Store arrays of `InputSection*` and `Symbol*`
	Store counts `num_sections` and `num_symbols`.
#### Name Resolution
1. Build **GST** visible to all modules.
	For each input `Symbol`:
		If `binding` $\in$ `{STB_GLOBAL, STB_WEAK}` -> insert or update in GST
		Mark undefined symbols are unresolved `definition == NULL`
2. Resolve undefined symbols
	For each undefined symbol in all `InputFile`s:
		Lookup by name in GST
		If found and has definition
			`symbol -> resolved = global_symbol`
	If `resolved->definition == NULL` (i.e., could not find any resolution to the global symbol found from original linker input)
		If libraries are available, search and load the defining object, updating the GST to complete the entry for the undefined global symbol. 
	Else report linker error.
3. Relocs
	Redirect relocation entries to point to entries in global symbol table, rather than local entries. `reloc->symbol = reloc->symbol->resolved->definition` where `resolved != NULL`

It should be noted that it is entirely valid to build a fully complete GST rather than mixing the completion of the GST with the resolution of names across modules. One may choose to add everything to the GST, then search libraries to resolve undefined symbols, and then resolve the symbols provided from the original linker input. Both methods are completely valid. 
#### Storage Allocation
Classify sections by runtime properties
	Determine `SHF_ALLOC`
	Determine access flags (RWX)
	Determine alignment requirement `sh_addralign` and size
	Determine whether loaded from file or can be zeroed in memory e.g., `.bss`
3. Form permission groups/segments(`PT_LOADS`)
	This is the initial grouping via *page permissions* according to headers:
	- Read+Execute - usually `.text`
	- Read Only - `.rodata`
	- Read + Write - `.data`, `.rel/a`, etc
	- Read + Write but zero initialised `.bss`
		- Avoid creating pages that are both W and X for security purposes. 
4. **Order sections within each group**
	The linker picks a conventional ordering of the sections of each permission group such that they can be arranged logically.
	![[Pasted image 20251106121541.png]]
	Often akin to this. Ensures predictable addresses and ensures that early executing code like `.init` lies close to main `.text. 
	Related sections kept contiguous to exploit spatial locality. Segments identified by segment type constant such as `PT_LOAD`, `PT_DYNAMIC`, `PT_GNU_RELRO`.
5. **Assign virtual addresses and file offsets**
	Assume page size of `0x1000`
	Assume load base address of `0x00008000`
	Start with first loadable segment e.g., RX
	Align to next 4KB boundary
	Assign `p_vaddr = base_addr`
	Assign `p_offset` = `file_offset_after_headers`
	For each section in this segment:
		Align its start to alignment requirement
		Place sequentially in memory
		Advance `current_addr` and `current_offset` accordingly to monitor address space progression for future assignments. 
		Continue until all segments written. 
#### Applying relocations
- Walk through each `InputSection` with relocations
	- For each `Relocation
		- Compute relocation target address
			`uint64_t *location = (uint64_t *)(section->data + reloc->offset); 
			`*location = reloc->symbol->value + reloc->addend;`

	- Apply relocation based on type. 

Just write final `ET_EXEC` ELF output now.



## Global Symbol Table
The linker may opt to keep a global symbol table internally, which keeps an entry for every global symbol referenced or defined in any input file. When the linker reads input files, it extracts global symbols and adds them to this master table. This process is usually managed using a hash function and chained lists to locate entries quickly.

```c
struct glosym {
	struct glosym *link;
	char *name; // actual string name
	long value; // aligned relative address to the start of the section
	struct nlist *refs; // linked list of entries in local symbol tables where symbol appears as definition or ref
	int max_common_size; // common blocks
	char defined;
	char referenced; 
	unsigned char multiply_defined; // handle multiple defs
}
```

For each input file:
- The linker reads its symbol table
- For each symbol
	- Look up its name in the global symbol table
		- If not found -> create a new `glosym` entry
	- Add this module's reference/definition to the symbol's refs chain

As symbols are added:
- If the symbol is defined in a file, `defined = 1`
- if the symbol is referenced, mark `referenced = 1`
- If the symbol is defined in multiple files, set `multiply_defined = 1`
- Every symbol appears once in the global table now, and has list of all occurrences

Each module's relocation records refer to symbols by local symbol table index. The linker must construct a vector of pointers for each module. As in, each entry in the module's local symbol table is replaced with a pointer to the corresponding global symbol table entry. 

Now when the linker encounters a **relocation**, it can just follow the `glosym` pointer and use its `value` field in the relocation. It is also free to further view its references linearly if need be. We do not manipulate any bytes in sections yet, we just merely construct the structures that permit our relocations to work globally post storage allocation. 


## Special Symbols
The names used in object file symbol tables and in linking are often not the same names used in the source programs from which the object files were derived.

There are three reasons for this:
1. Avoiding name collisions
2. Name overloading(same name, different parameters)
3. Type checking
The process of turning source program names into object file names is called **name mangling**, a technique derived from the need to resolve unique names for programming entities. 

### Name Mangling
In older object formats, compilers used names from the source program directly as the names in the object file, perhaps truncating long names. Problems occurred as a result of collisions with names reserved by compilers and libraries.

`CALL MAIN`
`END`
Would crash, on OS/360 systems, the programming convention was that every routine including the main program has a name, and the name of the main program is called `MAIN`. The system routine that started Fortran programs was also called `MAIN` and this led to a recursion and a crash. People learned to not use *reserved* names but this was fragile. 

The approach taken on early UNIX systems was to mangle the names of procedures so they wouldn't collide with the names of libraries and other routines. C procedure names were modified with a leading underscore such that `main` became `_main`. 
	The reason the name mangling approach was chosen rather than using collision-proof library names e.g., `$printf` was simply due to convenience, as the assembler language libraries were already extensive by the time UNIX was rewritten in C, was easier to mangle names of new C and C-Compatible routines. 

Name mangling in C is no longer needed as  ELF provides a more structured symbol table that stores rich metadata such as binding(local, global, weak), type, and visibility. Older formats like `a.out` had limited or truncated symbol names, requiring compilers to encode extra info into the symbol name itself. Since C has no overloading, each external identifier is already unique at link time, as such can resolve symbols directly using their literal names. Even main is just linked to `crt1.o` which simply declares a reference to `main` which the linker resolves to the user's main definition exactly as it would any other global function. 

Collisions with libraries(static) are handled via only pulling in object files from a library is they resolve an undefined symbol currently in the global symbol table. 

#### Name Mangling in C++
 C++ supports features that C does not, including:
- Templates(generics)
- Function overloading
- Classes and member functions
- Namespaces
Because of this, the simple symbol name in the object file is not enough to uniquely identify a function or method.

E.g., 
```C++
void print(int);
void print(double);
```
While perfectly valid code, at the linker level without mangling this would result in a name collision, and for 1 global symbol this would result in 2 strong definitions and a linker error. 

Many names are mangled in C++ to preserve semantics, although, as linker authors we are generally only concerned with global/weak symbols that are mangled to preserve uniqueness and avoid collision to ensure correct name resolution.

Variables names outside of C++ classes are not mangled at all. 

Function names not associated with classes are mangled to encode the types of the arguments by appending `__F` and a string of letters that represent the argument types and type modifiers. For example, a function:
```C++
func(float, int, unsigned char) -> func__FfiUc
```

Class names are considered types and are encoded as the length of the class name followed by the name, such as `4Pair`. When a class name includes internal classes across multiple levels (a 'qualified' name) it is encoded starting with the letter `Q`, followed by a digit indicating the number of levels, followed by the encoded class names. For example, the name `First::Second::Third` becomes `Q35First6Second5Third`.
A function signature uses this encoding to represent its argument types; a function `f(Pair, First::Second::Third)` becomes `f__F4PairQ35First6Second5Third`. The notation `__F` indicates the function arguments begin, followed by the type encodings for `Pair` and `First::Second::Third` respectively.

Class member functions are encoded as the function name, two underscores, the encoded class name, then F and the arguments. So `cl::fn(void)` becomes `fn__2clFv`. Special functions including constructor, destructor, new, and delete all have encodings too. 

Since mangled names can be so long, there are two shortcut encodings for functions with multiple arguments of the same type. The code `Tn` means same type as the `nth` argument. and `Nnm` means `n` arguments of the type as the `mth` argument. A function `segment(Pair, Pair)` would be `segment__F4PairT1` and a function `trapezoid(Pair, Pair, Pair, Pair)` would be `trapezoid__F4PairN31` (non-member functions).

#### Link-Time Type Checking With Mangled Names
The process by which a linker enforces type checking across modules via type info encoded in mangled names.

When one object files calls a function defined in another, the call refers to the mangled symbol for that function signature. The linker matches undefined references against defined symbols by their mangled names.
If a mangled name is produced in an object file and a definition cannot be found for it that matches the type string for the reference, an error is reported. Thus mangled names act as link-time type signatures enforcing strong typing across modules. 


### Weak External and other kinds of symbols
Up to this point, considered all linker global symbols to present themselves as either a definition or a reference to a symbol. Object formats such as ELF can qualify a reference as weak or strong.

There are three main symbol bindings in ELF. The ELF specification says:
- `STB_LOCAL`
	Have a value of 0, defined by the rule that they are not visible outside the object file containing their definition. Local symbols with the same name can exist across multiple files without creating conflicts. All entries with `STB_LOCAL` binding are placed before `STB_GLOBAL` and `STB_WEAK` entries. To facilitate this distinction, a symbol table section's `sh_info` section header member holds the symbol table index for the first non-local symbol. 
- `STB_GLOBAL`
	- Global symbols have a value of 1, they are visible to all object files being linked. Link editors do not permit multiple definitions of `STB_GLOBAL` symbols, and throw a linking error if there is. The primary linking rule for global symbols is that one file's definition of a global symbol satisfies another files undefined reference to that same symbol.
- `STB_WEAK`
	- Weak symbols have a value of 2. They resemble global symbols but their definitions carry a lower precedence. 
	- If defined global symbol exists with the same name as a weak symbol, the linker editor honours the global definition and ignores the weak one. The presence of a weak symbol does not cause an error. 
	- If a weak symbol is referenced but undefined i.e., it is a weak reference where `st_shndx` is `SHN_UNDEF`, the link editor does not extract archive members. Such unresolved weak symbols are typically given a zero value. 
	- Weak symbols allow linking to proceed without error even if a reference remains unsatisfied. They can be employed to provide fallback/default definitions e.g., library files which can be overridden by other files.
	- Compilers like GCC and clang support marking a symbol weak using `__attribute__((weak))` or `#pragma weak symbol`.


### Maintaining Debugging Information
Modern compilers support source language debugging, can debug object code by referring to source program function and variable names. compilers support this by putting information in the object file that provides a mapping from source line numbers to object code addresses as well as information that describes all of the functions, variables, types, and structures used in the source program.

#### Line Number Information
To support breakpoints, single-stepping and stack tracebacks, compilers generate a mapping from program addresses to source line numbers. 

Each line of generated code has a line number entry generated, giving the file scoped line number and the beginning address of the corresponding object code. If the program address lies between two line number entries, the debugger reports it as being the lower of the two line numbers. 

DWARF lets the compiler map each byte of object code back to a source line, while other techniques may just specify approximate locations. 
#### Symbol Information
Compilers generate debug symbols describing the names, types, and locations variables, functions, and structures. Needs to encode type definitions so that debugger can correctly format all of the subfields in a structure or union. 

The symbol information is an implicit or explicit tree, with top-level entries for types, functions, and variables, and nested entries for fields, local variables, and blocks. Special markers track variable scope and lifetime within function e.g., `begin block` and `end block` markers referring to line numbers allowing the debugger to identify what variables are in scope at each point in the program.

Location information for symbols is complex, the location of a static variable is just a fixed memory address, local variables typically stack allocated and accessed at offset from frame pointer. If register allocated the debugger must know which register holds the variable at which instruction. In heavily optimising compilers, compiler may compute values on the fly or eliminate them; the debugger in this case can only approximate. 



