### Component overview 
- 2 main components, and 1 planned out
- All planned out according to my own notes, with inspiration/implementations that are derived elsewhere cited in the comments. 
- Features an AArch64 + ARM32 ELF parser as both a library and CLI tool
- Features a static linker accepting ET_REL objects and static archives to produce statically linked executables (both ET_DYN and ET_EXEC), able to link against musl, a standard library implementation, supports RELRO, symbol versioning with versioning scripts, many synthetic symbols, .got, .got.plt, .plt, .dynamic, dynamic reloc generation, relaxation for MOVT* relocations, archive support, essentially everything other than debug support, thumb support, and TLS support is provided in this static linker. 
- Features the concepts for a userspace loader which could theoretically unmap the majority of itself from memory; i discussed this prior and the challenges of such an implementation (which i am not sure exists), but did not get round to an implementation unfortunately.
### Capabilities
- 

### Tests that verify capabilities

### Lens
- Zero-copy (what do we mean by this)
- Additional things defined
- Error codes
- SetErr introduced
	- Context structures maintain an error message, and the enum of the error code that was last thrown
- Elf structure with discriminated union(as a library, only care for e32, we end up with some branches for word specific object contents, but the predictor learns quickly.)
- Expose API, which dispatch internally like mentioned prior, based on elf_class discriminator
- Parsing is generally defensive, and this style is maintained in the linker too e.g., use of built_in_overflow to ensure that ELF's with huge e_shnum or ph_num do not break lens.
- Two main open functions
	- open concerns itself with elf files that are on disk that do not currently reside in memory, mmap'd in, and that elf descriptor owns the memory
	- open_mem means that the caller owns the memory, for example, opening an archive, and we don't unmap the memory on error.
- Lots of pretty printing functions for use as a CLI tool rather than a library, supports AArch64 and ARM32, generally analogous to readelf -a, and supports all common relocation types, not types, section types etc as according to the processor supplement. 


## Meld
### Input (30 min)
### Symbols/GST (45 min)
### Archive support (45 min)
### Sections (35 min)
### Relocs (30 mins)
### Dynlink support (40 mins)
### Output (1hr 30)
### Layout (30 min)
### Synthetic (10 min)
### Link (20 min)



