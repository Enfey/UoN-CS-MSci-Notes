### Top-level
- MLIR is **reusable compiler infrastructure** which permits someone to implement their own semantics on top.
- If you wanted to construct a cutting-edge compiler for a new language, it would be laborious to write your own parser infrastructure, expressive intermediate representations, graph manipulation utilities, rewrite engine, dataflow analysis, code-generation and optimisation, etc
- MLIR provides many of the above for free, only requiring the ability to target the dialect you define. 

### Compiler IR
> An IR is a machine-agnostic representation placed between front-end and back-end.
- Naturally these are denoted at different levels of abstraction to permit certain optimisations, analyse/annotate the program, and progress toward code-generation.
- The structural organisation of an IR has generally has a strong impact on how the compiler considers analysis, optimisation, and code generation. 
	- E.g., treelike/graphlike IRs will naturally lead to passes structured as some form of treewalk. Similarly linear IRs (representing machine-agnostic pseudocode e.g., SSA naturally lead to passes/partial passes that iterate over operations in-order)
- Common intermediate representations include:
	- **Directed Acyclic Graph**
	- **SSA**
	- **Control-Flow Graph**
	- More on this stuff in my prior compilers notes.
### MLIR
- MLIR's intermediate representation is what we'll call composable.
- That is, one can define their own elements that correspond to MLIR's form. 
- **Operations** are the central unit of computation and structure in MLIR. 
	- Everything from simple multiplication to a complete source module or a function declaration is represented as an operation
	- An operation is fully open and extensible, and consists of the following components:
		- **Opaque name** - prefixed by the dialect identifier
		- **Operands** - some inputs consumed by an operation
		- **Results** - SSA value produced by the operation
		- **Attributes** - metadata attached to an operation or another IR object, this can be literally anything and is quite powerful for representing semantics.
		- **Regions** - 
		- **Source location**
