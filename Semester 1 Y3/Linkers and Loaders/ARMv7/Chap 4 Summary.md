Linker/Loader first major task is storage allocation, involves establishing coherent model of target address space and determining actual addresses via placement of all code and data segments within final output file.

Most symbols in linkable file, defined relative to respective storage areas, cannot resolve symbols until know these addresses. 

# Segments and addresses
Input files contain a set of segments of varying type. Differing kinds are treated in different ways, all segments of a particular type are often concatenated into a single larger segment in the output file. 

Sometimes, linker itself creates segments and lays them out. Storage layout is generally a two pass process, as location of each segment cannot be assigned until size of all segments that logically precede it are known. 


# Simple Storage Layout
In simples scenario, linker input is set of modules $M_{1}...M_{n}$ each of which consist of a single segment starting at location 0 of length $L_1...L_n$, with target address space starting at 0.

Examine each module, allocating storage sequentially; the starting address of $M_i$ is the sum of $L_1$ through $L_{i-1}$ and length of linked program is $\sum_1^nL_{i}$.

Most architectures require that data aligned on word boundaries, round up each $L_i$ to multiple of most stringent alignment. 

![[Pasted image 20251101211927.png]]

# Multiple Segment Types
Necessitates that linker groups corresponding segments from all input modules, requiring two-level storage allocation strategy.

## Pass 1 - Sizing
Linker reads all input modules, calculates total size for each segment type. Each module $M_i$ has text size $T_i$, data size $D_i$, BSS size $B_i$. Linker allocates space for each of the $T_i, D_i,$ and $B_i$ as though segments were all separately allocated at 0. 

## Pass 2- addressing
Final segment addresses assigned sequentially; data after text, bss after data and text. 

Simply add total size(s) of the modules logically preceding to allocate space. E.g., $T_{total}$ added to address assigned for each of the data segments to compute locations. 

# Segment and Page Alignment
Linkers must round segment sizes up to satisfy alignment constraints. Often means word boundary rounding. In ELF, the section header field `sh_addralign` specifies alignment  constraints. Requires `sh_addr` to be congruent to 0 modulo this alignment value. Segments can start at any offset in the actual file to save space, but the virtual starting address must be aligned.  

Many $*nix$ systms use the trick to save file space that involves starting data immediately after `.text` in object file, mapping that page into virtual memory twice, once **COW** for `.data` and once **RO** for `.text`. 
Data addresses logically start exactly one page(pgsize) beyond the end of the RO text. 

# Common Blocks and other special segments
Common blocks such as those with history in Fortran(block of storage, enables different subprograms to share data without using arguments)(or equivalently, unallocated C external variables) are designed to use **overlaid storage** rather than being concatenated sequentially. 
1. If a common block is declared with varying sizes across multiple files i.e., same name, reserve space based on largest instance
2. Linkers traditionally handle uninitialised global vars and common blocks by assigning space in **BSS** segment of output file.

Other special segments include:
- **Initialisation and Termination**
	- `.init` and `.fini` sections hold exec instructions run at process startup and termination, C++ requires these. Related sections include arrays of function pointers `.{name}_array` that exact startup and shutdown code. Thee array sections often have `SHF_ALLOC` and `SHF_WRITE` set.
		- The reason for `SHF_WRITE` is to accommodate dynamic relocations
			- During program startup, dynamic linker must modify certain memory areas, to allow this, placed in RW data segment, once all relocations completed, parts of that segment no longer meant to be modified, and as such are collected into `GNU_RELRO`. Initially this is overlaid, but linker calls mprotect to mark this subsegment of the RW segment as read-only when dynamic relocations are finished.
- **Read only data
	- Sections like `.rodata` and `.rodata1` contain read-only data intended to contribute to non-writable segment, `SHF_ALLOC`.
- **Thread local storage**
	- If supported, sections like `.tdata` and `.tbss` hold data replicated for each execution thread. 
- **Section Groups**
	- Sections of type `SHT_GROUP` define set of related sections that must be treated as a unit during linking, often appearing only in in relocatable objects `ET_REL`. 
# Linker Control Scripts
Detailed control over arrangement of output file and assignment of addresses, often need for embedded. 

The GNU linker `ld` accepts a configuration file `.ld` to permit control over how sections from object files are combined and mapped into final executable. Control alignment, placement, define where data is loaded from. 

There are `MEMORY` blocks which define memory regions and where the start. There are `SECTIONS` which declare where code and data, and other potentially loadable segments, are to appear in memory. The `ENTRY`(symbol) command sets program entry point, predicated on the value of the symbol. Gets much more complex. 


# Unix a.out Storage Allocation
Not exceedingly complex; the set of segments is known in advance, each input file has `text`, `data`, `bss` segments and perhaps common blocks disguised as external symbols. 

Linker collects sizes of the text, data, and bss from each of the input files via the header. 

