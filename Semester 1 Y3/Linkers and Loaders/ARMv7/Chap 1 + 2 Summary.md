### Linker
A linker is a program that receives one or more object code files, and combines them with the necessary libraries to create a binary executable. 
Resolves references between different files, connects compiled code to precompiled libraries, and performs code relocation. 

Must maintain sanctity of target address space rules e.g., alignment, segmentation.
#### Two-Pass Linking
General structure of linkers:
1. Input - command line args, object files, linker control files, shared libraries, normal libraries, each input file also contains symbol table.
2. Output - debug symbol file, executable file, link/load map 

1st pass scans all object files and find sizes of segments to be placed in output file, finds symbol definitions, finds symbol references. Builds a symbol table, and a segment table. Using this data, determine size and locations of the segments in output address space, figure out placement in output file e.g., merging segments. 

2nd pass uses the data, read and relocate object code, replace symbolic references with actual memory address, with respect to relocated segment addesses, and write to file. If using dynamic linking, symbol table contains info runtime linker will need to resolve dynamic symbols. Some object formats are relinkable.
#### Linker Command Languages
Every linker, some sort of command language, can be used to control linking process. Generally long list of options, whether to keep debugging symbols, whether to use static or shared libraries, which of several output formats to use. Most linkers permit some way to specify the address at which the linked code is to be bound, useful when using a linker to link a system kernel or program that will not pass control to OS.

Include commands via either DSL, embedded in object file, intermixed with object file, or via command line arg as mixture of file names and flags.

#### Code Relocation
- Compilers + assemblers emit object code with program addresses often starting at address zero, but do not load into memory at location zero.
- If a program is created from multiple subprograms, must load at non-overlapping addresses.
- Relocation is process of adjusting addresses i.e., providing load addresses so that it can execute correctly when loaded at a memory address different from the one it was originally compiled for. Involves alteration of both instruction and data references, with respect to the addressing mode of the architecture and the instruction format. 
#### Symbol Resolution
Compiler identifiers symbols and emits object code with placeholders for addresses. Then perform symbol resolution between object files, finding placeholders defined across different modules and assign concrete memory addresses such that external symbol references are valid.

The distinction between relocation and symbol resolution is murky. Linkers can already resolve references to symbols, thus one way to handle code relocation, is to assign a base address symbol to the base address of each part of the program, and treat relocatable addresses as references to the base address symbols. Just now needs to calculate relocation for the start of each segment rather than entire program, and the rest of the addresses are expressed relative to that base.
#### Program Addressing
Refers to conventions regarding memory layout, data alignment, and overall address space available to program. Conform to defined address space architecture.
##### Byte Order and Alignment
Byte order or endianness determines how a multi-byte value e.g., word, halfword is stored in memory i.e., the sequence in which individual bytes are addressed. Big endian is where the most significant byte of a word is stored at the lowest numbered byte address, and the least significant byte is stored at the highest numbered byte address. Little endian is where the most significant byte of a word is stored at the highest numbered byte address, and the least significant byte of a word is stored at the lowest numbered byte address. 

Alignment refers to the principle of storing data or instructions at addresses that are multiples of their size or a specific boundary. Misaligned access either causes exceptions or slows access; worth maintaining. Struct padding and other compiler tricks and proper relocation schemes preserve instruction and data alignment in memory. Any $n-byte-datum$ must have at least $log_2(N)$ low zero bits. 
E.g., log2(4) = 2, has to end with at least 2 low bits of address e.g., 0x00, 0x04, 0x08. 0x0C etc. 
##### Address Formation
Refers to the specific rules and mechanisms a given hardware architecture uses to calculate final memory address referenced by an instruction. Quite precise; must be maintained by linkers when patching object code to match addressing scheme. Constrained by address fields in instructions, direct addressing not always possible. Either use direct addressing, register indirect addressing(address in register), or based/index addressing(offset + constant + optional index)

