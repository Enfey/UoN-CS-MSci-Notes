## Context sensitive analysis
- Phase, compiler pipeline, also called semantic elaboration, which adds semantic information to the compiler IR, and often begins the construction of further, semantically emboldening IRs.
- Scope checking, function call validation, type checking, rather than just structural verification. Enforces language rules e.g., control flow, reachability, scope and sym management, accessibility + visibility. 
- Phase is necessary, before emitting executable, must answer some questions:
	What kind of value is stored in a variable - types
	How big is x - must be known to be manipulated
	If x is a procedure, what arguments does it take and what does it return
	How long must x's value be preserved - lifetime
	Who allocates space for x and initialises it. 


### Translation
- Each stage of compiler translates one representation of a program into another
- To begin, need to cover source code into suitable IR to work with.
- **Syntax-directed translation**, method of translation guided by the syntax of the source program. 
- Compiler engineer defines how to produce specific IR structures from different sections of the parse tree. 
- Different parser generators, have their own methods for defining these actions - actions are some semantic effect e.g., building AST, producing intermediate code, based on the syntax when parsing. 


### Choice of IR
- AST, obvious choice, structure is already like the structure of the grammar; generally attach these rules to to the grammar, called an attribute grammar. 
- Linear IRs can work, but translation is more complex, representing loops and conditionals is more difficult; depends on talent of compiler engineer and constraints of parser generator. 


#### Structure of translation
-  Nearly all translation schemes are recursive
- The representation of a given grammar nonterminal depends on the representations of the other symbols that go into it. 
- $SExpr -> List -> ( ListContent )$ 
- In the above production, the representation of $SExpr$ depends on the list, which depends on the list content.


#### Embedded actions
- Introduced in yacc LALR(1) parser generator
- Grammar write, defines snippets of code that the parser generator inserts into its code when executing productions
- ![](Pasted%20image%2020260119161701.png)
- Becomes part of generated parser method and executes immediately at that point in parsing. 

#### Visitor Pattern
- Software **design pattern** that permits the separation of algorithms from the objects on which they operate. 
- One supplies a **VISITOR** object, with a method for every node type; the tree accepts the visitor, and dispatches to the correct visit method given the tree structure. 
- Don't have to modify the IR itself; define actions that can be separated. 
- After running grammar actions to build AST nodes, can run separate passes; supply VISITOR object, and visit root of parse tree to perform translation. 
- The traversal order is important; depends on the type of attributes the parse tree holds. 

#### Designing the IR
- Every language feature should be expressible in chosen IR
- Syntax-directed translators, generally just have access to details in current subtree
- IR data structures should be constructible using just this 'local' knowledge.

#### Auxiliary data structures
- Translation often needs extra information e.g., a symbol table, that is not practical to attach to a parse tree.
- Depending on the paradigm, can be done differently. 
- Visitors can be augmented to pass in these IRs, such that when visiting and translating, can create the symbol table on the fly and use its info; permitting that local knowledge suffices. 

### Memory layout
#### Lifetimes
> A lifetime is the duration which a variable is valid or in scope, from its creation, to its destruction. 

- Compiler must calculate lifetimes; determines where to store, and how long it needs to be stored. 
- Three kinds of lifetimes:
	1. **Automatic**
		- Created when control enters a scope(function, block)
		- Destroyed when control exits that scope
		- For optimised code, stored in registers only, otherwise present in a stack frame
	2. **Static**
		- Exist from program load until program termination(or last use)
		- .data, .rodata, .tbss, .bss. .tdata, .got.
	3. **Irregular/dynamic lifetime**
		- User controlled via allocation/deallocation, or managed by runtime environment(GC)
		- Lifetime not tied to lexical scope, compiler/runtime track and free
		- Objects, vectors etc. 

#### Storage locations
- Every type has size and alignment
- Must ensure that each value is placed at an address that is multiple of alignment, not regarding section alignment(and segment alignment).
- The place that data ends up in, depends on lifetime
	- **Static**
		- .data, .bss, .got.plt, .tdata, .tbss, .rodata etc. generally placed below heap in traditional memory model where the stack grows downwards and the heap grows upwards. 
	- **Automatic**
		- Accessed via frame pointer offset in given stack frame instantiated for a particular scope. 
	- **Irregular**
		- Heap-allocated. 
#### Storage layout
- Amount of memory needed for value depends on type
- Individual values simple to store
- Arrays, stored contiguously. 
	- Multi-dimenstional arrays have certain address computations, depending on access order
- Strings are contiguous sequences of characters
	- Can be null-terminated, or store the length as an extra word
- Structs/records record all their fields contiguously with alignment(unless packing with pragmas).

### Recording storage layout 
- To begin, each namespace gets its own separate memory area 
- This helps with relocating code later • Entities can be located with a base address and an offset 
- Base address is address of the first word in their section’s memory area 
- Offset is how many words ‘up’ from base the entity sits
- Relative address is then [base + offset] 
- Once actual location of base is determined, assign actual value. These are relatively relocated at runtime by ld.so or whatever dynamic linker/loader the toolchain is using. 

