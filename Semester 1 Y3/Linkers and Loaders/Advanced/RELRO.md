GNU extension that makes parts of an ELF image read-only after the dynamic loader has applied relocations. There are 2 modes: 
1. **Partial RELRO**
	The linker creates a PT_GNU_RELRO segment spanning a region that the loader is to `mprotect()` read-only after dynamic relocations are applied (excluding `.got.plt` relocations)
2. Full **RELRO**
	Partial RELRO plus disabling lazy binding such that `.got.plt` is not runtime writable.
In many distributions partial or full may be default; the typical linker flags are `-Wl, -z,relro` and `Wl,-z,now` for partial and full, respectively.


## What RELRO actually protects
- The goal is to prevent runtime overwrites of relocation targets, such as `.got`
- The contents of the `PT_GNU_RELRO` segment vary by toolchain, but generally include sections that would facilitate GOT/PLT hijacks.
- With `Wl,-z,-now`, all PLT/JMPREL relocs are resolved at startup with no further writes to GOT/PLT necessary.

## Linker responsibilities 
- A static linker is responsible for emitting a distinct segment `PT_GNU_RELRO` covering the sections that should be made readonly later. This typically overlaps the tail of a writable `PT_LOAD` mapping.
- The RELRO region must be rounded out to page boundaries for the obvious reason that permissions are per page, thus `mprotect()` can only operate on a range that plays nicely with the system's page size.
- The tag `DT_BIND_NOW` should be set to force eager resolution for a dynamic linker. 