Can also use relative addressing - instructions contain offsets relative to the segment base address they belong to or contain relative addresses which are added to the PC or some other instruction. These relative addresses historically represent the number of halfwords or words to jump, to calculate number of bytes when it comes time to calculate, a number of left shifts must be performed to ensure a valid address is formed. IBM 390 used 16 bit signed offset in address field shifted 2 bits left and added to PC to calculate jump target. 


Can also use pseudo-direct addressing, often on RISC. Use multi-instruction sequence like in SPARC e.g., a SETHI instruction which loads its 22-bit immediate value into the high 22 bits of a register and zeros the lower 10 bits, which is then followed by OR immediate which ORs its 13 bit immediate value into the low part of the register. Linker must handle specific relocation modes to arrange to put the high and low parts of the desired 32 bit address into the two instructions.
#### Instruction Formats
Each instruction, consists of opcode and operand; operand may be immediate, or located in memory. Address of each operand must be calculated according to address formation rules for that architecture.
##### Instruction Length
Architectures differ in their instruction size, which affects parsing and modification efforts. Fixed vs Variable length instruction size.
- Fixed length: $SPARC$ instructions are always four bytes long
- Variable length: $IBM 370$ instructions can be 2, 4, or 6 bytes long. x86 instructions can range from one byte up to 14 bytes. 
	- From POV of linker writer, the address and offset fields that linker has to adjust occur on byte boundaries, thus linker does not need to be concerned with the instruction encoding.
NOTE: ARM7TDMI, executes 32 bit word aligned instructions when not in thumb mode, otherwise execute 16 bit halfword aligned instructions. 

Bootstrapping issue on architectures without support for direct addressing.
#### ABI
> An application binary interface is defined as set of programming conventions that OSs ensure applications conform to, in order to run under that system.

Cannot violate ABI. ABI specific to hardware architecture. Linker has to do significant work to achieve ABI compliance. 

##### ABI components
- Processor instruction set, memory access types and address formation etc
- Size, layout, alignment of data types processor can directly access.
- Calling convention - controls how args of functions are passed, return values retrieved e.g., how call stack is organised e.g., whether all parameters passed on call stack, or some passed in registers, which registers used for function parameters, whether callee or caller is responsible for cleaning up call stack after function call. 

In many cases the linker often has to do significant part of the work involved in ABI compliance. E.g., ABI requires that each program contains table of all the addresses of static data used by functions in program, linker will create that table by collecting address information from all the modules linked in.

##### Procedure calls
Every ABI defines standard procedure call sequence, relies on combination of hardware defined instructions and conventions regarding register, stack, and memory use. 

A hardware calls instruction saves return address and jumps to called procedure. On architectures with hardware stack, such as x86 ESP, return address is pushed there, on other architectures, saved in register(may need to be committed to memory via PCB if ISR invoked). Usually corresponding hardware return instruction, pops return address from stack and jumps to that address.

Within procedure, data addressing falls into 4 categories:
1. Caller can pass arguments to procedure
2. Local variables allocated within procedure, freed before return
3. Local static data stored in fixed location in memory, private to procedure.
4. Global static data stored in fixed location in memory, can be referenced from many different procedures.

Piece of stack memory allocated for single proceure call is known as a stack frame. Allocate arguments and local variables and return address here. One register, serves as stack pointer. Another register, frame pointer, point to base of current function's stack frame, can address arguments and variables and return address at fixed offset as this stays constant.
![[Pasted image 20251019020805.png]]

To access static data, compilers and linkers often cooperate to maintain an address pool ( a pointer table). This address pool contains pointers to all static objects referenced by the routines in a module. Involves designating a special register, the table pointer, points to base of address pool. Compiled code uses relocated base addressing with regard to table pointer register, load into another register, and then use that as register indirect addressing to perform operation. But have to get the address of the able into first register, essentially becomes callers job to load table pointer. Covered later. 


##### Paging and Virtual Memory
Paging uses fixed partitioning and relocation to devise a non-contiguous memory management scheme. Divides program address space into fixed size logical pages, and divides the physical memory of the computer into page frames of the same size. 

