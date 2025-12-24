Libraries are collections of object files that are lazily included in a linked program. Every modern linker enables library support, both static and dynamic. Here we are concerned with traditional statically linked libraries.

## Purpose of libraries
A statically linked library is merely a collection of compiled object files usually with some added metadata bundled into what one calls an archive. 

The purpose of static libraries is to promote modularity and code reuse, without enforcing runtime dependencies. Linker's extract only the object modules that resolve currently unresolved symbols, thus the resultant executable contains all necessary code, becoming self-sufficient by satisfying dependencies early. Static libraries need not be PIC. They contain `ET_REL` ELF files for our purposes; the linker extracts the needed members and permanently relocates them into fixed addresses. 

Dynamic linking is often regarded as superior in multi-program OS targets, as multiple programs often have the same dependencies; we can leverage this to produce smaller binaries, and link-time relinking is not required for changes to `.so` files, whereas static executables would be re-linked to pick up fixes. Although there is the constraint that for dynamically linked libraries, they must be PIC. 

## Static Library Formats
### The Archive Container Format (.a)
This is the static library format using on $*nix$(and also Windows). It can be used for collections of any types of files though in practice it's rarely used outside of this domain.

It should be noted that all modern $*nix$ systems maintain this format although with slight variations - the output of an `ar` call was never standardised.
#### Archive Header.
Every archive begins with the magic ASCII signature:
```C
!<arch>\n // 21 3C 61 72 63 68 3E 0A
```
Which marks the file as an archive.

#### Archive Member Headers
Each archive member is preceded by a 60-bye fixed-width ASCII header.
```c
char name[16]; // Member name (space-padded or '/' offset)
char st_mtime[12] // Decimal alteration time
char st_uid[6]; // Decimal user ID
char st_gid[6]; // Decimal group ID
char mode[8]; // Octal file mode (permissions)
char size[10]; // Decimal file size in bytes
char eol[2]; // `<newline>
```
Each field is ASCII to keep archives printable and endianness agnostic. 
The member's contents immediately follow according to `size`.

If `size` is odd, a newline padding byte follows the member's contents such that the next header starts on an even boundary. There is no requirement that the member's contents start on any particular boundary. 

The UID, GID, mode, time fields are preserved for completeness but have no linking relevance. 

#### Member names and the `//` long-name table
The name field is 16 bytes. Historically older systems limited filenames to 14 characters so this was fine. Modern systems provision a special member to store member names longer than 16 characters. 

The `//` member, the long-name table:
- If a name $\leq$ 16 characters, store directly in member header (padded)
- If longer, the name field holds `/NNN` where `NNN` is a *decimal offset* into `//`
- The `//` member stores all long filenames concatenated, separated by `/\n`($*nix$) or `\0`(windows).
The member need not exist on $*nix$ systems oftentimes, provided there are no long names, but typically placed after the **symbol directory**if it is required. 

##### The alternative overflow method
The name field can have provision for overflow via an alternative method. If the filename exceeds 16 characters in length, the string "#1/" followed by the ASCII length of the name is written in the name field. The file size is incremented by the length of the name, and the name is then written immediately following the archive header. 

#### Symbol Directory (and its variants)
The symbol directory formats haver varied somewhat, but they are all functionally the same.

The symbol directory maps *symbol names* to *member offsets* inside the archives. 

Without the symbol directory, a linker would have to linearly scan every object to find definitions. 

##### 1st Gen
Did not include a symbol directory; a linker was forced to scan all object files in a library to resolve a definition in the source objects. 
##### \_\_.SYMDEF (a.out)
The a.out archives store the directory in a member called `.__SYMDEF`. This has to be the first member in the archive. This member starts with a 4-byte positive integer containing the size in bytes of the symbol tables that follows it, hence
$$ \text{number of entries} = \frac{\text{tablesize (in bytes)}}{8}$$

As each entry is 8 bytes long. Each symbol table entry contains a zero-based offset into the string table providing the symbol's name, and the file position of the header of the member that defines the symbol. The symbol table entries are conventionally in the order of the members in the file. 
```c
int tablesize;
struct symtable {
	int symbol; //offset in string table
	int member; //member pointer
} symtable[];

int stringtsize; //size of string table
char strings[];
```

##### / Directory
ELF archives utilise this scheme. 
The first 4 byte value is the number of symbols. Following that is an array of file offsets of archive members, and a set of null-terminated strings. 
```c
int 32_t nsymbols;
int 32_t member_offsets[nsymbols]; // file offset to start of each member header
char strings[]; //null terminated, one per symbol
```
Usually use native byte order to encode this data.
The first member offset points to the symbol named by the first string, and so on and so forth. 

Say `nsymbols = 3` and the directory describes:

| Symbol   | Member Offset |
| -------- | ------------- |
| \_main   | 0x000000A0    |
| \_printf | 0x000012F0    |
| \_exit   | 0x00002320    |
Thus:

```asm
03 00 00 00 # nsymbols
A0 00 00 00 F0 12 00 00 20 23 00 00 # member offsets
5F 6D 61 69 6E 00                     # "_main"
5F 70 72 69 6E 74 66 00               # "_printf"
5F 65 78 69 74 00                     # "_exit"
```
All concatenated with no padding. 

##### /SYM64/
The `/SYM64/` archive member is simply a 64 bit variant of `/`
It was introduced to handle archives larger than 4GB as the 32 bit `member_offset` and `nsymbol` fields would overflow. The semantics are identical; same ordering and endianness rules, and same logic in that the $i-th$ symbol corresponds to the $i-th$ offset.

