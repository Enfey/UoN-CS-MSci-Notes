Linkers and loaders, compilers and assemblers, all extremely sensitive to architectural details, both the hardware architecture and the architecture conventions required by target OS.

Chapter focus on architectural issues affecting linkers.

Two aspects of hardware architecture affect linkers:
1. **Program Addressing**
	Refers to conventions regarding memory layout, data alignment, and overall address space available to program, affects linkers as first major task for linker is storage allocation, determining where in output address space all program segments will reside. Affects address binding and code relocation as symbols often defined relative to segments, so absolute addresses of base address of segment must be determined to resolve the rest of them. 
	Linkers must also conform to defined address space architecture e.g., width of the address bus, ensuring different libraries are mapped to non-verlapping addresses. Must also conform to data alignment.
2. **Instruction formats**
	Specify how addresses or offsets are encoded within instructions themselves, dictating how linkers must modify code during relocation. Linkers must abide by instruction formats when patching object code to ensure any modifications to addresses or offsets match the addressing scheme and do not result in an invalid instruction. 
	Must also handle limited address fields, many modern architectures like RISCs have instructions that do not contain enough bits to hold a full direct address. Black magic clearly required to execute instructions to derive a full address. Alternatively, linker may organise all addresses into an address pool, and subsequently modify instructions to refer to the appropriate entry in that pool 


### Application Binary Interfaces
> An application binary interface is defined as the set of programming conventions that operating systems ensure applications follow in order to run under that system. 

From the perspective of an application, the ABI is considered just as crucial as underlying hardware architecture, as violating the constraints of either the ABI or the hardware architecture will result in catastrophic failure. 

An ABI is specific to a hardware architecture, and may be OS independent.