We use a page table to store mapping between virtual pages and physical frames.

A virtual address is relative to start of program, or start of a segment. Leftmost $n$ bits represent the page number, rightmost $n$ bits represent offset within the page. In a 4KB block, 4 bits used for page number, 12 bits used to represent the offset within the page to address all bytes. 

The frame number is a function of the page number, offset kept as is when accessing memory.

![[Pasted image 20251019232419.png]]
Page table entry consists of:
present/absent bit if frame is in memory
modified bit
RWX protection bits
frame number
user/supervisor bit

Move pages back and forth as needed between main memory and disk as needed, virtual memory. If page fault and consequent page in or page out occurs, take severl ms. More page faults = slower, worst case begins thrashing(performance problem occurs when kernel spends majority of time swapping pages between RAM and disk, can be prevented by good page replacement policy, paging daemon, reducing degree of multi-programming). If linker can pack related routines into a single page or into small set of pages, paging performance improves. If pages marked as RO, performance improves to by eliminating I/O cycles.

An x86 with 32-bit addressing and 4KB pages would need a page table with 2$^{20}$ entries to map an entire address space. Because each page table entry is usually 4 bytes, this would make the page tables an impractical 4MB long.

Provoked advent of multi-level page tables. Organise virtual address space into hierarchal page tables. Entry of lvl1 page table are pointers to a level 2 page table and so on and so forth. Entries of innermost page table(s) store frame info. Only outermost table in main memory, pulls other tables into memory when required. Page number now divided into index into root page table, followed by subsequent indexes to specified depth to reach frame mapping. Often use TLB to optimise virtual address translation (can be searched in parallel.)


##### Memory Mapped Files
Virtual memory systems often unify the paging mechanism with the file system via **mapped files**. A memory mapped file is a portion of virtual memory assigned a byte-for-byte correlation with some portion of a file or file-like resource(anything referenced via a file descriptor.)

When a program maps a file to part of program address space, all pages in that segment marked as not present, and uses the file as a paging disk(backing store) for that part of the address space. Can read the file merely by referencing that part of the virtual address space. 

Can map file as RO, RW (written back by time file is unmapped), COW(copy on write, map each page as RO until written to, at which point creates private copy of page for that process.)

NOTE:(object files never contain frame references, but still need relocation as each obj file is compiled to load at 0x0, all absolute references would collide, must organise tidily so that virtual address references within the program is made consistent). Then OS decides which frames will back which pages. 

## ARM7TDMI
Target architecture for my linker. Poses unique architectural challenges and advantages that will affect the design and implementation of the linker seeking to target this core.

### Instruction sets and architectural states
Supports two distinct instruction sets and operating states:

1. $ARM$ State: Executes 32 bit, word-aligned ARM Instructions
	- 4 byte long instructions
	- Processor operates in 32 bit mode accessing 32 bit registers and a 32 bit address space.
	- All exception handling is entered in ARM state. If an exception occurs in Thumb state, the processor reverts to ARM state.
2. $Thumb$ State: Executes 16 bit, halfword-aligned Thumb Instructions. 
	- Provide higher code density (typically 65% of ARM code size)
	- 2 Byte Long Instructions
	- PC uses bit 1 (0-1) to select which halfword of a 32 bit word is used regarding address references.
Transitioning between ARM and Thumb states occurs using the Branch and Exchange(BX) instruction.

A core task for the linker then, is managing code sections generated for both ARM and Thumb states, ensuring correct alignment upon relocation and generating the correct glue code for inter-state calls using the BX instruction. When transitioning between states, linker must ensure target address aligns with the requirement of the target state. The operating state is signalled externally by the TBIT signal (LOW for ARM, HIGH for Thumb).

(more detail on actual instruction forms, e.g., address field size, offset and bit shift, sign extensions etc)
### Address Bus and Memory Interface Signals
ARM7TDMI uses a Von Neumann architecture with a single 32-bit data bus.

