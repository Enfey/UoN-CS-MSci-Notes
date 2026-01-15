Initialisers and Finalisers, in a modern context, refer to code that runs before `crtx.o` calls `main()` and as the process is exiting, respectively.

In ELF, this amounts to some modifications made to `.dynamic` and some distinct sections who hold arrays of function pointers:

- `DT_PREINIT_ARRAY`/`.preinit_array`
- `DT_INIT_ARRAY`/`.init_array`
- `DT_FINI_ARRAY`/`fini_array

The runtime executes arrays in order, except for .`fini_array`, which is executed in reverse.

Before initialisation functions for an object $A$ are called, the initialisation functions for any other objects that object $A$ depend on are called, taking a dependency-first, depth-first approach. The initialisation of objects occurs by recursing through the needed entries of each object. The order of initialisation for circularly dependent objects is entirely undefined.


## Static linker responsibilities
- Must collect and coalesce incoming ctor/dtor sections into final sections with well defined ordering according to the dependency chain (and the subsequent reverse, for `.fini_array`)
- We must further construct the dynamic tags once the layout is known. 
- Array entries are function pointers and generally require relocations, and must be emitted correctly, the runtime will resolve such relocations before calling constructors.