Simply allocates the storage and places in output file as described earlier, and assigns addresses in accordance to a.out format being used e.g., NMAGIC vs QMAGIC with header considered part of text segment and mapping program at start of second virtual page, cluster headers, cluster text, start data after text according to format specification, and deal with bss accordingly. 
# ARM7TDMI and ELF Storage Allocation
ELF storage allocation considerably more complex than predecessor a.out objects. 

Of course, ELF has unique nature, loadable sections emitted as segments that will actually reside in memory. ELF maintains dual nature, organised on disk into ELF header, program header table(optional for linking view), section header table(optional for execution view) and the actual section/segment contents. 

When ELF loaded, creates more mapped regions consisting of file-backed pages described by program headers. 

To coalesce input sections from ELF objects such that a coalesced ELF executable object can be produced, a number of steps are taken. 

1. Collect all Sections from inputs
	Linker pulls together all input sections by type: `.text, .rodata, .data, .bss` etc
2. Classify sections by runtime properties
	Determine `SHF_ALLOC`
	Determine access flags(RWX) `SHF_WRITE`, `SHF_EXEC_INSTR`
	Determine alignment requirement and size `sh_addralign`, `sh_size`
	Determine whether loaded from file, or can be zeroed in memory (BSS)
3. Form permission groups/segments
	This is the initial grouping via runtime page permissions according to headers:
		Read+Execute - `.text`
		Read Only - `.rodata`
		Read+Write - `.data`, `.rel/a`, `.init_array` etc
		Read+Write but zero initialised - `.bss`
			Avoid creating pages both W and X for security purposes
4. Order sections within each group
	Linker picks conventional ordering within each permission group such that they can be arranged logically:
		 ![[Pasted image 20251106121541.png]]
	ensures predictable addresses and ensures early executing code like `.init` lies close to main `.text`
	Related sections kept contiguous to exploit spatial locality. 
	Segments are identified by segment type constants in the ELF specification such as `PT_LOAD`, `PT_DYNAMIC`, `PT_GNU_RELRO`
	Typically only a few `PT_LOAD` segments e.g., RX, RO, RW. 
5. Assign Virtual Addresses and File offsets
	Assume a page size of 0x1000
	Assume a load base address of 0x00008000
	Start with first loadable segment e.g., RX
	Align to next 4KB boundary
	Assign p_vaddr = base_addr
	Assign p_offset = file_offset_after_headers
	For each section in this segment:
		Align its start to alignment requirement
		Place it sequentially in memory
		Advance `current_addr` and `current_offset` accordingly.
6. Can be done during all of this, but one needs to populate program headers as go along.

Thus all sections are now in segments, at the discretion of the linker, whilst trying to directly reflect the hardware execution environment. 
During the above process, the linker also notes which symbols will be resolved at run time from shared libraries and creates .interp, .got, .plt, and symbol table sections to support run-time linking.

![[Pasted image 20251106122911.png]]



# Exercises
1.  Why does a linker shuffle around segments to put segments of the same type next to each other? 
	As i understand it, there are a few reasons. One, spatial locality, two security, three internal fragmentation issues. Spatial locality is the notion that if a particular memory location is accessed, its likely that nearby memory locations will be accessed soon. For example, coalescing .text, which will often call one another, means fewer cache misses. Internal fragmentation reduced by aligning by segment type, as will have same alignment requirements. Further simplifies relocations as they can be computed to a single base, rather than scattered bases, adding extra complexity to linker. Security, by enforcing page permissions such that a segment is never Write and Execute, attack surface is reduced. If the linker were to place .data in the same segment as .text, this would be catastrophic. 

2. When, if ever, does it matter in what order a linker allocates storage for routines? In our example, what difference would it make if the linker allocated newyork, mass, calif, main rather than main, calif, mass, newyork?
	From a semantic perspective, there is no issue, although from a performance perspective, one can consider spatial locality. Say these functions are all laid out somewhat consectively, when main is fetched, the next few function bodies are likely within adjacent cache lines. Enforcing simple rules like this where possible speeds up program execution. Can also enforce fewer TLB misses if in same page, and enforce smaller working set. 

3. In most cases, a linker allocates similar sections sequentially, for example, the text of calif, mass, and newyork one after another. But it allocates all common sections with the same name on top of each other, why?
	In the context of C, unitialised symbols are global variables that are not yet placed in any section, linker sees multiple common symbols, emits common symbol entries in the symbol table, and treats them all as single variable, merging them into a single allocation in .bss, resolved to same address. Uses the largest size amongst all definitions, and the strictest alignment. This saves memory, otherwise each module's reference would go to its own address; use one memory slot and the program behaves as if theres truly one global variable as intended. 
	Furthermore, it turns out common symbols behave simiarlly to weak symbols, they can be overridden by a strong definition in another object or a shared library, e.g., int buffer_size, int buffer_size = 1024, allowing interfacing with libraries without changing code.

## ASLR
### Classic Model
### Middle of Address space model

### How ASLR is exerted

