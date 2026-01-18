> An intermediate representation (IR) is a machine-agnostic representation placed between front-end(parsing+semantics) and back-end(codegen).

A well formed IR permits one to:
- Apply optimisations cleanly (e.g., constant folding, DCE)
- Separate language concerns from machine concerns, 
- Make retargeting easier; many backends can consume the same IR. 
Compiler generally, can only optimise details, that the IR exposes.

### Taxonomy of IRs
Can classify IRs according to the following
- **Structure**
	- Can be graphical (trees, graphs) providing abstract view of code, linear(sequence of pseudo-operations with ordering), or a combination of the two
- **Level of abstraction**
	- Informs how much info about the source or target language is encoded in the representation
- **Mode of use**
	- Determines if the IR is the official program representation(definitive), or is only used temporarily(derivative)
The structural organisation of an IR, has strong impact, on how compiler write consider analysis, optimisation, code gen e.g., treelike lead to passes structured as some form of treewalk. 
### Graphical IRs
- Encode knowledge in graph, algorithms expressed in terms of objects e.g., nodes, edges, trees, abstract and provide way to trivially navigate logically connected components.

#### Abstract Syntax Tree
- Concise, near-source level representation, retains essential structure of the parse tree(also technically an IR) but omits most nodes for nonterminals. 
- Due to close correspondence to parse tree, can build directly after syntax analysis with some rules; condense down to the useful elements e.g., no need to represent semicolons at end of statements, omit nonterminals/extraneous that serve no purpose basically. 
![[Pasted image 20251022113659.png]]

#### Directed Acyclic Graphs
- Contraction of AST achieved via sharing. 
- In **acyclic graph**, no routes through the graph that start and end at the same node (never form closed loop)
- Identical subtrees instantiated once, potentially having multiple parents (which makes it different from a tree), making a DAG more compact, often employed to expose redundancies
- Optimisation over AST basically, particularly regarding literal subexpressions. To reuse a subexpression when sharing though, must prove that its value cannot change between uses, hard for expressions with assignments and function calls. 
	![[Pasted image 20251022114017.png]]

### Linear IR
- Represent program as sequence of discrete instructions, typically resembling pseudocode for some instruction set. 
- Often compact, human readable, algorithms iterate over these sequences.
- Differ based on 'kinds' of instructions used
	- Some used abstract, idealised operations
	- Others resemble assembly quite closely
- The level of abstraction will depennd on purpose
- All levels of abstraction need some kind of branch structure
	- Allow labelling of specific instructions
	- Simple branch and compare instructions 
#### Stack-machine IR
- Stack-machine code is a form of **one-address code** (uses single explicit memory address for its operation)
- Utilises an implicit stack for operands
- Most operations, consume operands from stack, and push the result of their computation back onto it
- Highly compact, Bytecode, CPython
![[Pasted image 20251022170053.png]]


#### Three-address code IR
- Models a machine where most operations, take two operands, produce a result
- Typically written in the form $i ←j \ op \ k$
- Format is attractive, reasonably compact, distinct names for operands and results grant opportunities for reuse and optimisation
- TAC often implemented as a sequence of quadruples:
	- **Operation** to carry out
	- One or two **source** address for operands
	- **Destination address** to save the result.
![[Pasted image 20251022170100.png]]