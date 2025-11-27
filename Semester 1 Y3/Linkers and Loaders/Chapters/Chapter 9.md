# Shared libraries
> A shared library (often called a Dynamic Link Library or Dynamic Shared Object) is a PIE executable that is loaded and linked at runtime, intended to be shared amongst multiple programs simultaneously, rather than being copied into the executable at static-link time. 

Shared libraries necessitate `PIE`s; for ELF this amounts to an `ET_DYN` object, the reason for this is that it ensures the library can be mapped into the address space at any address without requiring drastic load-time relocation. 

Shared libraries further export symbols that are resolved to their final memory addresses at runtime, sometimes lazily at runtime via `.plt` for function calls, and generally via `.got`. This resolution mechanism is known as **dynamic linking**, and is expanded upon in [[Chapter 10]].

The primary benefit acquired from the use of shared libraries is memory optimisation; whereby the kernel guarantees that the text segment resides in one shared copy on physical RAM. Writable data segments are mapped to processes **Copy-on-Write**(CoW) to preserve separation of process states.
### Binding Time Issues
Shared libraries introduce unique issues that don't exist with fully static, conventionally linked executables.

The most basic error is a missing library. Since the code isn't inside the executable, if the OS cannot find the library at runtime, the program crashes immediately due to failing resolution. The library must be present and compatible when the application runs.

Levine's linkers and loaders book describes an older model called 'Static Shared Libraries'. In this model, library addresses were fixed at link time, and if the library was updated and the addresses of functions or data shifted, the calling program would jump to the wrong memory location. Significant updates inevitably shifted addresses, meaning programs were either rebuilt, or the system must support multiple versions of the same library. 


### Address space management

### Shared library structure

### Creating shared libraries
