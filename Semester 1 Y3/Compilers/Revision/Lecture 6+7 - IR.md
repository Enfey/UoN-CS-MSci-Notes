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


### Hybrid IR
Combine elements of both graphical and linear IRs

#### Control Flow Graphs
- A $CFG$ is a directed graph $G = (N, E)$ that models the flow of control between basic blocks in a program.
- Each node $n \in N$ represents a **basic block** - a maximal length sequence of guaranteed <mark style="background: #FFF3A3A6;">in-order/branch-free</mark> code
	- With single-entry, and single-exit.
- Each edge $e = (n_i, n_j) \in E$, corresponds to possible transfer of control from one block to another
- Fundamental for control flow analysis and global optimisation, dead code elimination, global register allocation
- $CFGs$ are often constructed from a linear IR initially to form a hybrid IR.

##### Constructing CFGs
Constructed from linear IR. 
1. Find **leaders**
	The initial operation of a **basic block** is called a leader, an operation is a leader if:
		It has the first operation in the procedure
		It has a label/jump target
		Any instruction following a jump is a leader
2. Form basic blocks
	Start from each leader, group consecutive instructions until the next instruction is another leader or jump/branch/return encountered (an instruction that does not fall through).
3. Add CFG edges
	For a conditional jump
		One edge to jump target block
		One edge to fall-through block
	For an unconditional jump
		One edge to target block
	For a block that does not end in a jump
		Add fall through edge to next block in program order.

##### Why don't function calls end a block
Different to unconditional jumps, conditional branches, and returns, which do end basic blocks, because calls will return control to the next instruction, thus executing in-order and preserving the properties of basic blocks. 



#### Symbol table
- Central repository, names and values; integral component of compiler IR
- Maps name into various annotations, recording type, size, address information, needed for code generation.
- Coalesces information derived from different compiler components, providing central access; chief method for communicating between phases, alongside definitive IR. 

##### Symbol scopes
- Languages often have naming scopes/lexical scopes which control visibility
- Symbol tables need to represent these scoping rules.
- Symbol table may then be implemented as a hierarchy of tables; either a linked list or a stack of tables, where each table represents a different scope.
- To look up an identifier:
	1. Check the table relating to the current scope
	2. If not found, proceed to previous table in chain (parent scope)
	3. Repeat until found, until reach root scope

Tree based tables = O(log n), hash  = O(1)(amortised, hash function should distribute well, not having to rehash), table elements should try to be small, but design not as impactful.


![[Pasted image 20251022160102.png]]

How do we represent the **namespace hierarchy**?

##### Namespaces 
Container, holding set of identifiers, allowing compiler to distinguish between identical names in different contexts, each procedure generally creates new namespace, insulating local declarations from surrounding context. 

Lexical scopes = scopes nest in order they are encountered in program text, name refers to definition that is lexically closest to its use.

Language specific scopes, C defines multiple levels, global, local(procedure-level), and block level.

Can represent the namespace hierarchy via a sheaf ot tables, where a new symbol table is created upon each lexical scope. When reference to name occurs, search sequentially and access each table starting at current scope, moving outward. The result will yield the lexical nesting level, where the name is declared, and $o$, the offset within the data area of that level. 

### SSA
- IR+Naming discipline employed to encode detailed info about **data flow** and **control flow**
- In regular TAC, variables are reassigned many times, in SSA, each variable is assigned exactly once. EACH ASSIGNMENT MUST INTRODUCE A NEW NAME.
- In SSA form, every new value, defined by exactly one operation, achieved by renaming original variables in TAC with subscripts e.g., $x_0, x_1, \dots x_n$, such that each new name corresponds to a unique definition. 

![](Pasted%20image%2020260118130332.png)
Use phi function when variable value depends on which path through the code we took. 

## Call Graph 
Graphical representation used for interprocedural analysis and optimisation. Control-flow graph representing calling relationships between procedures. Each node represents a procedure and each edge $f, g$ indicates that procedure $f$ calls procedure $g$. Thus a cycle indicates recursive calls. 



### Questions
1. SSA conversion
B1:
  a = 1
  b = 2
  if a < b goto B2 else goto B3

B2:
  c = a + b
  d = c * 2
  goto B4

B3:
  c = a - b
  d = c * 2
  goto B4

B4:
  e = d + a
  return e

Becomes


B1:
 a_1 = 1
 b_1 = 2
 if a_1 < b_1 goto B2 else goto B3
B2
 c_1  = a_1 + b_1
 d_1 = c_1 * 2
 goto b4
B3
 c_2 = a_1 - b_1
 d_2 = c_2 * 2
B4
 e_1 = $\phi$ (d_2, d_1) + a_1
 return e_1

Actually b4 should be

B4
 d_3 = $\phi$ (d_2, d_1) 
 e_1 = d_3 + a_1
 return e_1

2. Convert parse tree into AST, then identify DAG from that which exposes redundancies

Keep top level expression, then the second most expression identifying there are 2 expressions in the overall group, then just keep the leaves and remove all the other nonterminals. In fact, just remove all NTs, the tree itself represents the operations, but may choose to keep them depending on pipeline; generally remove.

        *
       / \
      +   +
     / \ / \
    a  b +  c
       / \
      a   b
Cheers obsidian.
Can have directed edge from first add in right subtree to second add in right subtree to eliminate the left subtree entirely via sharing the subexpression. 


