# Relocation
Assemblers emit object files lacking information such as where each sections will end up in executable image, where other object files' symbols will be placed, and whether a symbol will be forcibly moved into the $GOT$. 

No ahead of time knowledge of virtual address assignments that correspond to the planned memory image means that relocations are emitted by the assembler to refer to positions where certain instruction fields and data values cannot yet be computed. The assembler emits placeholder bytes, and a corresponding relocation entry according to the object format in use to permit adjustment when a semblance of the memory image has been computed. 

To give a concrete definition of relocation:
> Relocations are transformation directives encoded in a relocatable object file (position dependent) that specify that the machine value stored at location $(s, p)$ is not fixed and must be computed by applying a relocation-type-specific transformation $F_t$ to the resolved address of the symbol $\sigma$. The set of all relocations defines the incomplete semantic state of a relocatable object and imposes constraints on valid program instantiation. 

It should be noted that even in the case of absolute references, it is not common for architectures to hold a $word$ displacement. Thus, $F_t$ is rarely the identity function, even in the case of absolute relocation (there is usually some sort of truncation or manipulation, but this is dangerous and very finnicky for the obvious reasons). Though in the case of literal pools, $F_t$ is perfectly fine being the identity function. 
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

Relocation entries vary according to object format. They generally describe the section/segment relative offset at which to apply the relocation $P$, which symbol is referred to $S_{\sigma}$, and what relocation type to apply $F_t(S_{\sigma}, P, A)$. 

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
Assembler generates an ELF relocatable object with placeholder values at locations $P_1...P_n$. Relocation entries are emitted as sections with `sh_name` = `.rel/a{name}` and `sh_type` = `SHT_REL/A`. 

Relocation entries in ELF are detailed below:
```C
typedef struct {
	Elf32_Addr    r_offset;
	Elf32_Word    r_info;
} Elf32_Rel;

typedef struct {
	Elf32_Addr    r_offset;
	Elf32_Word    r_info;
	Elf32_Sword   r_addend;
} Elf32_Rela;
```
`r_offset` gives $P$ where $P$ in a relocatable file is the byte offset from the beginning of the section to the unit affected by the relocation. In `ET_exec` or `ET_dyn`, this is a virtual address of the storage unit affected by the relocation. 

`r_info` gives both the`.symtab` index with respect to which the relocation must be made, and relocation type. If the index is `STN_UNDEF` the relocation uses 0 as the symbol value. 

`r_addend` specifies a constant addend to compute the value to be stored into the relocatable field. 

### Absolute relocation (data relocations)
These occur in data sections, or in instruction formats that utilise absolute addresses. For example, global pointer stored in `.data`, literal pools containing an address. Such relocations are thus for absolute references. 

For these relocations, it is often enough that:

```C
FinalValue = SymbolAddress + Addend
```
### Relative relocation(instruction relocation)
These occur when an instruction is to encode a relative offset, not a full absolute address. For example, branch instructions, and PC relative loads in some architectures. These relocations require something akin to:
```C
Offset = SymbolAddress - PC
```
Yield distance. Linker inserts the offset and ensures it fits. 

#### ARM7TDMI
ARM7TDMI executes branch instructions PC-relative and accesses data using absolute or relative addresses. ARM ELF defines a rich set of relocation types tailored for a number of different use cases. TO COMPLETE: GOT, TLS etc

Supports two execution states: $ARM$ and $Thumb$ executing word and halfword instructions, respectively. In $ARM$ state, the CPU fetches instruction $i$ at $PC$, it decodes instruction $i$ while fetching instruction $i+1$ at $PC+4$, by the time instruction $i$ executes, the $PC$ register has already moved ahead $2$ instructions. Thus during execution in $ARM$ state, $PC =\text{(address of current instruction) + 8}$.


There are many instruction set formats in $ARM$:
![[Pasted image 20251117145749.png]]
Some of the most used ones include:
- **Branch** - The offset is signed 24 bits byte offset, shifted left twice to ensure alignment, and added to $PC+8$.
- **Single Data Transfer** - $ARM$ can load and store from a register plus an immediate offset, a register plus a register, or PC relative, using the PC as a base register; this is used for literal pools.
- **Data processing** - use registers only

##### Addressing modes 
There are only three that matter in ARM state for relocation purposes. 
1. **PC relative branch offset in an instruction**
2. **PC relative literal load**
3. **Direct use of symbol in data**
###### PC relative branch offset in instructions
```C
bl my_function
```

The assembler knows the location of the BL instruction but often does not know the location of the function. It cannot compute the distance from the BL instruction to `my_function` without knowing the full memory image, thus a BL instruction with an offset of 0 is emitted, and a relocation entry in `.rela{text}` looks akin to this:
```C
offset: &bl
type: R_ARM_CALL
symbol: my_function
addend: 0(implicit)
```
###### PC relative literal load
```C
ldr r0, = symbol
```