- $A[31:0]$ - 32 bit address bus providing 4GB linear addressable space. Linker must ensure all generated program addresses fit within this space (logically) and respect memory layout requirements.  There can only be 4GB of actual physical memory that is addressable, and there can be up to 4GB of virtual address space for each program. Linker ensures all symbol addresses fit in 0x00000000 – 0xFFFFFFFF.
- $MAS[1:0]$ - Encodes the size of the transfer (byte(00), halfword(01), or word(10)). Used by memory controllers and implicitly by linker to verify alignment. i = 2 for ARM state and i = 1 in Thumb state
- $nTRANS$ - Indicates the privelige level of access (0=user, 1=privileged). If a memory system requires privileged access, the linker must ensure the code is structured to reflect this, e.g., preserving instructions like LDRT and STRT that force User mode access from priviliged mode. 
(more detail)
### Alignment and Natural Boundaries
Enforces strict alignment requirements. Words must be aligned to 4 byte boundaries, halfwords must be aligned to 2 byte boundaries. Instructions in ARM state must be word aligned, and instructions in Thumb mode must be halfword aligned. Linker must insert padding or ensure allocation boundaries are preserved when performing code relocation. 

Biendian support: processor is bi-endian, must support the endianness configuration, a decision deferred to hardware config via the BIGEND signal [0-1]. 
### Addressing Modes and Register-Based Addressing
The PC is heavily involved in address formation. R15 holds the PC.

Branch instructions (B and BL) calculate the branch target address **relative** to the PC. 

When an instruction is executed, the PC holds the address of the instruction plus 8 in ARM state (or +4 in Thumb state)

For BL instructions, the linker needs to ensure the branch offset (a 24 bit immediate offset, shifted left by two bits for word alignment in ARM state), is sufficient for the destination. If distance is too great, the linker must introduce glue code or indirect branching via a literal pool (type of address pool). 

Load and Store instructions (LDR, STR) use base registers (Rn) combined with an immediate offset, a register offset, or a scaled register offset.

The small offset sizes means that if a symbol is located far from the instruction referencing it, a single instruction cannot address it directly. The linker must handle intricate relocation processes for addresses outside the immediate reach of PC relative addressing for branch instructions. Must identify target address, and substitute the original single instruction with an intermediate table lookup or sequence of instructions to construct full 32 bit address.
### ABI and Register Conventions affecting linking
ABI dictates calling conventions, register usage, exception handling.

- Link Register - Used for subroutine return addresses (BL writes to R14). Linker must ensure the function epilogue uses R14 correctly to return, (MOV PC, R14) in ARM state. 
- Stack Pointer - R13 used implicitly for stack operations (LDM, STM). Linker ensure correct usage of the SP according to the ABI.
- Exceptions/Mode registers - Has multiple operating modes e.g., User, Abort, Supervisor, Undefined, each with banked registers (multiple copies of registers that are accessible depending on the processor's current execution mode(R13, R14, SPSrs)). 

### Exercises
1.1 : What is the advantages of separating linker and loader into separate programs? Under what circumstances would a combined linking loader be useful?

One could postulate that the advantage lies in their simple distinction of responsibilities. A linkers inputs and outputs and transformations differ in nature to a loader, although they are part of the same pipeline. Linkers perform relocation, patching object code according to relocation entries, often express addresses relative to some base, such as relocating segments and using all addresses relative to those relocated segments. In modern computers, programs are entitled to their own logical address space, often starting at 0x0. Older systems which link programs to load at 0x0, may have the loader perform final relocation. Also DLLs and such.

Combined linking loaders would have been useful in archaic machines, before the advent of OSs, as monoprogramming was prevalent, could be linked to load at fixed memory addresses each time, the distinction was not exactly needed, thus the program could do symbol resolution, relocation, and loading in one fell swoop. 

1.2: Nearly every programming system in the last 70 years or so includes a linker. Why?
Linkers permit modularity and reuse via permitting programs to be built from modules rather than a monolithic program. The fundamental job of a linker is bind abstract names to more concrete names. The evolution of programming languages and systems increased the complexity of name management and address binding, which gave rise to this component of the execution pipeline; encompassing features like dynamically linked shared libraries, name mangling, storage layout and ABI compliance etc. 

1.3: Linkers in interpretive systems
Java Virtual Machine (JVM) Linking Model: Java employs a sophisticated loading and linking model. Java organizes a program into **classes**, usually compiled into separate binary object code files.

Java loads and links **one class at a time**. If an initial class refers to others, those classes are **loaded on demand when they are needed**. This process is known as **resolution**, which is analogous to dynamic linking in other languages. Resolution includes resolving symbolic references in a classes constant pool to other classes, fields, or methods, which occurs after the class is loaded. 


2.1a) In a CALL instruction, the high 2 bits are the instruction code, and the low 30 bits are a signed word offset. What are the hex addresses for X, Y and Z

