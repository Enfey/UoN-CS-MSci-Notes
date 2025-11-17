# Relocation
Assemblers emit object files lacking information such as where each sections will end up in executable image, where other object files' symbols will be placed, and whether a symbol will be forcibly moved into the $GOT$. 

No ahead of time knowledge of virtual address assignments that correspond to the planned memory image means that relocations are emitted by the assembler to refer to positions where certain instruction fields and data values cannot yet be computed. The assembler emits placeholder bytes, and a corresponding relocation entry according to the object format in use to permit adjustment when a semblance of the memory image has been computed. 

To give a concrete definition of relocation:
> Relocations are transformation directives encoded in a relocatable object file (position dependent) that specify that the machine value stored at location $(s, p)$ is not fixed and must be computed by applying a relocation-type-specific transformation $F_t$ to the resolved address of the symbol $\sigma$. The set of all relocations defines the incomplete semantic state of a relocatable object and imposes constraints on valid program instantiation. 

It should be noted that even in the case of absolute references, it is not common for architectures to hold a $word$ displacement. Thus, $F_t$ is rarely the identity function, even in the case of absolute relocation (there is usually some sort of truncation or manipulation, but this is dangerous and very finnicky for the obvious reasons).
## Hardware and Software relocation
### Hardware Relocation 
> Any mechanism provided in the CPU/MMU that automatically translates the addresses used by a program into other addresses at runtime, without modifying the machine code. 

The program believes it is loaded at address zero (or some fixed base 
$B$), but the hardware adds a base/offset to reach real memory locations.

#### Base and Limit Registers
The classic example is base and limit registers. A base register holds the start of the program address, and the limit register enforces the bounds of valid references. Every memory reference becomes
```c
physical_address = base_register + logical_address
```
Runs if always loaded at logical address 0 and provides each process with one contiguous logical address space. 
```c
if logical_address > LIMIT → fault
```
#### x86 Segmentation
Each code/segment has multiple segments, each with a base, limit, access permissions, type flags etc. The semantics are not necessarily important, but each memory access on protected mode x86 goes through a pipeline:
```C
logical address (selector:offset)
        → segmentation
linear address (base + offset)
        → paging
physical address
```
logical address yields offset and segment register, offset is added to segment base to yield a linear address, which has a frame number interpolated to access the referred byte in memory. 

#### Paging/Virtual memory
In modern computing, relocation is achieved via page tables and the TLB. The kernel is responsible for providing the page to frame mapping. The MMU walks this page table and performs appropriate interpolation for memory accesses such that they are valid. Though, a large portion of this is software related(page faults, page replacement, paging daemon etc), there is hardware support and is classed as such. 

### Software relocation
> Process via which linker or loader enacts relocation entries to facilitate patching of code such that addresses that depend on where symbols or sections are placed in memory will function as intended. 

Many instructions contain incomplete or symbolic references; relocation entries are emitted by an assembler, often for calls to functions that have not been placed in the object context, accesses to global variables with no fixed address yet, PC relative displacements whose relative distance is unknown and for GOT/PLT entries that haven't been allocated.

Relocation entries vary according to object format. They generally describe the section/segment relative offset at which to apply the relocation $P$, which symbol is referred to $S_{\sigma}$, and what relocation type to apply $F_t(S_sigma, P, A)$. 

Despite the prior existence of complex hardware relocation schemes like say, providing a segment per routine and global datum in x86 and treating references as intersegment references to be looked up in the system's segment tables and bound at runtime, this would be very slow, much slower than linking conventionally.  (although run-time binding is of course useful, it is best to avoid it where possible(according to Levine)).

Paging and virtual memory provides different benefits; while what it performs is indeed relocation and provides a whole host of benefits, namely security, individual logical contiguous address spaces, extreme support for high-degree of multiprogramming given strong paging policies etc, it does not fix addresses inside instructions. It is not privy to symbol names and program structure. They each solve an independent problem, and do it well, such is the nature of good design. 
## Link-time and Load-time relocation
### Link-Time Relocation
Occurs during static linking - object file inputs are coalesced into an output executable or shared library. This includes the typical stages of parsing, binding and name resolution including library searches, storage allocation, and code relocation to modify $RX$ segment before emitting an executable.

