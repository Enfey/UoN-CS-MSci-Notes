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

The only true solutions are to hand off to the kernel which can grant a new private address space for a process, or to relocate the loader to avoid overwriting. An implementation of a [userspace loader](https://github.com/olsner/elfload)which implements the 2nd trick is publicly available, named a 'trampoline loader'. I am led to believe this is similar to what QEMU and Wine do. Given that ET_EXEC files are often not expecting relocation, relocating the actual object is usually not an option. It is possible though, some linkers will generate small `.rel/a{name}` tables for absolute pointers, and enforce position-independent-code where possible

If relocation is truly needed, then heroic binary-rewriting techniques will be required. If the linker was for some reason courteous enough to provide relocation entries, then these will factor in: `base_delta = new_base - old_base` to the relocations such that they are valid. It is fundamentally unreliable to scan through a binary and do this without relocation info, this will manifest as a heuristic and not a guarantee.
## Position independent code and position independent executables 

### Position independent code
Code that can execute correctly regardless of the absolute address at which it is loaded. It avoids embedding absolute addresses in instructions or data; it instead uses PC-relative addressing or indirection through a table like the **GOT.** Total enforcement of this will yield a **Position Independent Executable(PIE)**. This notion permits sharing of code among a number of processes, we talk more about this in [[Chapter 9]].
### Position independent executables
This refers to an an executable object constructed such that it can be loaded at any base address. For $*nix$ this manifests as an `ET_DYN` object that is used as a main program. PIE enables ASLR of the executable image, and enables shared libs. Separates compile/link-time address assumptions from runtime, but such executables rely heavily on load-time relocation. 

### Motivation
The above exist for a few related reasons:
- **Memory sharing and flexibility**
	If code does not contain absolute addresses, the same code pages can be mapped into multiple processes at arbitrary virtual addresses. Shared libs are the canonical example. 
- **Security (ASLR)**
	If the main executable is built as a PIE, the loader can place it at a randomised base, raising the bar for memory corruption exploits. 
- **ABI & Tooling simplicity**
	Using PIC/PIE standardises how code obtains addresses: either PC-relative addressing or through a small per-object table. This avoids needing different calling conventions for PIC vs non-PIC. However, we pay cost for the received benefits; function prologues and epilogues now have to save and restore a GOT register, PIC code is bigger and slower than non-PIC code, and the load-time relocations increase startup time. 


### Implementation techniques (ELF)

#### Global Offset Table
The global offset table is a per-object array of pointers instantiated by the linker to support PIC. Its purpose is to break the dependency between code and absolute addresses. Instead of embedding the final address of a symbol in the instruction stream, PIC code utilising the GOT loads the address indirectly through a slot in the GOT.

In ELF objects, the GOT is placed by the linker in the .got and .got.plt sections, which belong to the data segment (PT_LOAD with PF_W enabled). This is chosen as the GOT must be writable, and the GOT must be close to the code. The linker will also emit a synthetic symbol `_GLOBAL_OFFSET_TABLE_` representing the absolute virtual base address of the GOT which will be resolved at load-time. The way this symbol is used varies.

In static ET_EXEC, the GOT does not appear, all addresses are fixed and known at link-time. In ET_EXEC with dynamic linking, the GOT is needed for external symbols defined in shared libraries. ET_DYN with static linking, holds the absolute runtime addresses of references (often for data and function pointers in code). ET_DYN with dynamic linking holds the absolute runtime addresses of all external symbols and is used as central lookup table by runtime loader to enable communication with other components.

On architectures without rich PC-rel addressing, PIC uses GOT heavily.
##### How GOT references are generated
PIC capable compilers emit relocation-annotated instructions whenever a symbol must be referenced through the GOT. For ARM the usual pattern is:
1. Emit a literal pool entry that will hold the GOT's absolute address
2. Emit a PC-relative LDR instruction to read that literal
3. The assembler attaches a GOT related relocation at the literal, such that the word will be filled with the absolute address of the GOT.
4. At static link time, the static linker relocates the literal with a PC-relative offset to the GOT entry (for GOT-PC relocations) or the absolute address of a GOT slot (in PIE/DL this is relocated again once the load-bias and final GOT address is known. )

##### Creating the GOT
During the link-step, all input relocations are processed. The GOT creation proceeds as follows:
1. Collect GOT-related relocations
	These include:
	- **R_ARM_GOT_PREL** - compute pc-relative offset of a symbol's GOT entry, $GOT_{sym} + A - P$ 
	- **R_ARM_GOT32** - rewrite word to contain absolute virtual address of GOT slot for $sym$, $GOT_{sym} + A$
	- **R_ARM_GOTPC** - compute the pc-relative offset of the GOT base address itself $GOT_{base} + A - P$ 
	or their architectural equivalents. Each relocation indicates 'this instruction expects symbol $S$ to be accessed via the GOT'.
2. Allocate GOT entries
	The linker creates the .got section and assigns each unique symbol referenced through the GOT its own slot. The slot holds a pointer which the loader will fill with the symbol's runtime address. ARM uses separate relocations for GOT-relative addresses (GOTOFF) vs actual GOT entries.
3. Emit dynamic relocation entries
	Generate relocation entries in the output binary's dynamic relocation sections. For every unique external symbol that received a GOT slot, the static linker performs the following:
	1. Select relocation type
		for GOT entries that hold the absolute address of a data symbol, this is typically R_ARM_GLOB_DAT. For entries related to function calls via the PLT, its typically R_ARM_JUMP_SLOT.
	2. Select offset
		The linker records the virtual address of the newly allocated GOT slot as the r_offset field in the dynamic relocation entry to indicate where the final address should be written. 
	3. Set the symbol index
		Sets the symbol index of the symbol $S$ into the r_info field to tell what address to look up. 

##### GOTOFF
 For local static variables which are not exported for linking, we may choose to provide `GOTOFF` style access, that is instead of providing a GOT entry, the code stores the variable at an offset from the GOT base. We could do this with data as the relative distance between code and data won't change, but this provides a uniform access model, and some ISAs have limitations on their offsets for PC-relative addressing, with the GOT we can ensure the data is not farther than the instruction range allows.
##### Advanced GOT facts

###### The undecided nature of GOT entries
When one compiles with -fpic, the compiler cannot safely determine:
- Whether a global variable reference will remain local after linking
- Whether the symbol could be interposed by another shared object or library
	- Symbol interposition is the ability for a symbol reference in a shared object to be resolved at runtime to a different definition than the one that exists in the same shared object. Only symbols that are STB_GLOBAL with default visibility, or reside in shared objects can be interposed and are said to be 'pre-emptible'
Because of this uncertainty, the compiler **always** generates references as if they require a GOT entry, even when the the definition is in the same source file, and even if the symbol turns out to be static, non-interposable, and/or resolvable at link-time. 

The linker must later determine the true nature of the symbol. If it later found to be non-preemptible, the linker can often suppress the dynamic relocation and use a cheaper relative relocations instead, perhaps rewriting the access to be GOTOFF, or resolving it entirely at link-time traditionally. This is known as **GOT relaxation.** We detail another potential optimisation below.

###### Code transformation
For non-preemptible symbols, the GOT indirection becomes functionally unnecessary. For example, on x86-64:
```C
movq var@GOTPCREL(%rip), %rax
movl (%rax), %eax
```
There is an extra memory-load from the GOT slot. The symbol's address is fixed relative to the object's base. Some ABIs like `x86-64` define GOT optimisation (e.g., using relocations like `R_X86-64_REX_GOTPCRELX`), allowing a linker to detect that the GOT indirection is unnecessary based on whether the symbol is non-preemptible, and permitting a rewrite for non-p using a PC-relative load instead.
```C
movl var(%rip), %eax
```

#### Procedure Linkage Table
This is an ELF mechanism for dynamic function calls. It supports lazy binding, resolving function addresses only when first called, and provide a workaround for the fact that PC-relative calls to external functions are impossible in fully dynamic code without some indirection.



Each dynamically linked function has a PLT entry in the .plt section. When code first calls an external function, the linker redirects this call to the corresponding PLT entry. The PLT entry's first instruction jumps indirectly to the function's address stored in a specific slot in the GOT/PLT section. Initially, GOT[N] does not hold the function's real address; it holds the address of a special resolver stub within the dynamic linker
#### PC-relative addressing
> Addressing mode where the target memory address is calculated by taking the current instruction address ($PC, rip$ etc) and adding a displacement that is encoded as an immediate. 

### Caveats
Discussed briefly again here for clarity.
- **Extra indirection** 
	Global variables which are pre-emptible are accessed via the GOT, and function calls which are external in shared object, or calls to pre-emptible global functions are via the PLT. Each memory access adds an additional load.
- **Startup overhead**
	For full [[RELRO]]'d GOT, there is a noticeable increase in startup time as all symbols must be resolved before the program starts execution.
- **Code size**
	- GOT/PLT accesses may require multiple instructions, especially on RISC and 'unorthodox architectures'.
- **Slower first-calls** 
	Lazy binding.

## Bootloading

## Overlays