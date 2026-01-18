# Motivation
- Most, if not all modern platforms, use registers for computations
- Number and width of registers vary, but undeniably faster than memory
- So optimising values processor registers contains speeds up program execution

The question becomes, "*which values do we assign to registers, and how should we do it, to maximise efficiency?*"


## Live Range
- <mark style="background: #FFF3A3A6;"> A variable is **live** if it holds a value that may be needed in the future; will be used before reassigned</mark>
- A **live range** is a segment of code where a variable is **live**
	- <mark style="background: #FFF3A3A6;"> I.e., all points in program from variable's first definition to its last usage, or when it is killed by a redef.</mark>
	- <mark style="background: #FFF3A3A6;">Redefinition begins a new live range, as the variable is killed. </mark>
	- By convention, live ranges begin the instruction **after** the definition. 


## Interference
Two live ranges **interfere** if:
- They are live at the same program point
- The compiler can't prove that they have the same program value
If two live ranges **interfere**, the variables they represent must be stored separately. Thus, cannot share same register, as the values must be available simultaneously(except for copy operations).


### Calculating interference
The simplest way to identify interferences:
1. Identify all operations that define variables
2. Label all other live ranges that are live at that point. 

For the given example in the slides:
	$t0 := a$
	$t1 := b$
	$t2 := c$
	$t3 := d$
	$t0 := t0 * 2$
	$t0 := t0 * t1$ 
	$t0 := t0 * t2$
	$t0 := t0 * t3$
	$a := t0$

DEAL WITH LHS ASSIGN
t0_0 = 2 5, t0_1 = 6, t0_2 = 7, t0_3 = 8,
t_1 = 3 - 6
t_2 = 4 - 7
t_3 = 5 - 8
a has no live range.


### Interference Graphs
- Each vertex represents a live range
- An edge exists between 2 vertices if the live ranges they represent interfere

E.g., 
For the following code:
	$1: a = 5$
	$2: b = a + 1$
	$3: c = b * 2$
	$4: a = 7$
	$5: d = a + c$
	The only variables with overlapping range are $a$ and $c$ with variable $c$ being live at $4,5$ and $a$ being live at $5$.

$$V = \{LR_{a1}, LR_{c0}\} \ \text{and} \ E = \{ (LR_{a1}, LR_{c_{0}})\}$$

### Spilling
> A live range is said to be **spilled** when it cannot be kept in a register due to resource limitations, and must be committed to memory.

- A live range is **restored** when it is brought back from memory into a register. 

- A live range is **dirty** if its current value has not previously been written to memory. Spilling a dirty live range requires a store operation. 

- A live range is **clean** if its current value already exists in memory. Requires no action. Spilling a clean live range requires no further action. 

#### Spill costs
- Spilling live range is costly, need to estimate how costly
- **Spill cost**
	- Estimate the cost of a single spill
	- Multiply by the estimated number of executions/uses of that variable.
	- FOR EXAM:
		- SPILL COST (number of loads + stores ) x memory cost
		- ![](Pasted%20image%2020260118212943.png)
		- ![](Pasted%20image%2020260118212845.png)
		- A has spill cost of 2010
		- F has spill cost of 2010
		- B has spill cost of 1000 etc

We can often take shortcuts to make very rough estimates:
- Assume that every loop executes 10 times
	- Block in a loop has a spill cost of 10, block in a nested loop has a spill cost of 100
- A live range that loads and stores to the same address and does nothing else; redundant, spill is essentially 'free'.
- A very short range, receives an infinite cost, cost for committing variables which are short-lived is high compared to keeping it in register. 


### Graph colouring
Given a graph $G = \langle V, E \rangle$ and a set of colours $K$, find an assignment of colours to vertices such that no two connected vertices share a colour. 

If we treat physical registers as colours, then assigning colours to the interference graph is the same as assigning registers to each live ranges. 

If a colouring is impossible, shows that the live range needs to be spilled. 


#### Graph Colouring Algorithm
1. Construct the interference graph
2. Estimate global spill costs
3. Simplify the graph (remove low-degree nodes, choose spill candidates)
4. Reinsert nodes and assign registers
5. Spill live ranges that cannot be coloured with $K$.


##### Example
1. Build a graph
2. Estimate global spill costs (annotate the graph)
![](Pasted%20image%2020260118213207.png)
3. Simplify the gaph
	Assume $K$ physical registers (4 in this example.)
	Repeatedly find a node with $degree < K$, starting with those that have the lowest degree, and push onto a stack. 
	![](Pasted%20image%2020260118213404.png)
	If no node with lower degree exists, pick node with **lowest spill cost** each time, mark as spill candidate, remove.
4. Rebuild the graph
	Pop nodes from stack one by one in order of spill cost (doubly ordered)
	Reinsert each node into graph, assign it colour different from its neighbours
	If no register available, uncoloured, must be spilled. 
5. Inserting spill code
	For every variable that couldn't get a register, allocate space in memory
	Insert:
	- **Store** to memory immediately after definition
	- **Load** from memory before every use
