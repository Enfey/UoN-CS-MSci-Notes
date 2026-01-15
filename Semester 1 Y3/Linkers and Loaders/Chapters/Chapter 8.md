# Loading and Overlays
> A loader is an operating system component responsible for loading programs and libraries. 

## Basic Loading (statically linked `ET_EXEC` in userspace)
A statically linked ELF executable has all symbol resolution enacted and relocations applied by the linker. The loader has the responsibility of placing load segments in the process address space (or physical RAM on bare-metal), ensuring memory protections and initialisation, and transfer of control to `e_entry`.

### Preconditions
A correct `Elf32_Ehdr` and a program header table `ELf32_Phdr[]` with one or more `PT_LOAD` segments. For each `PT_LOAD` the linker has previously set `p_vaddr`, `p_offset`, `p_filesz`, and `p_memsz` and `p_flags`. Alignment in `p_align` is also set reflecting the requirements of the execution environment. 

### Loader steps 
0. **Run in a single-threaded context.**
	A userspace loader must operate in this context when it starts tearing down or replacing the address space. Since we are effectively emulating `execve()` which discards the old address space, there is the possibility that during loading, the other threads' stacks and the instruction pointer's code pages are unmapped, resulting in a potential immediate crash. A call to `fork()` and performing the load in the child where the loader is multi-threaded would be advised to avoid further complication.
1. **Read and validate ELF header and program headers**
	`pread()` the `Elf32_Ehdr` at offset 0. Check `e_ident`, `e_type == ET_EXEC`, `e_machine == EM_ARM`. Read program headers at `e_phoff` (`ehdr.e_phnum * ehdr.e_phentsize`)
2. **For each `PT_LOAD`, map at the linked virtual address.**
	The goal here is to place each loadable segment at `p_vaddr`. One uses `mmap()` and the `MAP_FIXED` flag (places mapping exactly at specified address, its only safe use is where the address space trying to be mapped was previously reserved by another mapping ) with file backing for `p_filesz` and anonymous mapped (`MAP_ANONYMOUS`) zero pages where `p_memsz` exceeds `p_filesz`. One can zero a portion of the page rather than assigning a blank mapping simply using `memset()`.
		Some key maths here for page size P:
			`map_addr = p_vaddr & ~(P-1)` (rounding)
			`offset_in_page` = `p_vaddr - map_addr` (where segment will begin in page)
			`map_offset` = `p_offset & ~(P-1)` (ELF allows file offset to be unaligned relative to page size, must be aligned for `mmap()`)
			`map_len = round_up(offset_in_page + p_filesz, P)` (computes total amount of file-backed data must map)
3. **Finalise protections**
	This doesn't necessarily apply here, as the permissions are congruent with the initial segment flags. But file-backed mappings may be created with RW to facilitate relocations or to facilitate [[RELRO]] and must have their permissions changed after the mapping and permissions have been established, this is achieved via calls to `mprotect()`.