X = 01 instruction code, 00 0000 0000 0000 0000 0011 0000 0000 in binary
256+512 = 768 words << 2  = 3072 = 0xC00 in hex
Address 0x1000 + 0xC00 = 0x1C00
MSB is 0 so know it is positive jump.



Y = 7F FF FE ED = 0111 1111 1111 1111 1111 1110 1110 1101
Instruction code  = 01 thus 30 bit offset = 11 1111 1111 1111 1111 1110 1110 1101 = 0x3FFFFEED
0x3FFFFEED−0x40000000=−0x00000113 (0x4... is the modulus)
Byte offset=(−0x113)×4=−0x44C
0x100C−0x44C =0x0BC0

2.1b)The CALL Z instruction at location 0x1010 targets address 0x1018. 

The instruction at 0x1018 is: `SETHI r1,3648367`. This is the first of a two-instruction sequence designed to load a full 32-bit address into register r1 to perform register indirect addressing, rather than PC/Instruction relative addressing. 

2.1c)



2.3) A linker of loader does not need to fully understand every instruction in the target architecture's instruction set. Exquisitively sensitive to program addressing and instruction formats, rather than the instructions themselves. The linkers job concerning instruction involves code relocation, modifying address fields and ensuring that such modifications ensure ABI and addressing mode compliance. Only if a new instruction format is introduced, must the linker need to be changed. If new addressing modes are added, generally the linker would need to be updated, although this depends on the nature of the addition. E.g., if the new modes define addresses that cannot fit within existing relocation structures, or if the calculation logic itself changes (e.g., changing from 16-bit segmented addressing to 32-bit flat/paged addressing), the linker would need updates to handle the complex encoding.

2.4) Linking for a word addressed architecture differs to linking for a byte addressed architecture primarily in how addresses are calculated, stored+aligned. In a word addressed system, offsets used within instructions refer to whole words, simplifying calculations. In a byte addressed system, offsets still represent words (saves $n$ bits per instruction, and guarantees alignment) and have to be shifted left to derive the byte offset before using it in address formation. 

e.g., arm, **When a word access is signaled the memory system ignores the bottom two bits, A[1:0], and when a halfword access is signaled the memory system ignores the bottom bit, A[0].**

All data values must be aligned on their natural boundaries. All words must be word-aligned.

2.5) Inherently difficult, as linkers are exquisitely sensitive to architectural details of addressing modes and instruction formats. The complex nature of ISAs mean that address encoding, alignment rules, necessary relocation methods vary widely from ISA to ISA e.g., SPARC requires  specialised relocation types to synthesise a 32 bit address (SETHI and OR) for register indirect addressing. Creating such a linker where this detailed, and foolproof patching logic is confined to 'changing a few parts of the code' would be remarkably difficult. 

A **multi-target linker** (one that can handle code for various different architectures, though not simultaneously) is considerably easier to build and is common in practice. Such a linker can achieve generality by having modular components to handle architecture-independent tasks like symbol binding and segment allocation, and separate, specialized components for architecture-dependent tasks. For example, the GNU linker handles a vast range of object file formats, machine architectures, and address space conventions, often relying on a DSL input to provide target details. 