Assembler expands this into:
```C
ldr r0 [PC, #offset_to_literal_pool_entry]
...
.pool
.word symbol
```

For a better example:
```C
ldr r0,hello
nop
nop
nop
hello: .word 0x12345678

Disassembly of section .text:

00000000 <hello-0x10>:
   0:   e59f0008    ldr r0, [pc, #8]    ; 10 <hello>
   4:   e1a00000    nop                 ; (mov r0, r0)
   8:   e1a00000    nop                 ; (mov r0, r0)
   c:   e1a00000    nop                 ; (mov r0, r0)

00000010 <hello>:
  10:   12345678    .word   0x12345678
```
A relocation is emitted for the literal pool entry, which holds the absolute address of the data referenced by the PC relative literal load. 

This relocation looks akin to:
```C
offset: location of literal pool word
type: R_ARM_ABS32
symbol: symbol
addend: 0
```
###### Direct use of symbol in data
```C
.word external_symbol
```
The assembler cannot possibly know the final address of `external_symbol`. 
A placeholder is thus emitted and a relocation entry similar to the one for PC relative literal loads is produced by the assembler. 
##### Literal pools
These are blocks of data placed in the code section, near to the instructions that reference them. $ARM$ cannot load a full-32 bit immediate in one instruction, but can hold in a register, so the pool holds either the constant or the address of some data, and the PC-relative $LDR$ retrieves it. Pools must be reachable from within $+/- 4KB$ from the $LDR$ instruction.  Relocations are emitted for references in literal pools given that the address of the data referred to cannot be determined when producing `ET_REL` objects.
##### Veneers
A veneer extends the range of a branch by becoming the intermediate target of the branch instruction. The branch instruction $BL$ is $PC-relative$ and has a limited branch range. The range of a $BL$ instruction is **32MB**(24 bit signed, shifted left 2, yields reach ≈ ±32 MB).

If the target is outside the above range, the linker inserts a veneer to bridge the gap. The branch instruction is re-targeted to the veneer, and the veneer then performs the jump to reach the actual symbol.

Veneers are inserted by the linker where $value = F_t(S, P, A)$ (where this is often $displacement = S - (A+8)$) does not fit in the 24-bit displacement

There are different types of veneers generated, below are the most common 2:

###### Inline Veneer
Inserted immediately near the branch that exists to facilitate interworking between CPU states. These are not relevant for our case.

###### Long Branch Veneer
Can reach anywhere in the 32 bit address space. Below is a long veneer for an ARM function call:
```C
	bl __veneer_far_func
...
__veneer_far_func:
	ldr pc, [pc, #0]
    .word far_func
```
It should be noted that that $imm$ is 0 as there is an implicit 4 byte padding inserted by the assembler such that `.word far_func` is 8 bytes away from the $LDR$ instruction. With no padding $imm$ = -4. 

##### ARM7TDMI relocation in ELF
Compute the following symbols:
- $S$ - address of the symbol
- $A$ - addend for the relocation
- $P$ - the address of the *place* being relocated (derived from `r_offset`)
- $P_{a}$ - the adjusted address of the place being relocated, defined as $P \ \& \ 0xFFFFFFFC$ 
- $T$ - is 1 if the target symbol $S$ has the type `STT_FUNC` and the symbol addresses a Thumb instruction; 0 otherwise. Not relevant for our case.
- $B(S)$ - addressing origin of the output origin of the output segment defining the symbol $S$. The origin is not required to be the base address of the segment however. Must be word-aligned. 
- $GOT\_ORG$ - the addressing origin of the Global Offset Table ($GOT$)
- $GOT(S)$ - the address of the $GOT$ entry for the symbol $S$
And provide case matching on types to perform the relocation operation and patch its result to $P$. 

###### Addend formation for REL entries
ELF files may use `REL` or `RELA` entries. In the case of `REL` entries, the addend must be formed by extracting the immediate field from the instruction (often 0 for external symbols, often not 0 for collection type element references). See [[Relocation generation in assemblers]] to learn more. 

The object producer must have encoded compensation for the $PC$ bias. The addend extraction respects this. In $ARM$ state $PC = instr\_address + 8$. 

If the place is subject to a *data-type relocation*, the initial value in the place is sign-extended to 32 bits and this is used as the the initial addend.

If the place contains an instruction, the immediate field for the instruction is extracted and used as the initial addend. If the instruction is a $SUB$ or $LDR/STR$ type instruction with the upper bit clear, then the initial addend is formed by negating the unsigned immediate value encoded in the instruction. Otherwise, the initial addend is merely the immediate, possibly sign-extended if the immediate field itself is signed.