4. **Compute Auxilary Vector(s)**
	ELF auxilary vectors are a mechanism to transfer certain kernel level information. This is emulated in a user-space loader; the loader will put an array of `auxv` vectors on the stack. Most of the contents of an aux vector are not needed for a static linker. An auxilary vector is defined in `/usr/include/elf.h` as:
	```C
	typedef struct {
		uint32_t a_type;
		union{
			uint32_t a_val;
		} a_un;
	} Elf32_auxv_t;
	```
	A useful vector for static linking may be that which denotes the system page size: `AT_PAGESZ`. There are plenty of others defined [here](https://man7.org/linux/man-pages/man3/getauxval.3.html).
5. **Stack construction and startup information dump**
	In our effort to emulate `execve()`, the construction of a new memory region to behave. `MAP_STACK` is a no-op on Linux currently, this requires the userspace loader to do more manual work. The userspace loader must allocate a new stack that is at an address 'high enough' and the space allocated is 'large enough'...  We push startup information onto the new stack so that the loaded program can use it exactly as if the kernel had executed `execve()`. This includes:
	- **auxilary vectors** - `(type, value)` pairs used by libc and dynamic linker, ended via `(AT_NULL, 0)`
	- **argv** - command line argument pointers and their strings. 
	- **envp** - environment variable pointers and their strings. These can come from `getenv()` if we wish to inherit the loader's environment. 
	Once the stack layout is complete, we set the CPU's stack pointer register (R13 in our case) to the final stack pointer value.
	It should be noted that certain binaries require specific auxilary vectors. Startup code from CRT objects (`crtx.o`) is what executes first after the entry point is jumped to, parsing the initial stack, running constructors, and ultimately calling main. For correct program execution, the initial stack provided must match the format expected by CRT. It should be noted that we are responsible for manually setting up guard pages using `mprotect(PROT_NONE)` at the bottom of the stack. Furthermore, the exact ordering of contents on the stack matters.
6. **Set up CPU state and transfer control**
	One must at least set the stack pointer in CPU to the value of the `sp` as denoted in the userspace loader, and set the PC to the address of the entry point. In our case, this is all that is required, CRT pulls `argc/argv/envp` from the stack, the ARM EABI does not mandate that these are passed via registers. Must ensure the instruction cache is coherent with the newly mapped pages before jumping to the entry point. GCC provides a builtin:
	 (`__builtin___clear_cache((char*)start, (char*)(end))`);
	 this must be called (or some other equivalent function) for each executable region mapped to prevent stale cache lines from corrupting execution via invalid instruction execution. If required, perform mode switch, can be derived by checking low bit of entry point address. After this, we can jump to the entry point. 
7. **Execution and Termination**
	After the jump, the first instructions run are in the CRT: `_start` extracts `argc/argv/envp/auxv` from the stack. Thread-local storage is setup and thread-pointer is initialised. `.init` and `.init_array` constructors are ran. Then `main(argc, argv, envp)` is called, and return from `main()` goes to `exit()` not to the loader.

## Load-time relocation for statically linked `ET_EXEC`
Concept in executable loading whereby the code or data has transformation directives exacted upon it to validate the program for execution at a different memory location to the desired link-time address(s).
### When is load-time relocation required for `ET_EXEC`?
It is extremely uncommon for `ET_EXEC` to require load-time relocation. It supplies an object with fixed, link-time assigned virtual addresses. The code may be position independent. The kernel loader will not require relocation or a workaround given that it establishes a new, private address space. 

`mmap(MAP_FIXED, ...)` is problematic to execute in a process with an existing memory layout. To do so will often require load-time relocation. If the loader is running in the same process we are mapping to, the potential for mappings to clash exists and thus can lead to crashes. The use of the `fork exec` pattern makes a return! `fork()` creates a child-process with a copy-on-write duplicate of the parent's process image, thus we can avoid the crashes by preserving the parent process.

We're still left with the potential issue of overwriting the loader's code the child has mapped COW. Even by linking the loader at unorthodox VMAs, we are making assumptions about the loader's candidate object. 

The only true solutions are to hand off to the kernel which can grant a new private address space for a process, or to relocate the loader to avoid overwriting. An implementation of a [userspace loader](https://github.com/olsner/elfload)which implements the 2nd trick is publicly available, named a 'trampoline loader'. I am led to believe this is similar to what QEMU and Wine do. Given that `ET_EXEC` files are often not expecting relocation, relocating the actual object is usually not an option. It is possible though, some linkers will generate small `.rel/a{name}` tables for absolute pointers, and enforce position-independent-code where possible.

If relocation is truly needed, then heroic binary-rewriting techniques will be required. If the linker was for some reason courteous enough to provide relocation entries, then these will factor in: `base_delta = new_base - old_base` to the relocations such that they are valid. It is fundamentally unreliable to scan through a binary and do this without relocation info, this will manifest as a heuristic and not a guarantee.
## Position independent code and position independent executables 

### Position independent code
Code that can execute correctly regardless of the absolute address at which it is loaded. It avoids embedding absolute addresses in instructions or data; it instead uses PC-relative addressing or indirection through a table like the **`.got`.** Total enforcement of this will yield a **Position Independent Executable(PIE)**. This notion permits sharing of code among a number of processes, we talk more about this in [[Chapter 9]].
### Position independent executables
This refers to an an executable object constructed such that it can be loaded at any base address. For $*nix$ this manifests as an `ET_DYN` object that is used as a main program. PIE enables ASLR of the executable image, and enables shared libs. Separates compile/link-time address assumptions from runtime, but such executables rely heavily on load-time relocation. 

### Motivation
The above exist for a few related reasons:
- **Memory sharing and flexibility**
	If code does not contain absolute addresses, the same code pages can be mapped into multiple processes at arbitrary virtual addresses. Shared libs are the canonical example. 
- **Security (ASLR)**
	If the main executable is built as a PIE, the loader can place it at a randomised base, raising the bar for memory corruption exploits. 
- **ABI & Tooling simplicity**
	Using PIC/PIE standardises how code obtains addresses: either PC-relative addressing or through a small per-object table. This avoids needing different calling conventions for PIC vs non-PIC. However, we pay cost for the received benefits; function prologues and epilogues now have to save and restore a `.got` register, PIC code is bigger and slower than non-PIC code, and the load-time relocations increase startup time. 


### Implementation techniques (ELF)

#### Global Offset Table
The global offset table is a per-object array of pointers instantiated by the linker to support PIC. Its purpose is to break the dependency between code and absolute addresses. Instead of embedding the final address of a symbol in the instruction stream, PIC code utilising the `.got` loads the address indirectly through a slot in the `.got`.

In ELF objects, the `.got` is placed by the linker in the `.got` and `.got.plt` sections, which belong to the data segment (`PT_LOAD` with `PF_W` enabled). This is chosen as the `.got` must be writable, and the `.got` must be close to the code. The linker will also emit a synthetic symbol `_GLOBAL_OFFSET_TABLE_` representing the absolute virtual base address of the `.got` which will be resolved at load-time. The way this symbol is used varies.

In static `ET_EXEC`, the `.got` does not appear, all addresses are fixed and known at link-time. In `ET_EXEC` with dynamic linking, the `.got` is needed for external symbols defined in shared libraries. `ET_DYN` with static linking, holds the absolute runtime addresses of references (often for data and function pointers in code). `ET_DYN` with dynamic linking holds the absolute runtime addresses of all external symbols and is used as central lookup table by runtime loader to enable communication with other components.

On architectures without rich PC-rel addressing, PIC uses `.got` heavily.
##### How `.got` references are generated
PIC capable compilers emit relocation-annotated instructions whenever a symbol must be referenced through .`.got` . For ARM the usual pattern is:
1. Emit a literal pool entry that will hold the `.got`'s absolute address
2. Emit a PC-relative LDR instruction to read that literal
3. The assembler attaches a `.got` related relocation at the literal, such that the word will be filled with the absolute address of the `.got`.
4. At static link time, the static linker relocates the literal with a PC-relative offset to the `.got` entry (for `.got`-PC relocations) or the absolute address of a `.got` slot (in PIE/DL this is relocated again once the load-bias and final `.got` address is known. )

##### Creating the `.got`
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

##### `GOTOFF`
 For local static variables which are not exported for linking, we may choose to provide `GOTOFF` style access, that is instead of providing a `.got` entry, the code stores the variable at an offset from the `.got` base. We could do this with data as the relative distance between code and data won't change, but this provides a uniform access model, and some ISAs have limitations on their offsets for PC-relative addressing, with the `.got` we can ensure the data is not farther than the instruction range allows.
##### Advanced `.got` 

###### The undecided nature of `.got` entries
When one compiles with -fpic, the compiler cannot safely determine:
- Whether a global variable reference will remain local after linking
- Whether the symbol could be interposed by another shared object or library
	- Symbol interposition is the ability for a symbol reference in a shared object to be resolved at runtime to a different definition than the one that exists in the same shared object. Only symbols that are STB_GLOBAL with default visibility, or reside in shared objects can be interposed and are said to be 'pre-emptible'. Note that if a symbol has been defined internally, then it is non-interposable; a DSO can override references, not definitions. 
Because of this uncertainty, the compiler **always** generates references as if they require a `.got` entry, even when the the definition is in the same source file, and even if the symbol turns out to be static, non-interposable, and/or resolvable at link-time. 

The linker must later determine the true nature of the symbol. If it later found to be non-preemptible, the linker can often suppress the dynamic relocation and use a cheaper relative relocations instead, perhaps rewriting the access to be `.got`OFF, or resolving it entirely at link-time traditionally. This is known as **`.got` relaxation.** We detail another potential optimisation below.

###### Code transformation
For non-preemptible symbols, the `.got` indirection becomes functionally unnecessary. For example, on x86-64:
```C
movq var@`.got`PCREL(%rip), %rax
movl (%rax), %eax
```
There is an extra memory-load from the `.got` slot. The symbol's address is fixed relative to the object's base. Some ABIs like `x86-64` define `.got` optimisation (e.g., using relocations like `R_X86-64_REX_`.got`PCRELX`), allowing a linker to detect that the `.got` indirection is unnecessary based on whether the symbol is non-preemptible, and permitting a rewrite for non-p using a PC-relative load instead.
```C
movl var(%rip), %eax
```

#### Procedure Linkage Table
This is an ELF mechanism for dynamic function calls. Its primary function is to allow PIC to call functions in shared libraries without knowing their final addresses at link-time. It implements lazy binding, that is, the deferring of resolution of external function addresses until those functions are actually called for the first time, significantly reducing program startup time. This table is a helper to the workaround that PC-relative calls to shared-library functions are impossible without some indirection at run-time. The PLT works in a collaborative capacity with the `.got` as will be demonstrated below.

The `PLT` is placed in an ELF object as the `.plt` section. The PLT header `PLT[0]` serves as the **resolver stub**, that is, the entry point to the dynamic linker's resolution logic. When execution lands here, it pushes identifying information onto the stack(or uses specific registers, depending on architecture) and then jumps to the dynamic linker's runtime resolution routine e.g., `_dl_runtime_resolve`. This transfers control from the program to the dynamic linker. For every unique external function called by the module, there is a dedicated PLT stub. 

Though it should be noted, the interpretation of the PLT depends on the architecture heavily. 

##### First calls
When code first calls an external function e.g., `call printf@plt`, the linker redirects this call to the corresponding `PLT` entry. The `PLT` entry's first instruction jumps indirectly to the function's corresponding slot in the `.got.plt`. The `.got.plt` is a subsection of the `.got` reserved for PLT-related entries, and each PLT entry has a corresponding entry in `.got.plt[N]` that initially contains a pointer to `.plt[0]`, set by the static linker.  Once the `PLT` header transfers control to the dynamic linker, the linker uses the relocation offset (pushed as an argument) to find the correct relocation entry. It then performs a symbol lookup to find the function's true absolute runtime address, $S$. The dynamic linker overwrites the content of `.plt.got[N]` with by exacting the relocation operation implied e.g., `R_ARM_JUMP_SLOT`. Execution can then resume.

##### Future calls
The program calls the function again. The PLT jumps indirectly, executing  `jmp *.got[N]`. Since `.got[N]` now holds the functions true absolute address, the indirect jump transfers control directly to the function's entry point.

##### Additional info:  [[RELRO]], eager binding
Only the `.got.plt` entries are updated during lazy binding. This leaves the `.got.plt` writable for an arbitrary amount of time; exposing valuable attack surface for binary exploitation. Eager binding and [[RELRO]]aim to solve this issue.

During startup, before ld.so transfers control to the component, the dynamic linker resolves relocations. If eager binding is requested explicitly, the dynamic linker resolves all R_ARM_JUMP_SLOT or equivalent relocations immediately. Each `.got.plt` slot is then guaranteed to hold the address of the target function, at the cost of increased startup time. Full `RELRO` transitions the `.`.got`.plt` region into the read-only segment via `mprotect()` calls to make `.got.plt` immutable for the remainder of execution. 

There are a few ways to request eager binding depending on toolchain:
- Link-time flags
	`Wl, -z, now` - forces non-lazy binding
	`Wl, -z, bindnow` - synonym for `-z now`
	`Wl, -z, relro -Wl, -z, now` - Full RELRO with eager binding
- Runtime flags
	`LD_BIND_NOW=1` - environmental variable understood by `ld.so`, equivalent to forcing `-z now` but applied at runtime, triggering eager binding if the binary was statically linked without `-z now`.
- ELF flags
	Some toolchains set `DF_BIND_NOW` in the ELF dynamic flags, which has the same effect as `-z now`
It should be noted that `.got` has the `SHF_WRITE` flag, and traditionally it is also always writable, despite only containing relocations which are eagerly resolved. [[RELRO]] will also place this into `PT_GNU_RELRO` if the flag is invoked.
#### PC-relative addressing
> Addressing mode where the target memory address is calculated by taking the current instruction address ($PC, rip$ etc) and adding a displacement that is encoded as an immediate. 

### Caveats
Discussed briefly again here for clarity.
- **Extra indirection** 
	Global variables which are pre-emptible are accessed via the `.got`, and function calls which are external in shared object, or calls to pre-emptible global functions are via the PLT. Each memory access adds an additional load.
- **Startup overhead**
	For full [[RELRO]]'d `.got`, there is a noticeable increase in startup time as all symbols must be resolved before the program starts execution.
- **Code size**
	- `.got`/PLT accesses may require multiple instructions, especially on RISC and 'unorthodox architectures'.
- **Slower first-calls** 
	Lazy binding, but this is essentially a tradeoff for faster startup. You have either one or the other, not both. 

## Bootstrap loading
Up to now, discussions have presumed there is already on OS or at least a program loader as a kernel component. The chain of programs being loaded by other programs must begin somewhere. The question then arises: how is the first program loaded into the computer?

The answer is that one needs a chain of increasingly capable loaders.
Early stages run with minimal CPU state, tiny memory, and zero drivers; later stages progressively add capabilities (DRAM init, paging, filesystem drivers, crypto, etc). A single monolithic loader cannot live in the MBR, so booting is a staged process that trades off the minimal trusted code in ROM against larger, more flexible loader code on disk. 

### `x86` staged bootstrap - Legacy BIOS
CPU begins in `real-mode`. BIOS stored at fixed addresses in ROM, typically at `0xFFFFFFF0h` for `x86` processors - this is known as the **Reset Vector**. Traditionally firmware performs `POST` and device initialisation searching for the boot device and reading the first sector (512 bytes) known as the `MBR` - the **Master Boot Record.** The `MBR` contains a small first-stage loader (446 bytes) and partition table. It precedes the first partition. Because of size limits, it typically just loads the a slightly larger second stage loader from a well-known location e.g., `GRUB stage2`. 

The larger **bootloader** lives in unpartitioned space, or on disk files. and can contain file system drivers (`FAT`, `ext4` etc), configuration parsing, and more. It loads the kernel image (often `vmlinuz`) and an `initramfs`, and passes boot options. The kernel image is commonly compressed, known as `vmlinuz`, and the loader decompresses it into memory. On `x86`, the boot protocol historically includes a small `real-mode` setup header that establishes protected or long mode before jumping into the kernel proper. The kernel extracts `initrafmf`s, enables paging and other devices, mounts root via `initfrafms` and starts `init`(`PID 1`)or `systemd`. From initialising the kernel, programs can be loaded, program loaders are kernel components. 

### x86 detailed bootstrap - UEFI
On x86-64 the CPU resets into a legacy-compatible state, but the firmware immediately switches to 32-bit and then 64-bit mode extremely early. [Long Mode](https://en.wikipedia.org/wiki/Long_mode) is active before any before any bootloader runs. TODO.

### User-space bootstrap
When the kernel `execve(2)`s a user program, the user-space bootstrap is as follows:
1. Kernel maps program segments 
	`PT_LOAD` segmets are mapped into the address space, for `ET_EXEC` the addresses are fixed; for `ET_DYN` the kernel maps with a random base, see [[ASLR]].
2. `PT_INTERP`
	The kernel maps the interpreter and transfers control to it if it exists, otherwise the kernel jumps directly into `_start` of the binary, transferring control to `crtx.o` which ultimately calls `main()`.
3. Kernel prepares initial stack
	Adds `argc` , `argv[]`, `envp[]`, and auxilary vectors `auxv`.
4. Dynamic linker responsibilities
	Build `link_map`, `DT_NEEDED` resolution, `mmap()` DSOs
	Compute `load_bias` for each object
	Apply relocations with an ordering, filling `.got` and `.got.plt` entries as required.
	Initialise TLS, call `.init_array` constructors, and then jump to `_start`. 

#### Explanation of the lack of ARM example
Generally more fragmented than x86 as ARM hardware is not standardised, but the general principles of bootstrapping loader remain the same, what the stages perform may just look slightly different. 

## Overlays (memory, historical, and how to somewhat achieve them now)
> Overlays are a memory management technique in which a program's code and/or data is partitioned into distinct **modules**, only a subset of which - the resident set - is kept in physical memory at any given time. The working set of a program at time $t$ is the subset of overlay modules required to execute the currently active control paths. Formally, let $\mathcal{M} = \{\mathcal{M}_1, ... \mathcal{M}_n\}$ denote the set of overlay modules of a program. The overlay system enforces: $$ W_{t} \Subset R_{t} \Subset \mathcal{M}, \ \ \ \forall{t}$$
> Where:
> 	$W_t$ is the **working set** at time $t$,
> 	$R_t$ is the resident set at time $t$. 

Prior to paging and virtual memory, overlays allowed programs to have a virtual memory footprint large than physical memory, by ensuring that only the working set resides in RAM at once, or satisfying some other constraint. 

### Tree-Overlays
Overlay method whereby overlay modules are organised into a hierarchy such that only a minimal amount of code must reside in memory at any given time. Widely used before virtual memory was common and still relevant in memory-constrained embedded systems.

![[Pasted image 20251126210801.png]]

The above is a **tree of overlay segments**, where each node contains some code or data that constitutes part of the program image. 

The **root segment** is always resident. It contains the program entry point, startup code, and global/persistent data. A generic **overlay segment** is a group of routines and optionally private data that can be loaded and unloaded together. Only one child in a **sibling group/segment** can reside in RAM at the same time. The **path** is the sequence of segments from **root to target segment.** To execute a routine in a leaf segment, the **overlay manager** must ensure all segments along the path are loaded into memory. Upward calls do not require overlay operations. 

#### Defining Tree Overlays and Implementation of overlays
Defining tree overlays happens at link-time - overlays are entirely a software trick managed by the linker and a runtime overlay manager. The goal is to organise the programs into segments and specify which parts of code share memory, such that the overlay manager can perform effectively at runtime. 


##### Linker Script Definition - GCC and LLVM
LD and LLD support overlays via linker scripts. They use the `OVERLAY` command to distinguish between:
- `VMA` - Where the code executes.
- `LMA` - Where the overlay segment is stored.

An example GCC linker script is provided below:
```C
SECTIONS {
    /* Root */
    .text : { *(.text) }

    /* Overlay definition */
    OVERLAY : NOCROSSREFS {
        .overlay_A { *(.overlay_A_code) }
        .overlay_B { *(.overlay_B_code) }
    } > RAM AT > FLASH
}
```
##### Implementation of Overlays
A small runtime kernel is included in the root segment, which acts as a runtime system for unloading and loading overlay segments on demand. 

Calls to routines in overlays are replaced with a jump to glue code which saves processor state, and redirects to the overlay manager, passing a token indicating which segment is needed. A segment table entry may look like this:

```C
typedef struct segtab {
	const void*    flash_addr;
	void*          p_addr;
	size_t         size;
	int            is_present;
	int            sibling_group;
} segtab;
```
For a downward call:
1. Check if the target segment is loaded
2. If not loaded:
	Load it from the ROM/Flash into `p_addr`
	Unload conflicting siblings in the same memory region
	Update `is_present` flag in `segtab`.
3. Jump to target routine






