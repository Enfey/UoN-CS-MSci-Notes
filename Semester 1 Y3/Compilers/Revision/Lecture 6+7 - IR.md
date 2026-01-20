> An intermediate representation (IR) is a machine-agnostic representation placed between front-end(parsing+semantics) and back-end(codegen).

A well formed IR permits one to:
- <mark style="background: #FFF3A3A6;">Apply optimisations cleanly </mark>(e.g., constant folding, DCE)
- <mark style="background: #FFF3A3A6;">Separate language concerns</mark> from<mark style="background: #FFF3A3A6;"> machine concerns, </mark>
- <mark style="background: #FFF3A3A6;">Make retargeting easier;</mark> many backends can consume the same IR. 
Compiler <mark style="background: #FFF3A3A6;">generally</mark>, can <mark style="background: #FFF3A3A6;">only</mark> <mark style="background: #FFF3A3A6;">optimise details,</mark> that the <mark style="background: #FFF3A3A6;">IR exposes.</mark>

### Taxonomy of IRs
Can classify IRs according to the following
- **Structure**
	- Can be <mark style="background: #FFF3A3A6;">graphical </mark>(trees, graphs) providing abstract view of code,<mark style="background: #FFF3A3A6;"> linear</mark>(sequence of pseudo-operations with ordering), or a combination of the two
- **Level of abstraction**
	- <mark style="background: #FFF3A3A6;">Informs</mark> <mark style="background: #FFF3A3A6;">how much info</mark> <mark style="background: #FFF3A3A6;">about the source </mark>or<mark style="background: #FFF3A3A6;"> target language</mark> is <mark style="background: #FFF3A3A6;">encoded </mark>in the <mark style="background: #FFF3A3A6;">representation</mark>
- **Mode of use**
	- <mark style="background: #FFF3A3A6;">Determines if the IR is the official program representation</mark>(<mark style="background: #FFF3A3A6;">definitive</mark>), or is only used temporarily(<mark style="background: #FFF3A3A6;">derivative</mark>)
The structural organisation of an IR, has<mark style="background: #FFF3A3A6;"> strong impact,</mark> on how compiler write <mark style="background: #FFF3A3A6;">consider analysis</mark>, <mark style="background: #FFF3A3A6;">optimisation,</mark> code gen e.g., treelike lead to passes structured as some form of treewalk. 
### Graphical IRs
- <mark style="background: #FFF3A3A6;">Encode knowledge</mark> in <mark style="background: #FFF3A3A6;">graph</mark>, algorithms<mark style="background: #FFF3A3A6;"> expressed</mark> in terms of <mark style="background: #FFF3A3A6;">objects </mark>e.g., nodes, edges, trees, abstract and provide way to trivially <mark style="background: #FFF3A3A6;">navigate logically connected components</mark>.

#### Abstract Syntax Tree
- Concise, near-source level representation, retains essential structure of the parse tree(also technically an IR) but omits most nodes for nonterminals. 
- Due to<mark style="background: #FFF3A3A6;"> close correspondence to parse tree</mark>, can <mark style="background: #FFF3A3A6;">build directly after syntax analysis</mark> with some rules;<mark style="background: #FFF3A3A6;"> condense down</mark> to the<mark style="background: #FFF3A3A6;"> useful elements</mark> e.g., <mark style="background: #FFF3A3A6;">no need to represent semicolons at end of statements</mark>, omit nonterminals/extraneous that serve no purpose basically. 
![[Pasted image 20251022113659.png]]