The linker often has to do significant part of the work involved in ABI compliance.
#### Components of an ABI
- Processor instruction set with details like register file structure, [memory](https://en.wikipedia.org/wiki/Computer_memory "Computer memory") access types, etc.
- Size, layout, and alignment of basic data types that the processor can directly access
- Calling convention - controls how arguments of functions are passed, return values retrieved e.g., how call stack is organised, whether all parameters passed on call stack or some are passed in registers, which registers are used for which function parameters, whether caller or callee is responsible for cleaning up call stack after function call. 
- System calls - may include a subset of system calls, or at the very least, include the technique necessary to make syscalls to the kernel.

In many cases the linker often has to do significant part of the work involved in ABI compliance. E.g., ABI requires that each program contains table of all the addresses of static data used by functions in program, linker will create that table by collecting address information from all the modules linked in.


### Memory addresses
Of course! Every computer includes main memory, invariably appearing as an array of addressed storage locations determined by the word size/width of the address bus.

#### Byte order and alignment
Each location consists of fixed number of bits. Location sizes historically have varied, but now identify a single byte of storage. A significant portion of data worked with, including memory addresses funnily enough, will exceed this byte, thus adjacent bytes must be grouped together. 

The way this data is stored is dependent on **endianness** of the underlying hardware; **endianness** describes the way multi-byte data like halfwords or words are stored in memory, determining the sequence in which the individual bytes are addressed. 
	NOTE: ARM7TDMI is bi-endian, thus can be configured to store words in memory using either little or bi endian. Using the BIGEND configurtation input signal 'high' for big-endian, and 'low' for little endian. 
**Big endian** is where the most significant byte of a word is stored at the lowest numbered byte address, and the least significant byte is stored at the highest numbered byte address. (leftmost byte=MSB, rightmost byte is = LSB) (right to left)

**Little endian** is where the least significant byte is placed at the lowest numbered byte address, and the most significant byte is placed at the highest numbered byte address.


![[Pasted image 20251018232452.png]]


Multibyte data must usually be aligned on a natural boundary. That is 4 byte data should be aligned on a 4-byte boundary, 2-byte on a 2-byte, and so on and so forth. Any $n-byte-datum$ must have at least $log_2(N)$ low zero bits. 
E.g., log2(4) = 2, has to end with at least 2 low bits of address e.g., 0x00, 0x04, 0x08. 0x0C etc. 

Misalinged data references work at the cost of either reduced performance, or even a program fault. Things like struct padding enforced by the compiler help maintain alignment.

Each architecture defines a number of special and general purpose registers; a small set of fixed length high speed memory locations to which program instructions can refer directly. The number of registers varies from one architecture to another, from as few as 8 in intel architecture, to 32 in some RISC designs. 



#### Address formation
> Refers to the specific rules and mechanisms a given hardware architecture uses to calculate the final memory address referenced by an instruction.

The precise address formation rules used by a computer architecture are critically important because linkers must ensure any addresses or offsets they modify when patching object code match the computers addressing scheme. 

Fundamental constraint is the limited size of address fields within instructions. Historically, early computers favoured direct addressing, but exponential memory growth necessitated abandonment of this scheme. 

Typically use one of **3** main strategies for address formation:
1. Direct Addressing - address contained entirely within the instruction
2. Register Indirect Addressing - address is found in one of the registers. 
3. Based/Index Addressing - address is calculated by adding a constant offset (contained in the instruction) to the contents of one or more registers. 

Different architectures employ diverse methods to form and utilise addresses, imposing unique challenges. Analyse 3 different architectures.
- IBM 360/370/390
- SPARC V8 and V9 - popular RISC architecture and fairly simple addressing, V8 uses 32 bit registers and addresses, V9 adds 65 bit registers and addresses.
- Intel x86

### Instruction formats
The instruction format of a target architecture is a key detail affecting the life of the linker or loader author!

Each instruction consists of an opcode (determining the operation), and operands. An operand may be encoded in the instruction itself (an immediate operand), or located in memory. The address of each operand in memory has to be calculated somehow. Address can be contained in instruction(direct addressing), contained in register(register indirect), or calculated by adding a constant in the instruction to the contents of a register(based/indexed addressing). 

Architectures differ in their instruction size, which affects parsing and modification efforts. Fixed vs Variable length instruction size.
- Fixed length: $SPARC$ instructions are always four bytes long
- Variable length: $IBM 370$ instructions can be 2, 4, or 6 bytes long. x86 instructions can range from one byte up to 14 bytes. 
	- From POV of linker writer, the address and offset fields that linker has to adjust occur on byte boundaries, thus linker does not need to be concerned with the instruction encoding.
	NOTE: ARM7TDMI, executes 32 bit word aligned instructions when not in thumb mode, otherwise execute 16 bit halfword aligned instructions. 



### Procedure calls and addressability
On architectures without support for direct addressing, programs have a bootstrapping problem. Some instruction uses base values in registers to calculate data addresses, but the standard way to get a base value into a register is to load it from a memory location, which is in turn addressed from another base value in a register. The problem becomes how do we get the first base value into a register at the beginning of the program and subsequently to ensure that each routine has the base values it needs to address the data it uses. 

#### Procedure calls
Every ABI defines a standard procedure call sequence, which relies a combination of hardware defined instructions, and conventions regarding register and memory use. 

 A hardware call instruction saves the return address and jumps to the procedure. On architectures with a hardware stack, such as x86, the return address is pushed onto said stack, whereas on other architectures its saved in a register, with software having the responsibility to save the register in memory if necessary (e.g., ISR invoked, PCB goes onto software stack.)  
	 NOTE: ARM7TDMI: return address is saved in a register designated as the **Link Register**, register 14. When a branch and link instruction is executed (**BL**), r14 receives a copy of the Program Counter. 

 Architectures with a stack generally have a hardware return instruction that pops the return address from the stack and jumps to that address, while other architectures use a 'branch to address in register' instruction to return.

Within a procedure, data addressing falls into four categories:
1. The caller can pass *arguments* to the procedure
2. *Local variables* are allocated within the procedure and freed before the procedure returns
3. *Local static data* is stored in a fixed location in memory and is private to the procedure.
4. *Global static data* is stored in a fixed location in memory and can be referenced from many different procedures.

The piece of stack memory allocated for a single procedure call is known as a **stack frame**. Arguments and local variables are usually allocated on the stack. One of the registers serves as a stack pointer, which can be used as a base register, pointing to the current top of the stack. Another register serves as a frame pointer, pointing to the base of the current function's stack frame. This remains constant, and allows a procedure to address arguments at a fixed positive offset from the $FP$, and address local variables at a fixed negative offset from the $FP$(assuming stack grows from higher to lower addresses.) Doing so with the SP would be impractical as the SP is dynamic. 

![[Pasted image 20251019020805.png]]
	NOTE: ARM7TDMI, register r13 is used as the stack pointer. 
	In thumb mode the SP is explicitly used, such as SP relative load and store, and ADD instructions used for loading addresses relative to the SP

The OS typically sets the initial SP register before a program starts, so the program need only update the register when it pushes and pops data.  

Addressing mechanism for local and global static data differs. Since this data is stored statically and not on the stack, its address is fixed at link time. However due to limited address field size, direct addressing often impossible. To access static data, compilers and linkers often cooperate to maintain an address pool ( a pointer table). This address pool contains pointers to all static objects referenced by the routines in a module. 

This mechanism involves designating a special register, the table pointer, which points to the base of the address pool. The compiled code uses based addressing - it combines a constant offset found in the instruction with the contents of the table pointer register, to load the full 32/64 bit address of the target static data into temporary register e.g., $Rx$. Can then use $Rx$ as register indirect addressing to perform final load or store operation etc. 

The trick then, is to get the address of the table into the first register. On SPARC, the routine can load the table address into the register using a sequence of instructions with immediate operands, and on the 370 the routine can use a variant of a procedure call instruction to load the PC(keeps address of current instruction) into a base register. 

![[Pasted image 20251019022831.png]]

Essentially, becomes caller's job to load table pointer. Since caller already has table pointer loaded, it can obtain the address of the called routine's table from its own table and pass it to the callee, often by loading the routine pointer into $R_t$ before maxing the call. 


Several optimisations are available here, in many cases, all the routines in a module share a single pointer table. The SPARC convention is that an entire library shares a single table - created by the linker - so that the table pointer register can remain unchanged in intramodule calls. Calls within the same module can be usually be made using a version of the call instruction with the offset to the called routine encoded in the instruction, which avoids the need to load the address of the routine into a register. 

Returning to the address bootstrapping question, how does this chain of table pointers get started, where does the initial routine get its pointer? Always involves special case code, the main routine's table may be stored at a fixed address, or the initial pointer value may be tagged in the executable so the OS can load it before the program starts. Whatever the technique, it invariably needs some help from the linker.

### Data and instruction references
Ways different computer architectures handle memory addressing for both data storage and program control flow, which critically dictates the relocation work performed by linkers.

#### IBM 370
Classic register-memory architecture that uses a straightforward, consistent addressing scheme that simplifies linkage.

There are 16 general registers, each 32 bits, numbered 0 from 15, all but one of which can be used as index registers; if registers 0 is specified in an address calculation, the value 0 is used rather than the register contents(used for arithmetic rather than addressing).

Address formation: Use indirect register addressing, add 12 bit unsigned offset contained in the instruction to a base register and potentially an index register. 

Instruction formats: the major instruction formats include RX, an instruction containing a register operand and a single memory operand(calculated via address formation mechanism). More often that not, the index register is zero so the address is just base plus offset. In the RS, SI, and SS formats, the 12 bit offset is added to a base register. An RS instruction has one memory operand, with one or two other operands being in registers. An SI instruction has one memory operand, the other operand is an immediate 8 bit operand in the instruction. An SS instruction has two memory operands. The RR format has two register operands and no memory operands at all, although some RR instructions interpret one or both of the registers as pointers to memory. Throughout all of these, the 12 bit address offset is always stored as the low 12 bits of a 16 bit aligned halfword. 
	This consistency allows linkers to specify fixups to address offsets in object files without needing to refer to the specific instruction format as the offset storage location is always the same. 

Instructions can directly address the lowest 4096 locations in memory by specifying base register zero - essential in systems programming, but appliocation programs just use base register addressing. 
	Since the offset is limited to 12 bits, a single base register "covers" only a **4K chunk of code or data**. Larger routines must use multiple base registers to maintain **addressability** across their entire size.

![[Pasted image 20251019043102.png]]

The original 360 had 24-bit addressing, with an address in memory or a register being stored in the low 24 bits of a 32 bit word and the high 8 bits discarded.  

The 370 extended addressing to 31 bits. Unfortunately many programs including OS/360, the most popular OS stored flags or other data in the high byte of 32 bit address words in memory, wasn't possible to extend the addressing to 32 bits in the obvious way and still support existing object code. Instead, system has 24-bit and 31-bit modes, anytime CPU interprets 24 bit addresses or 31-bit addresses, a convention enforced by a combination of hardware and software states than an address word with the high bit set contains a 31 bit address and vice versa. 

As a result, linkers have to be able to handle both these addressing schemes, as programs can and do switch modes depending on how long ago a particular routine was written. 

Instruction addressing on the 370 is also relatively straightforward. All jumps are RR or RX.
1. RR jumps - the second register operand contains the jump target, with register 0 representing no jump
2. RX jumps - the memory operand is the jump target. 
The **procedure call is branch and link**, which stores the return address in a specified register, and then jumps to the address in the second register in RR form, or to the second operand address in RX form. 

For jumps within a routine, the routine must establish addressability, that is a base register points to the beginning of the routine that RX instructions can use. By convention, register 15 contains the the address of the entry point to a routine and cab be used as a base register. Since RX instructions have a 12 bit offset field, a single base register (15) covers a 4KB piece of code. If a routine is bigger than that, it has to perform more complex address formation, using multiple base registers.

Alternatively, an RR branch and link with a second register of 0 stores the address of the subsequent instruction in the first operand register but doesn't jump; this can be used to set up a base register if the prior register contents are unknown. ???


Anyway - later IBM 390 models added relative forms of most jumps. That is, the instruction contained a signed 16 bit offset (shifted left one bit for alignment, instructions must be aligned on 2 byte boundary) and that offset is added added to the address of the instruction to get the address of the jump target. This permits jumps within +/- 64K bytes, sufficient for most intraroutine jumps. 
	The reason for the left shift is because instructions are aligned on a 2 byte boundary, the LSB of any valid address is 0, so the is no point storing odds in the offset. Thus the offset actually represents the number of halfwords to jump, as to not waste bits. To calculate the number of bytes when it comes time to execute that addition a left shift must be performed to reach a valid address. 

#### SPARC
SPARC has 4 major instruction formats, and 31 minor instruction formats, 4 jump formats, and 2 data addressing modes. 
SPARC versions through V8 are 32-bit architectures. SPARC V9 expands the architecture to 64 bits.
![[Pasted image 20251019194407.png]]
Long.
##### SPARC V8
In SPARC V8 there are 31 general purpose registers, numbered 1 to 31. Register 0 is a pseudo-register always containing the value 0. 

An unusual register window scheme attempts to minimise the amount of register saving and restoring at procedure calls and returns. 


###### Addressing
Address formation: SPARC uses one of two addressing modes.
1. Compute address by adding the values in two registers(one of these can be register 0 if one register already contains desired address).
2. Compute address by adding a 13 bit signed offset in the instruction to a base register.
SPARC assemblers and linkers support a psuedo-direct addressing scheme using a two instruction sequence. The two instructions are SETHI, which loads its 22-bit immediate value into the high 22 bits of a register and zeros the lower 10 bits. This is followed by OR immediate, which ORs its 13 bit immediate value into the low part of the register. The linker must handle specific relocation modes to arrange to put the high and low parts of the desired 32 bit address into the two instructions.

###### Procedure call and jumps
The procedure call instruction and most conditional jump instructions (referred to as branches in SPARC literature) use relative addressing with various size branch offsets ranging from 16 to 30 bits. Whatever the offset size, the jump shifts the offset 2 bits left (must be aligned at 4 byte word addresses). Sign extension is performed - identify the sign bit (0 for positive, 1 for negative), this sign bit is then duplicated and prepended, preserving the magnitude of the offset, whilst expanding the product of the shift to the full width of the register (32 bits for V8, 64 bits for V9). This result is added to the address of the jump or call instruction to get the target address.

The call instruction uses a 30 bit offset, can reach any address in a 32 bit V8 address space. Calls store the return address in register 15.
Various jumps use a 16, 19, or 22 bit offset, which is large enough to jump anywhere in a plausibly sized routine. 

cba covering the rest of this, not really relevant. 
#### x86
By far most complex of the architectures covered. Features asymmetrical instruction set and segmented addresses. 
![[Pasted image 20251019201436.png]]
^ Generalised.

There are **six** 32 bit general purpose registers named EAX, EBX, ECX, EDX, ESI and EDI.

There are two 32 bit registers used primarily for addressing named EBP and ESP. 

There are six specialised 16-bit segment registers, CS, DS, ES, FS, GS and SS. The low half of each of the 32 bit registers can be used as 16 bit registers, called AX, BX, CX, DX, SI, DI, BP, and SP, respectively.

The low and high bytes of each of the AX through DX registers are 8 bit registers called AL, AH, BL, BH, CL, CH, DL, and DH. On the 8086, 186, and 286 chips, many instructions required their operands to be in specific registers, but on the 386 and later chips, most, but not all of the functions that required specific registers have been generalised to use any register.

The ESP is the hardware stack pointer and always contains the address of the top of the stack. The EBP is usually used as a frame register that points to the base of the current stack frame (but this is not required, though it is encouraged).

At any moment an x86 is running in one of three modes.
1. Real mode - simpler operating mode for x86 which emulates the 8086.
2. 16 bit protected mode - introduced protected mode features allowing for memory protection and such, but limited to 16 bit addressing.
3. 32 bit protected mode - added on 386.
Protected mode involves the x86s segmentation. 

##### Address formation(data)
Most instructions addressing data use a generalised instruction format that calculates the memory address by adding together any or all of:
- A signed 1, 2, or 4 byte displacement value in the instruction. 
- A base register, which can be any of the 32 bit registers
- An optional index register, which can be any of the 32 bit registers except ESP.
	- Index register often holds relative position or offset from base address. Very useful for indexing arrays of values.
	- Value in index registers can be logically shifted 0, 1, 2, or 3 bits, multiplying the index value, making it easier to calculate the address of an element in an array where each element occupies 1, 2, 4, or 8 bytes. 
Althought it is possible for a single instruction to include all of the displacement, base, and index, most just use a 32 bit displacement, which provides direct addressing. Alternatively, a displacement of 1 or 2 bytes is used in conjunction with a base register (often the EBP or ESP) to achieve stack addressing or pointer referencing.

From a linker's point of view, direct addressing simplifies relocation as the instruction or data address to be modified can be embedded anywhere in the program, often on any byte boundary. Couple this with the segmentation of x86 and relocation becomes much more trivial.


##### Procedure calls and jumps
x86 predominantly uses relative addressing for control flow within the current address space, whilst also retaining complex segmented addressing capabilities for intersegment operations.

1. Relative addressing (intra-segment control flow)
	All conditional and uncoditional jumps and procedure calls rely heavily on relative addressing. The instruction contains a signed offset of 1, 2 or 4 bytes. This offset is added to the address of the instruction immediately following the jump or call to determine the target address. This allows jumps and calls to reach anywhere in the current 32 bit address space. No relocation required for jumps and calls that remain in same segment, as the relative position(offset) from base address of segment never changes.
2. Address calculation for jumps and calls
	In addition to simple relative offsets, unconditional jumps and calls can determine the target address using the more generalised addressing mechanism. Compute target address by adding displacement + base register + optional index register.
3. Call instructions
	Performs two actions: saving the return address and transferring control
	Call instructions push the return address onto the stack, maintained by the extended stack pointer(ESP) register.
4. Intersegment control flow 
	Unconditional jumps and calls can include a **full 6 byte segment/offset address** directly within the instruction, or calculate the address where this 6 byte target address is stored. Push both the return address and the caller's segment number, to permit valid return, and facilitate intersegment calls. 

#### Paging and virtual memory
**Paging** uses fixed partitioningand relocation to devise a non-contiguous memory management scheme. Paging hardware divides a program's address space into fixed size pages, and divides the physical memory of the computer into page frames of the same size ranging from 512 bytes to 1GB. 

Process perceives its segments to be contiguous.

We use a page table to store the mapping between virtual pages and physical frames. A logical address is relative to the start of the program, or the start of a segment. The leftmost $n$ bits represent the page number, the rightmost $n$ bits represent the offset within the page. In a 4KB block, 4 bits used for the page number (16 pages) and 12 bits used to represent the offset within the page, respecting alignment. The frame number is a function of the page number, when calculating memory address from page number, just replace it with frame number and keep offset as is. 

![[Pasted image 20251019232419.png]]
A page table entry consists of a present/abset bit of the frame is in memory, a modified bit which is set if the page/frame has been modified since in memory (only modified pages have to be committed to disk when evicted), a present/absent bit, and RWX protection bits. 

The OS can load a copy of the contents page from disk into a free page frame, then let the application continue. By moving pages back and forth between main memory and disk as needed, the OS can provide virtual memory, which appears to the application to be far larger than the real memory in use. If a page fault and consequent page in or page out occurs, takes several milliseconds due to disk transfer.

The more page faults a program generates, the slower it runs, the worst case being thrashing (performance problem that occurs when kernel spends majority of its time swapping pages between RAM and disk, can be prevented by good page replacement policy, paging daemon, reducing degree of multi-programming). **If a linker can pack related routines into a single page or into a small set of pages, paging performance improves**. 

If pages are marked as read only, performance only improves, no need to commit back to disk when swapping out if not in use.

An x86 with 32-bit addressing and 4KB pages would need a page table with 2$^{20}$ entries to map an entire address space. Because each page table entry is usually 4 bytes, this would make the page tables an impractical 4MB long.


This provoked the advent of multi-level page tables. Organises virtual address space into hierarchical page tables. The entries of a level 1 page table are pointers to a level 2 page table and so and and so forth. The entries of the last level page table(s) store actual frame information. Only the outermost page table resides in main memory, and other tables are not brough to main memory unless required, and are stored in virtual memory. Page number is now divided into an index into the root page table, followed by subsequent indexes according to depth of page table, to reach frame mapping. Though mult-level page tables contain up to 5 levels in practice, often use TLB to optimise virtual address translation, part of MMU that exploits temporal locality (stores recent translations of virtual addresses to frame numbers and can be searched in parallel.)
![[Pasted image 20250115222222.png]]

In x86, each upper-level page table contains 1024 entries. Each lower level page table also contains 1024 entries, to map the 1024 4KB pages in the 4MB of address space corresponding to that page table. Yeah whatever!


##### The program address space
Set of addresses available to an application program, determined by combination of hardware and OS. Linker/Loader must create a runnable program that correctly matches this address space. 

###### Historical examples
PDP-11 UNIX. The address space was 64KB. The read only code of the program was loaded at location zero, with the read/write data following the code. The PDP had 8KB pages, so the data started on the 8KB boundary after the code. Stack grew downward, starting at 64KB, as stack and data grew, enlarged; if met, program ran out of space.

DOS on x86 systems used no hardware protections, so the system and running applications share the same address space. Places programs in largest piece of free memory.

Windows has unusual loading scheme. Each program is linked to load at a standard starting address, but the binary executable contains relocation information, used if the starting address is not available; relocates at load time. 


##### Memory Mapped files
Virtual memory systems commonly unify the paging mechanism with the file system through **mapped files**. 

A memory mapped file is a segment of virtual memory that has been assigned a direct byte-for-byte correlation with some portion of a file or file-like resource (anything that can be referenced via a file descriptor).

When a program maps a file to part of program address space, the OS marks all the pages in that segment as not present, and uses the the file as the paging disk(backing store) for that part of the address space. The program can read the file merely by referencing that part of the address space, at which point the paging system loads the necessary pages (chunks of the file) from disk. 

There are three different approaches to handling writes to mapped files.

1. Map file as read only(**RO**) - An attempts to write fail.
2. Map file as read write(**RW**) - changes to the memory copy of the file are paged back to the disk by the time the file is unmapped, according to modified bit. 
3. Map the file as copy on write (**COW**) - Maps each page as RO until program writes to it, at which point create a private copy of the page for that process.

Facilitates IPC, efficiency and reduces need for I/O system calls if managed correctly.


##### Shared libraries and Programs
In nearly every system that handles multiple programs simultaneously, each program has a separate set of page tables, giving each program logically separate address space. Makes system considerably more robust, but can also cause performance programs.

If a single program or library is used in one more address space, the system can save a great deal of memory if all the address spaces share a single frame copy of the program or library. This is relatively straightforward for the OS to implement - just map the executable file into each program's address space. 

Unrelocated code and read-only data are mapped RO; writable data is mapped COW. The OS can use the same physical page frames for RO and unwritten COW data in all the processes that map the file.  If the code has to be relocated at runtime, the relocation process changes the code pages and they have to be treated as COW, not RO. 

Considerable linker support is required by grouping all executable code into one segment that can be mapped, and all data into another part that can be mapped. Each segment must also be aligned on page boundaries. When several different programs use a shared library, the linker needs to mark each program so that when each starts, the library is mapped into the program's address space.

Binding/Mapping of libraries depends on the nature of said library e.g., DLL vs Static Shared.

##### Position Independent Code
PIC is a body of machine code that executes properly regardless of its memory address. This is commonly used for shared libraries, so that the same library code can be loaded at a location in each program's address space where it does not overlap with other memory in use by, for example, other shared libraries. Can map code as read only. 

PIC is fairly straightforward to create, must use relative addressing(all modern architectures support). E.g., ones discussed earlier all use relative jumps for intraroutine jumps, and use frame pointer in stack frame as relative point to access local variables and such. The only challenges are calls to routines not in the shared library, and references to global data. Discussed later.


### Intel x86 segmentation system
Only segmented architecture still in common use. 
Segmentation was a compromise introduced by original 8086 designers to provide a larger address space than the 16 bit architecture of its predecessors while still supporting compact code. 

Modern x86 processors have relegated segmentation to a legacy role. The vast majority of modern processor architectures do not support segmentation, and favour paging and hardware support for paging over segmentation. 

1. Multiple 16 bit Address spaces: The 8086 created multiple 16 bit address spaces, each known as a segment
2. Segment registers: A running x86 program has four active segments defined by specialised 16 bit segment registers:
	$CS$ (Code Segment) - Defines the segment from which instructions are fetched
	$DS$ (Data Segment) - Defines the segment for most data load and store operations
	$SS$ (Stack Segment) - Defines the segment used for the stack and for data references using EBP or ESP
	$ES$ (Extra Segment) - Used by a few string manipulation instructions.
The 386 and later chips provided two more segment registers, accessed primarily via **segment overrides**.
	Any data reference can be directed into a specific, non-default segment using a segment override e.g., MOV EAX, CS:TEMP fetches data from the code segment instead of the default data segment.

Often set DS and SS to the same value, allowing pointers to stack variables and global variables to be used interchangeably. Small programs might set all four segment registers the same, creating a single address space known as the **tiny model**.

Most 386 operating systems now run applications in the **flat model** (or tiny model). They create a single code segment and a single data segment, both 4GB **long**, and map them to the full 32-bit paged address space, creating a unified address space once more. 

In the flat model, segmentation becomes largely invisible to applications. This makes the x86 relatively **easy to handle** from a linker's perspective, as the linker primarily deals with 32-bit direct and PC-relative addresses.

### Embedded architectures
Linking for embedded architectures pose a variety of problems that rarely occur in other environments. Embedded chips have limited amounts of memory and limited performance, but embedded programs, may be built into millions of devices, great incentive to optimise!
#### Address space quirks
Small address spaces with quirky layouts. A 64KB address space can contain combinations of fast on-chip ROM and RAM, slow off-chip ROM and RAM, on-chip peripherals, and off chip-peripherals. There may be several noncontiguous areas of ROM or RAM. 

Essentially, they do whatever they like, and for very good reason. A linker for an embedded system needs a way to specify the layout of the linked program in great detail by assigning particular kinds of code or data or even individual routines and variables to specific addresses to maintain the sanctity of the given address space. 
#### Non uniform memory
References to on-chip memory, faster. In system with both on and off chip, most time-critical routines go in fast memory. Sometimes possible to squeeze all of the programs time critical code into fast memory at link time, other times makes more sense to copy code or data from slow memory to fast memory as possible. 
Linker has to perform some black magic to accomplish this trick!

#### Memory alignment
DSPs and embedded chips often have stringent memory alignment requirements for certain kinds of data structures. E.g., Motorola 56000 series has an addressing mode to handle circular buffers very efficiently, so long as the base address of the buffer is aligned on a power of two boundaries