Examples of link-time relocations include:
- Global variable references
- Static TLS offsets(local-exec model(see [[Thread-local Storage]]))

Static executables generally do not need load-time relocation. There are two exceptions to this rule. 
1. They contain absolute relocations that must adjust for ASLR
2. If a statically linked program $p_{i}$ shares its address space with one or more programs $P = \{p_1, p_2, ... , p_n\}$ and is linked to load at a fixed address which is not available at load-time, then the binary must be relocated to wherever there is free space.
### Load-time relocation
#### Static executable
Load-time relocation here simply refers to the adjustment of the load-base where the original fixed load address was not available as determined at load-time. The binary is relocated to this new load-base. 

At load time, in bare-metal systems with no per-process address space and no hardware relocation, statically linked executables are treated as one big segment. Load-time relocation is then simply adjusting all addresses by one delta. 
```C
A_new = A + 𝛿
```
#### Dynamic loading
TO COMPLETE.
## Symbol and segment relocation
### Symbol relocation
> Symbol relocation refers to adjusting machine code or data that contains symbol references, via application of a relocation that will provide the final resolved address of that symbol.

It involves a relocation entry (`.rel/a{name}`) which refers to a symbol in `.symtab`. Thus`.symtab` (or more accurately, the internal linker symbol table(enriched with a pointer to the definition in the $GST$ for `st_shndx == UNDEF` symbols)) defines the symbol by providing an address. The relocation formula for the relocation type is implied, and the result of this is computed to provide the reference to the symbol's final address, and subsequently patched in. 

The linker computes:
	$S$ = resolved address of `symbol` $\sigma$
	$value = F_t(S, P, A)$ ; *according to relocation type*
	write $value$ to $P$ 

Things requiring symbol relocation includes calls to external functions, loading an address of an external variable, TLS references and GOT references in PIC code.
### Segment relocation
> Segment relocation refers to relocating a whole program or large region of memory by adjusting all absolute addresses by a single delta when the program is loaded at an address different from its link address. 

This is by and large, no longer necessary due to private logical address spaces and things like position independent executables. In ELF symbols of type `STT_SECTION` do appear, and their primary purpose is for relocation, and typically have `STB_LOCAL` binding. 

For example, could emit less relocations by placing global variables in `.data` and emitting this as a symbol, and accessing symbols via `.data + known offset at compile time` where the `value` of data goes in some register. This is an alternative to creating relocations for each variable usage, simply need a relocation for `.data`. This relates to the idea of anchored addresses, a compiler optimisation which allows sharing addresses instead of computing separate symbolic addresses.
## Basic Relocation techniques 
### a.out

Each object file has two sets of relocation entries. One for the `text` segment and one for the `data` segment. Each relocation entry contains a bit `r_extern` that specifies whether this is a segment-relative or symbol relative entry. If the bit is clear, it's segment relative and `r_symbolnum` is actually a code for the segment: `N_Text(4)`, `N_DATA(6)`, `N_BSS(8)`. The `pc_rel` bit specifies whether the reference is absolute or relative to the PC. 

At link time, base addresses are computed for each segment, and every symbol within such segments in an `a.out` file are relative addressed. For an intra-segment pointer/absolute address, the linker adds `TR` or `DR` to the offset within the segment to correct the pointer.  For an inter-segment pointer/absolute address, the linker adds the relocated base (`TR` or `DR`) of the target segment to the offset, which is an offset emitted relative to the target segment.

### ELF
### Data relocation


### Instruction relocation


### Background
ARM7TDMI executes branch instructions PC-relative and accesses data using absolute or relative addresses. ARM ELF defines a rich set of relocation types tailored for a number of different use cases. 
TO COMPLETE: GOT, TLS etc

### ELF relocation

## Relinkable and relocatable output formats

## Other relocation formats
### Chained references
### Bitmaps
### Special Segments