#### Directed Acyclic Graphs
- <mark style="background: #FFF3A3A6;">Contraction of AST</mark> achieved via<mark style="background: #FFF3A3A6;"> sharing. </mark>
- In **acyclic graph**, <mark style="background: #FFF3A3A6;">no routes through the graph that start and end at the same node</mark> (never form closed loop)
- <mark style="background: #FFF3A3A6;">Identical subtrees</mark><mark style="background: #FFF3A3A6;"> instantiated once</mark>, <mark style="background: #FFF3A3A6;">potentially</mark> having<mark style="background: #FFF3A3A6;"> multiple parents </mark>(which makes it <mark style="background: #FFF3A3A6;">different from a tree</mark>), making a DAG more compact, often employed to expose redundancies
- <mark style="background: #FFF3A3A6;">Optimisation over AST basically</mark>, particularly regarding literal subexpressions. <mark style="background: #FFF3A3A6;">To reuse a subexpression</mark> when sharing though, <mark style="background: #FFF3A3A6;">must prove</mark> that<mark style="background: #FFF3A3A6;"> its value</mark> <mark style="background: #FFF3A3A6;">cannot change between uses</mark>, <mark style="background: #FFF3A3A6;">hard for expressions</mark> with <mark style="background: #FFF3A3A6;">assignments</mark> and <mark style="background: #FFF3A3A6;">function calls. </mark>
	![[Pasted image 20251022114017.png]]

### Linear IR
- <mark style="background: #FFF3A3A6;">Represent program</mark> as sequence of <mark style="background: #FFF3A3A6;">discrete instructions</mark>, typically resembling pseudocode for some instruction set. 
- Often <mark style="background: #FFF3A3A6;">compact,</mark> human readable, <mark style="background: #FFF3A3A6;">algorithms iterate over these sequences</mark>.
- Differ based on 'kinds' of instructions used
	- <mark style="background: #FFF3A3A6;">Some used abstract, idealised operations</mark>
	- <mark style="background: #FFF3A3A6;">Others resemble assembly quite closely</mark>
- The level of<mark style="background: #FFF3A3A6;"> abstraction</mark> will <mark style="background: #FFF3A3A6;">depend on purpose</mark>
- <mark style="background: #FFF3A3A6;">All levels of abstraction</mark> <mark style="background: #FFF3A3A6;">need </mark>some kind of <mark style="background: #FFF3A3A6;">branch structure</mark>
	- Allow <mark style="background: #FFF3A3A6;">labelling </mark>of specific instructions
	- Simple <mark style="background: #FFF3A3A6;">branch and compare</mark> instructions 
#### Stack-machine IR
-<mark style="background: #FFF3A3A6;"> Stack-machine code</mark> is a form of **one-address code** (uses<mark style="background: #FFF3A3A6;"> single explicit memory address</mark> for its operation)
- Utilises an <mark style="background: #FFF3A3A6;">implicit stack </mark>for <mark style="background: #FFF3A3A6;">operands</mark>
- <mark style="background: #FFF3A3A6;">Most operations,</mark> <mark style="background: #FFF3A3A6;">consume operands</mark> from stack, and<mark style="background: #FFF3A3A6;"> push the result of their computation back onto it</mark>
- Highly compact, Bytecode, CPython
![[Pasted image 20251022170053.png]]


#### Three-address code IR
- <mark style="background: #FFF3A3A6;">Models </mark>a <mark style="background: #FFF3A3A6;">machine</mark> where <mark style="background: #FFF3A3A6;">most operations</mark>, take<mark style="background: #FFF3A3A6;"> two operands</mark>,<mark style="background: #FFF3A3A6;"> produce a result</mark>
- Typically written in the form $i ←j \ op \ k$
- <mark style="background: #FFF3A3A6;">Format</mark> is <mark style="background: #FFF3A3A6;">attractive,</mark> reasonably <mark style="background: #FFF3A3A6;">compact,</mark> <mark style="background: #FFF3A3A6;">distinct names for operands</mark> and results <mark style="background: #FFF3A3A6;">grant opportunities</mark> for<mark style="background: #FFF3A3A6;"> reuse</mark> and <mark style="background: #FFF3A3A6;">optimisation</mark>
- TAC often implemented as a sequence of <mark style="background: #FFF3A3A6;">quadruples</mark>:
	- **Operation** to carry out
	- One or two **source** address for operands
	- **Destination address** to save the result.
![[Pasted image 20251022170100.png]]


### Hybrid IR
<mark style="background: #FFF3A3A6;">Combine elements of both graphical and linear IRs</mark>

