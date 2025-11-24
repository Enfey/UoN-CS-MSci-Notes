# Loading and Overlays
> A loader is an operating system component responsible for loading programs and libraries. 

## Basic Loading (statically linked `ET_EXEC`)
A statically linked ELF executable has all symbol resolution enacted and relocations applied by the linker. The loader has the responsibility of placing load segments in the process address space (or physical RAM on bare-metal), ensuring memory protections and initialisation, and transfer of control to `e_entry`.

### Preconditions
A correct `Elf32_Ehdr` and a program header table `ELf32_Phdr[]` with one or more `PT_LOAD` segments. For each `PT_LOAD` the linker has previously set `p_vaddr`, `p_offset`, `p_filesz`, and `p_memsz` and `p_flags`. Proper page alignment in `p_align` is also set reflecting the requirements of the execution environment. 

### Loader steps for statically linked ELF with no MMU support
1. Read object headers
	Parse `Elf32_Ehdr` and program header table; section headers are linker metadata and are not required by the linker. 
2. Validate and determine memory requirements
	Validate the ELF magic number and machine type (For our case, validate that  `e_machine = EM_ARM`).  One determines the span of memory by iterating over the program headers of type `PT_LOAD` to determine:
	- The lowest `p_vaddr` - start of memory region to allocate
	- The highest `p_vaddr + p_memsz`
	- Total memory = `highest - lowest` (aligned)
3. 

### Loader steps for statically linked ELF with MMU support

## Load-Time Relocation
### Load-Time relocation with no MMU support

### Load-time relocation with MMU support

## PIC and PIE

### Loader steps
The loader performs the following actions in order. 

1. Read object headers
	Parse `Elf32_Ehdr` and program header table; section headers are linker metadata and are not required by the linker. 
2. Validate and determine memory requirements
	Validate the ELF magic and machine type (`EM_ARM` in our case.)
	The total virtual address span required by `PT_LOAD` segments is noted. This is given by computing the delta between each `p_vaddr` and summing them, save for the last segment, where one adds on `p_vaddr + p_memsz`(respecting `p_align`).
3. Create VMAs or reserve RAM
	MMU present: VMAs established for each `PT_LOAD` segment with flags derived from `p_flags` $\to$ `PROT_READ/WRITE/EXEC`$\to$ `VM_READ/WRITE/EXEC`. The VMAs are typically created with `MAP_PRIVATE`(COW updates and no updates to backing file) semantics and file-backed mappings for the portion up to `p_filesz`. Modern linux uses `mmap()` to create lazy-loaded file-backed pages.
	No MMU present: the loader reserves contiguous physical RAM regions at the run-time addresses `p_paddr`. One here often reads using I/O calls the bytes starting from `p_offset` into memory at `p_paddr`.
4. Initialise BSS and zero data.
	If `p_memsz` > `p_filesz` for a segment, the loader must ensure the remaining bytes are zeroed out.
	MMU present: using `user_clear()`, specifying the destination address in user space and the number of bytes to zero `(p_memsz - p_filesz)`.  `mmap()`'s `MAP_ANONYMOUS` is used to allocate pages whereby the mapping is not backed by any file, and the contents are initialised to zero. This is used to allocate space for `.bss` according to its program header info (often separate segment but does depend. 
	No MMU present: the loader explicitly zeroes the memory range `p_paddr + p_filesz` up to `p_paddr + p_memsz`
5. Set protections and finalise mapping
	After mapping and BSS initalisation, protections are set. In MMU systems, mmap() handles this. 
6. Stack, argv, envp, and auxilary vector (MMU-only)
7. Transfer control
	If `PT_INTERP` is present, the kernel transfers control to the interpreter (dynamic linker). For statically linked `ET_EXEC` programs, the kernel jumps to `e_entry`. 

## Loading with load-time relocation
A small number of systems still do load-time relocation for executables. Many do load time relocation of shared libraries. 

## Position independent Code
### ELF PIC and PIE