For example:
```C
0x1000: CALL sym
```
An `REL` relocation `R_ARM_CALL` is generated for `0x1000`. One extracts the signed 24-bit immediate from bits `[23:0]` of the $BL$ encoding. This is shifted twice to yield a byte displacement. After sign extension, this yields our addend. Then, `V = (S + A) - P`.

Below is a table detailing the `REL` addend formation according to instruction type. Insn modification is detailed shortly.
![[Pasted image 20251117183239.png]]

Note: If the initial addend cannot be encoded in the space available, the object must produce a `RELA` format relocation instead.

###### High level flow for every relocation
1. Read `.rel/a{name}(s)` and parse
	If `sh_type` = `SHT_REL` then for each `Elf32_rel` extract `P` as `r_offset`, and compute `A` as per the instruction class rules.
	If `sh_type = SHT_RELA` then for each `ELF32_rela` extract `P` as `r_offset` and compute `A` as `r_addend`.
2. Compute the relocation expression implied by the relocation type:
	`r_info` gives the symbol table index and relocation type. The type implies the expression, typically `V = (S + A) - P` for pc-relative references, and `V = S + A` for absolute references.
3. For instruction relocations:
	Check overflow according to relocation's allowed bit fields. If overflow, choose a remedy: a) generate a veneer(if permitted for this type) or b) report error. The relocation types that permit long-branch veneers are as follows:
	- `R_ARM_PC24(deprecated)`
	- `R_ARM_CALL`
	- `R_ARM_JUMP24`
	Where the target symbol has type `STT_FUNC` or the target symbol and relocated place are in separate sections input to the linker.
4. Patch the immediates with the processed `V`
	(after applying any group masks for multi-instruction immediate sequences). 

###### Overflow Checking
Only some relocation types require formal overflow checking. Overflow checking is often performed on the value $V$ before it is masked and patched at $P$. If the relocation type has Overflow = 'Yes' then overflow checking is performed according to the rule for that relocation type. If there is no overflow, apply the result mask and modify the instruction. 

Branch relocations must have their overflow checked to ensure that an immediate is produced that does not exceed the 24 bit signed width. If $V$ is not representable in signed 26 bits (i.e., `X >> 2 outside 2^24`): Overflow error and/or veneer generation. 

In cases of simple masked relocations such as `R_ARM_ABS12` one performs:
```C
if ABS(X) > 0xFFF
```
True results in overflow error.

Without performing overflow checking garbage immediates will be produced and the binary will not function properly. 

###### Result Masks
> Declare which bits of the computed relocation value $V$ are written back into the instruction. 

In the spec's tables, these show how the 32-bit relocation result $V$ is masked/partitioned before being written in the encoded instruction field. 

Example: `R_ARM_ABS12`, the mask is `ABS(V) & OxFFF` - the resulting value becomes the 12 bit immediate of an $LDR/STR$ instruction.

Example: `R_ARM_CALL`, the mask `V & 0x03FFFFFE` corresponds to the 24-bit signed branch offset (stored shifted right by 2 and with sign bit). 

If a result mask contains `ABS(X)` it means the immediate field cannot encode a sign; the sign is encoded elsewhere e.g., bit 23 of $LDR/STR$ 

These tables in the specification describe how to weave masked bits back into the `insn` fields. For example, for $LDR$ instructions, `insn[11:0] = Result_Mask(V)`

###### Key Relocation types
The class is implied by the relocation code in the specification. Addend extraction for REL entries is subject to the class of the relocation type.
Dynamic relocations will be added here later.
- `R_ARM_CALL `
	Used for unconditional function call instructions ($BLX$ and $BL$ with the condition field to set to `0xE`). A linker would prefer changing the instruction to $BLX$ to effect the transition to Thumb state here.
- `R_ARM_JUMP24`
	Used for conditional function call instructions. A linker would prefer an inline veneer to effect the transition to Thumb state here.
- `R_ARM_ABS32`
	Used for absolute data references - global data pointers, vtables, jump tables, and literal pools. 
- `R_ARM_ABS12`
	Used to relocate a 12 bit unsigned immediate in $LDR, STR, LDRB, STRB$. The sign is encoded separately via $U-bit$ (bit 23). This relocation stores imm12 = `ABS(V) & 0xFFF` provided that `ABS(X) > 4096` ($ARM$). Sets $U = 1$ if `V >= 0` and $U = 0$ if `V < 0`.