#### Control Flow Graphs
- A $CFG$ is a directed graph $G = (N, E)$ that <mark style="background: #FFF3A3A6;">models the flow of control</mark> between <mark style="background: #FFF3A3A6;">basic blocks i</mark>n a program.
- Each node $n \in N$ represents a **basic block** - <mark style="background: #FFF3A3A6;">a maximal length sequence</mark> of<mark style="background: #FFF3A3A6;"> guaranteed</mark> <mark style="background: #FFF3A3A6;">in-order/branch-free</mark> code
	- With single-entry, and single-exit.
- <mark style="background: #FFF3A3A6;">Each edge</mark> $e = (n_i, n_j) \in E$, <mark style="background: #FFF3A3A6;">corresponds</mark> to <mark style="background: #FFF3A3A6;">possible transfer of control</mark> from one block to another
- Fundamental for <mark style="background: #FFF3A3A6;">control flow analysis </mark>and<mark style="background: #FFF3A3A6;"> global optimisation,</mark> <mark style="background: #FFF3A3A6;">dead code elimination,</mark> <mark style="background: #FFF3A3A6;">global register allocation</mark>
- $CFGs$ are often <mark style="background: #FFF3A3A6;">constructed from a linear IR initially</mark> to form a hybrid IR.

##### Constructing CFGs
Constructed from linear IR. 
1. Find **leaders**
	The <mark style="background: #FFF3A3A6;">initial operation</mark> of a **basic block** is called a<mark style="background: #FFF3A3A6;"> leader</mark>, an operation is a <mark style="background: #FFF3A3A6;">leader if:</mark>
		It <mark style="background: #FFF3A3A6;">has the first operation in the procedure</mark>
		It<mark style="background: #FFF3A3A6;"> has a label/jump target</mark>
		<mark style="background: #FFF3A3A6;">Any instruction</mark> <mark style="background: #FFF3A3A6;">following a jump</mark> <mark style="background: #FFF3A3A6;">is a leader</mark>
2. Form basic blocks
	<mark style="background: #FFF3A3A6;">Start from each leader,</mark> <mark style="background: #FFF3A3A6;">group consecutive instructions</mark> <mark style="background: #FFF3A3A6;">until the next instruction</mark> <mark style="background: #FFF3A3A6;">is another leader</mark> or<mark style="background: #FFF3A3A6;"> jump/branch/return encountered</mark> (an instruction that does not fall through).
3. <mark style="background: #FFF3A3A6;">Add CFG edges</mark>
	For a <mark style="background: #FFF3A3A6;">conditional jump</mark>
		One <mark style="background: #FFF3A3A6;">edge</mark> to <mark style="background: #FFF3A3A6;">jump target </mark>block
		One <mark style="background: #FFF3A3A6;">edge</mark> to<mark style="background: #FFF3A3A6;"> fall-through</mark> block
	For an <mark style="background: #FFF3A3A6;">unconditional jump</mark>
		One <mark style="background: #FFF3A3A6;">edge</mark> to <mark style="background: #FFF3A3A6;">target block</mark>
	For a <mark style="background: #FFF3A3A6;">block</mark> that <mark style="background: #FFF3A3A6;">does not end in a jump</mark>
		Add <mark style="background: #FFF3A3A6;">fall through edge</mark> to next block in program order.

##### Why don't function calls end a block
<mark style="background: #FFF3A3A6;">Different to unconditional jumps</mark>, conditional branches, and returns, which do end basic blocks, because <mark style="background: #FFF3A3A6;">calls will return control </mark>to the <mark style="background: #FFF3A3A6;">next instruction</mark>, thus <mark style="background: #FFF3A3A6;">executing in-order </mark>and <mark style="background: #FFF3A3A6;">preserving the properties of basic blocks. </mark>



#### Symbol table
- <mark style="background: #FFF3A3A6;">Central repository</mark>, <mark style="background: #FFF3A3A6;">names and values</mark>; integral component of compiler IR
- <mark style="background: #FFF3A3A6;">Maps name</mark> into <mark style="background: #FFF3A3A6;">various annotations</mark>, recording type, size, address information, needed for code generation.
- <mark style="background: #FFF3A3A6;">Coalesces information</mark> derived from different compiler components, providing central access; chief method for communicating between phases, alongside definitive IR. 

