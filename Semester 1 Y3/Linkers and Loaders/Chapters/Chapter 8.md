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

`mmap(MAP_FIXED, ...)` is problematic to execute in a process with an existing memory layout. To do so will often require load-time relocation. If the loader is running in the same process we are mapping to, the potential for mappings to clash exists and thus can lead to crashes. The use of the `fork exec` pattern makes a return! `fork()` creates a child-process with a copy-on-write duplicate of the parent's process image, thus we can avoid the crashes by preserving the parent process. We're still left with the potential issue of overwriting the loader's code the child has mapped COW. Even by linking the loader at unorthodox VMAs, we are making assumptions about the loader's candidate object. The only true solutions are to hand off to the kernel, guarantee no conflict resulting in potential load-time relocation.



## Position independent code and position independent executables

## Overlays