Generated automatically by `ar` if the archive exceeds 4GB. 
#### Using AR(GNU) to create libraries
`ar` is used to 'create and maintain library archives'. When an archive consists entirely of valid object files, `ar` formats the archive such that it it is usable as a library for link editing. 

One creates a library by invoking a command like the below:
```bash
ar rcs libtest.a foo.o bar.o baz.o
```
1. r - Replace existing members with the same name, or add new ones
2. c - create the archive if it doesn't already exist
3. s - write the symbol table (/ || /SYM64/) for the link editor
##### Create/Update the Archive
1. `ar` writes the 8-byte magic header.
2. For each input file: 
	Write the 60-byte ASCII member header containing the name, timestamp, UID, GID, permissions, size, terminator. 
	It write's the files contents (the .o bytes).
##### Build Symbol Directory 
If `-s` passed, `ar` constructs a symbol table member at the beginning of the archive. It is constructed as follows.

For every .o in the archive:
- `ar` reads `.symtab` section
- Collects symbols with the following properties: `st_bind == STB_GLOBAL || STB_WEAK && st_shndx != SHN_UNDEF` and notes the file offset for the member that they belong to. 
- It writes the symbol and corresponding file offset to the special archive member (`/` || `/SYM64/`)

##### Modes
Modern ar variants support additional modes:
1. Deterministic mode
	Zero out UID, GID, modification time. 
2. Thin archives (--thin or magic `!<thin>)
	Instead of embedding `.o` files, a thin archive stores relative file paths to the object files. 


## Searching Libraries
After a library has been created, the linker has to be able to search it. The searching of a library occurs after all input files have been processed, i.e., in the linker's first pass after constructing all internal data structures. Once the $GST$ has been constructed (the exact implementation and contents are linker-dependent), the linker can determine unresolved global/weak symbols that remain. The linker must satisfy these by pulling in enough objects from library files. _Linker's extract only the object modules that resolve currently unresolved symbols_

First, the linker must read the library index `/` || `/SYM64/`. This is then parsed into an in-memory data structure like so:
```c
typedef struct {
    char *name;
    uint32_t member_offset;
} ArchiveIndexEntry;

int nsymbols;

ArchiveIndexEntry *archive = malloc(nsymbols * sizeof(ArchiveIndexEntry));
```

Then, for each symbol $S$ in the **GST**:
	If $S$ is undefined
	Check the library index for `S.name`
	If found
		Extract the corresponding member according to `member_offset`
		Parse it as normal input file
			Load its sections
			Load its symbols
			Add them into the `GST`
		Iterate until stable
Library searching is iterative as pulling in one member may introduce new undefined symbols which must then be resolved, often satisfied by other members in the same library. One iterates until a fixed point - no unresolved symbol can be satisfied by remaining members. 

There are few things to note.
1. First definition wins
	When several members define the same symbol, the first pulled defines the final symbol, unless a weak definition is pulled and a strong symbol is later discovered.
2. Circular dependencies
	If a routine in library A depends on a routine in library B, but another routine in library B depends on a routine in library A, neither searching A followed by B nor B followed by A will find all of the required routines. Such dependencies get worse as the number of dependent libraries increases. One can direct the linker to repeatedly search, but this often requires foresight or trial and error and is not very elegant. The solution is to implement group semantics. The linker treats a contiguous range of libraries as a group and keeps scanning the group until no new members are pulled. (`--start-group / --end-group`)
3. Ordering
	The search order of libraries is according to the order in which the inputs appear. 

## Performance issues
The primary performance issue related to libraries used to be the time spent scanning libraries sequentially. Once symbol directories became standardised this was no longer an issue. Library searches can still be slow if a library has a lot of tiny members. Particularly in the now common case that all of the library members are combined at run time into a single shared library anyway, it'd probably be faster to create a single object file that defines all of the symbols in the library and to link using that rather than by searching a library. More on this in [[Chapter 9]].

## Weak External Symbols
Early UNIX linkers maintained a rigid definition/reference model in that:
- Every undefined symbol must be satisfied by a definition somewhere
- Every symbol defined somewhere in a library pulls in the object that defines it
This turned out to be insufficiently flexible for many applications. For example, most C programs call routines in the `printf` family to format some data for output. `printf` can format many kinds of data, including floating point. Linker-over eagerness was caused due to this model; even programs that didn't use floating point ended up linking in floating point libraries simply because `printf` *could* reference `fcvt`.

Originally this was solved by ordering the library such that if the program used floating point, then it loaded the correct floating point libraries reducing unused dependencies. Brittle solution.

Weak references and definitions were introduced to solve this problem.

### Weak References
> An undefined symbol that does not trigger a library load, and is allowed to remain unresolved.

If a real definition exists somewhere, the weak reference resolves to it. 
If none exists, the linker leaves it undefined, typically resolved as zero at runtime. 
It is not an error to leave a weak reference unresolved. 

Thus in the above example, `printf` makes a weak reference to `fcvt`. The floating point code defines `fcvt`. If the floating point code is linked, the strong definition wins. If not, the weak reference stays unresolved and is not an error. 
### Weak Definitions
> A definition that only takes effect if no strong definition exists. 

Less common, but are useful for providing fallbacks and providing an interface. 

```c
__attribute__((weak)) void error_handler() {
	//default stub
}
```
If another object file (either in library or in original linker input) provides a strong `error_handler`, that one overrides the weak one. If not, the stub version is used. 