##### Symbol scopes
- Languages often have naming scopes/lexical scopes which control visibility
- Symbol tables need to represent these scoping rules.
- Symbol table may then be implemented as a<mark style="background: #FFF3A3A6;"> hierarchy of tables;</mark> either a linked list or a stack of tables, where each table represents a different scope.
- To look up an identifier:
	1. <mark style="background: #FFF3A3A6;">Check the table relating to the current scope</mark>
	2. If not found, <mark style="background: #FFF3A3A6;">proceed to previous table in chain </mark>(<mark style="background: #FFF3A3A6;">parent scope</mark>)
	3. <mark style="background: #FFF3A3A6;">Repeat until found</mark>, until reach root scope

<mark style="background: #FFF3A3A6;">Tree based tables = O(log n)</mark>, <mark style="background: #FFF3A3A6;">hash  = O(1)(amortised, hash function should distribute well, not having to rehash)</mark>, <mark style="background: #FFF3A3A6;">table elements should try to be small,</mark> but design not as impactful.


![[Pasted image 20251022160102.png]]

How do we represent the **namespace hierarchy**?

##### Namespaces 
<mark style="background: #FFF3A3A6;">Container,</mark> holding <mark style="background: #FFF3A3A6;">set of identifiers,</mark> allowing compiler to <mark style="background: #FFF3A3A6;">distinguish between identical names</mark> in<mark style="background: #FFF3A3A6;"> different contexts</mark>, each procedure generally creates new namespace, <mark style="background: #FFF3A3A6;">insulating local declarations</mark> from <mark style="background: #FFF3A3A6;">surrounding context. </mark>

Lexical scopes =<mark style="background: #FFF3A3A6;"> scopes nest in order they are encountered in program text</mark>, <mark style="background: #FFF3A3A6;">name refers to definition that is lexically closest to its use.</mark>

Language specific scopes, C defines multiple levels, global, local(procedure-level), and block level.

<mark style="background: #FFF3A3A6;">Can represent the namespace hierarchy</mark> via a <mark style="background: #FFF3A3A6;">sheaf ot tables</mark>, where a n<mark style="background: #FFF3A3A6;">ew symbol table </mark>is created upon <mark style="background: #FFF3A3A6;">each lexical scope</mark>. When reference to name occurs, search sequentially and access each table starting at current scope, moving outward. <mark style="background: #FFF3A3A6;">The result will yield the lexical nesting level</mark>, where the name is declared, and $o$, the <mark style="background: #FFF3A3A6;">offset within the data area of that level.</mark> 

### SSA
- <mark style="background: #FFF3A3A6;">IR</mark>+<mark style="background: #FFF3A3A6;">Naming discipline</mark> employed to <mark style="background: #FFF3A3A6;">encode detailed info</mark> about **data flow** and **control flow**
- In <mark style="background: #FFF3A3A6;">regular TAC</mark>, <mark style="background: #FFF3A3A6;">variables are reassigned many times</mark>, in SSA, each variable is assigned exactly once. <mark style="background: #FFF3A3A6;">EACH ASSIGNMENT MUST INTRODUCE A NEW NAME.</mark>
- <mark style="background: #FFF3A3A6;">In SSA form</mark>,<mark style="background: #FFF3A3A6;"> every new value</mark>, <mark style="background: #FFF3A3A6;">defined by exactly one operation,</mark> <mark style="background: #FFF3A3A6;">achieved</mark> b<mark style="background: #FFF3A3A6;">y renaming original variables</mark> in TAC with <mark style="background: #FFF3A3A6;">subscripts </mark>e.g., $x_0, x_1, \dots x_n$, such that<mark style="background: #FFF3A3A6;"> each new name</mark><mark style="background: #FFF3A3A6;"> corresponds</mark> to a <mark style="background: #FFF3A3A6;">unique definition. </mark>

![](Pasted%20image%2020260118130332.png)
<mark style="background: #FFF3A3A6;">Use phi function</mark> <mark style="background: #FFF3A3A6;">when variable value</mark> depends on which path through the code we took. 

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