- `R_ARM_PREL31`
	Used to relocate a signed 31-bit PC relative value. 
	V = (S + A) - P
	Check that V fits in signed 31 bits 2's complement
	Store V & 0x7FFFFFFF with sign bit preserved.
	Data-only used for things like GOT-like tables in static linking.
- $MOVW$ and $MOVT$ relocations
  ```C
 	MOVW Rd, #imm16 ; write low 16 bits
 	MOVT Rd, #imm16 ; write high 16 bits
 	```
	The pair permits constructing a full 32-bit constant. 
  ```C
 	MOVW Rd, (V & 0xFFFF)
 	MOVT Rd, (X >> 16)
 	```
	There are a plethora of relocation types, we denote these via `R_ARM_MOVW_**`, and `R_ARM_MOVT_**`. For `ABS` relocations under these, we compute `V = S + A`. For PREL relocations under these, we compute `V = S + A - P`. Then for $MOVW$ `imm16 = V & 0xFFFF`. For $MOVT$ `imm16 = (V >> 16) & 0xFFFF`. Certain types may not be overflow checked. Implemented according to the type in the specification. 
- Group Relocations
	These are used when encountering a sequence of instructions to form a large constant or address. 
	For example:
	```C
 	ADD r0, pc, #imm1
 	ADD r0, r0, #imm2
 	LDR r1, [r0, #imm3]
 	```
 	`ABS(V)` is decomposed into groups. `G0` is the low chunk, `G1` is the next chunk, `G2` is the chunk after that. Each instruction gets one chunk. One starts with Y = ABS(V). Find the highest non-zero bits that fit into G0 and extract them into G0. Subtract G0 from Y, and repeat. If leftover bits remain, check overflow. Overflow checking is performed on the highest number Group relocations appear as ARM immediates are limited. 
 	


## Relinkable and relocatable output formats



## Exercises
Why does a SPARC linker check for address overflow when relocating branch addresses doing the high and low parts of the addresses in a SETHI sequence?
	When performing a relocation at an offset for a branch instruction, I would assume the SPARC linker performs address overflow ensuring that the result of the relocation computation before it is result masked, will fit into the immediate for that instruction, otherwise the branch instruction will not work. I would imagine that address overflow is not performed for the relocation computation derived at a SETHI instruction, as the high 22 bits are masked out regardless, and will be valid, the same for low 10 bits. The result of the computation will at most be 32 bits, as this is the max size of the relocation type in 32 bit ELF and other object formats, as such there is a guarantee that patching the whole address into this relocation, a type of group relocation, that overflow cannot occur. Although if there are leftover bits, this would suggest overflow, so the linker must verify that the computed address will fit, and silently drop the top bits if not, rather than trigger an error or generate a veneer. The details of this are probably subject to the actual relocation type. More about validating the address width as being 32 bit rather than ensuring will fit into immediate, as will be masked anyway. 
References to symbols that are pseudo registers and thread local storage are resolved as offsets from the start of the segment, while normal symbol references are resolved as absolute addresses, why?
	Pseudo register symbols are those that mark base addresses used for addressing modes. For example $gp$ on MIPS is a global pointer base, points to 64K block of memory in heap that holds constants and global variables. These registers are not real objects, simply reference points for computing offsets. TLS symbols represent per-thread variables. `__thread int x`, the address of `x` is different in each thread, so the linker cannot know its absolute address at static link time. TLS symbols are referenced relative to a per-thread TLS segment base at runtime. Normal global symbols have one copy in the entire program, and are often fixed and resolved at link-time. Resolutions, are thus involving absolute addresses which can be determined at link-time, and subsequent relocations can be computed from these absolute addresses. Thus given that TLS and Pseudo register symbols do not have fixed, global addresses and instead relative to some base, the result of relocation operation is an offset according to the thread base or psuedo-register, rather than ever producing an absolute address or PC relative address that does not need to be modified beyond link-time. At runtime, the final address to be patched at the relocation is computed once the base register (thread pointer, GP, GOT base etc) is assigned to its concrete runtime value, necessitating a more complex form of relocation. 

We said that $a.out$ and $ECOFF$ relocation doesn't handle references like $A - B$ where $A$ and $B$ are both global symbols. Can you come up with a way to fake it?
	One, this relocation may be needed at run-time to compute boundaries. If A marks the start of some collection type and B marks the start of the next data structure, then A - B is the size of the data structure for A. If the final value isn't known yet, may require a relocation. PIC code, may need this to express relative distance between two objects which do not yet have assigned addresses. Given the above, relocations in a.out and ECOFF only permit relocations of the form `value = S + A` or `value = S + A - P`. The trick is make A and B be in the same section, compute their delta which is known at link-time. One emits a relocation choosing A as the relocation target, and encodes the addend as -offset_B, now fits the allowed form and computes the required operation. 
