# Motivation
- <mark style="background: #FFF3A3A6;">Most</mark>, if not all modern platforms, <mark style="background: #FFF3A3A6;">use registers for computations</mark>
- <mark style="background: #FFF3A3A6;">Number and width of registers vary</mark>, but undeniably <mark style="background: #FFF3A3A6;">faster than memory</mark>
- So optimising values processor registers contains s<mark style="background: #FFF3A3A6;">peeds up program execution</mark>

The question becomes, "*which values do we assign to registers, and how should we do it, to maximise efficiency?*"


## Live Range
- <mark style="background: #FFF3A3A6;"> A variable is **live** if it holds a value that may be needed in the future; will be used before reassigned</mark>
- A **live range** is a <mark style="background: #FFF3A3A6;">segment of code where a variable</mark> is **live**
	- <mark style="background: #FFF3A3A6;"> I.e., all points in program from variable's first definition to its last usage, or when it is killed by a redef.</mark>
	- <mark style="background: #FFF3A3A6;"><mark style="background: #FFF3A3A6;">Redefinition begins a new live range, as the variable is killed. </mark></mark>
	- By convention, live ranges begin the instruction **after** the definition. 


## Interference
Two <mark style="background: #FFF3A3A6;">live ranges </mark>**interfere** if:
- They are <mark style="background: #FFF3A3A6;">live</mark> at the <mark style="background: #FFF3A3A6;">same program point</mark>
- The <mark style="background: #FFF3A3A6;">compiler</mark> <mark style="background: #FFF3A3A6;">can't prove</mark> that they <mark style="background: #FFF3A3A6;">have</mark> the <mark style="background: #FFF3A3A6;">same program value</mark>
If<mark style="background: #FFF3A3A6;"> two live ranges</mark> **interfere**, the variables <mark style="background: #FFF3A3A6;">they represent must be stored separately</mark>. Thus,<mark style="background: #FFF3A3A6;"> cannot share same register</mark>, as the values must be available simultaneously(<mark style="background: #FFF3A3A6;">except for copy operations</mark>).


### Calculating interference
The <mark style="background: #FFF3A3A6;">simplest way</mark> to<mark style="background: #FFF3A3A6;"> identify interferences</mark>:
1.<mark style="background: #FFF3A3A6;"> Identify all operations</mark> that <mark style="background: #FFF3A3A6;">define variables</mark>
1. <mark style="background: #FFF3A3A6;">Label all other live ranges that are live at that point. </mark>

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
- <mark style="background: #FFF3A3A6;">Each vertex represents a live range</mark>
- <mark style="background: #FFF3A3A6;">An edge exists </mark>between 2 vertices if the<mark style="background: #FFF3A3A6;"> live ranges they represent interfere</mark>

E.g., 
<mark style="background: #FFF3A3A6;">For the following code:</mark>
	$1: a = 5$
	$2: b = a + 1$
	$3: c = b * 2$
	$4: a = 7$
	$5: d = a + c$
	<mark style="background: #FFF3A3A6;">The only variables with overlapping range</mark> are $a$ and $c$ with variable $c$ being live at $4,5$ and $a$ being live at $5$.

$$V = \{LR_{a1}, LR_{c0}\} \ \text{and} \ E = \{ (LR_{a1}, LR_{c_{0}})\}$$

### Spilling
> A <mark style="background: #FFF3A3A6;">live range</mark> is said to be **spilled** when it <mark style="background: #FFF3A3A6;">cannot be kept</mark> in a <mark style="background: #FFF3A3A6;">register</mark> due to resource limitations, and <mark style="background: #FFF3A3A6;">must be committed</mark> to <mark style="background: #FFF3A3A6;">memory.</mark>

- A <mark style="background: #FFF3A3A6;">live range</mark> is **restored** when it is <mark style="background: #FFF3A3A6;">brought back from memory</mark> into a register. 

- A live range is **dirty** if its<mark style="background: #FFF3A3A6;"> current value </mark>has <mark style="background: #FFF3A3A6;">not previously been written</mark> to memory. <mark style="background: #FFF3A3A6;">Spilling a dirty live range</mark> requires a<mark style="background: #FFF3A3A6;"> store operation. 
</mark>
- A <mark style="background: #FFF3A3A6;">live range</mark> is **clean** if its<mark style="background: #FFF3A3A6;"> current value already exists in memory.</mark> Requires no action. <mark style="background: #FFF3A3A6;">Spilling a clean live range requires no further action. 
</mark>
#### Spill costs
- <mark style="background: #FFF3A3A6;">Spilling live range is costly,</mark> need to<mark style="background: #FFF3A3A6;"> estimate how costly</mark>
- **Spill cost**
	- <mark style="background: #FFF3A3A6;">Estimate the cos</mark>t of a single spill
	- <mark style="background: #FFF3A3A6;">Multiply</mark> by the <mark style="background: #FFF3A3A6;">estimated number of executions/uses of that variable.</mark>
	- FOR EXAM:
		- SPILL COST (number of loads + stores ) x memory cost
		- ![](Pasted%20image%2020260118212943.png)
		- ![](Pasted%20image%2020260118212845.png)
		- A has spill cost of 2010
		- F has spill cost of 2010
		- B has spill cost of 1000 etc

We can often take shortcuts to make very rough estimates:
- <mark style="background: #FFF3A3A6;">Assume</mark> that <mark style="background: #FFF3A3A6;">every loop</mark><mark style="background: #FFF3A3A6;"> executes 10 times</mark>
	- <mark style="background: #FFF3A3A6;">Block in a loop has a spill cost of 10</mark>, <mark style="background: #FFF3A3A6;">block in a nested loop has a spill cost of 100</mark>
- A <mark style="background: #FFF3A3A6;">live range</mark> that<mark style="background: #FFF3A3A6;"> loads and stores</mark> to the <mark style="background: #FFF3A3A6;">same address</mark> and <mark style="background: #FFF3A3A6;">does nothing else</mark>; <mark style="background: #FFF3A3A6;">redundant</mark>, spill is essentially 'free'.
- A <mark style="background: #FFF3A3A6;">very short range, receives an infinite cost</mark>, cost for committing variables which are short-lived is high compared to keeping it in register. 


### Graph colouring
Given a graph $G = \langle V, E \rangle$ and a set of colours $K$, <mark style="background: #FFF3A3A6;">find</mark> an <mark style="background: #FFF3A3A6;">assignment</mark> of <mark style="background: #FFF3A3A6;">colours</mark> to <mark style="background: #FFF3A3A6;">vertices</mark> such that <mark style="background: #FFF3A3A6;">no two connected vertices</mark><mark style="background: #FFF3A3A6;"> share a colour. </mark>

If we <mark style="background: #FFF3A3A6;">treat physical registers</mark> as <mark style="background: #FFF3A3A6;">colours</mark>, then<mark style="background: #FFF3A3A6;"> assigning colours</mark> to the <mark style="background: #FFF3A3A6;">interference graph </mark>is the<mark style="background: #FFF3A3A6;"> same</mark> as <mark style="background: #FFF3A3A6;">assigning registers</mark> to each <mark style="background: #FFF3A3A6;">live ranges. </mark>

If a <mark style="background: #FFF3A3A6;">colouring is impossible</mark>, <mark style="background: #FFF3A3A6;">shows</mark> that the<mark style="background: #FFF3A3A6;"> live range</mark> <mark style="background: #FFF3A3A6;">needs to be spilled. 
</mark>

#### Graph Colouring Algorithm
1. <mark style="background: #FFF3A3A6;">Construct the interference graph</mark>
2. <mark style="background: #FFF3A3A6;">Estimate global spill costs</mark>
3.<mark style="background: #FFF3A3A6;"> Simplify the graph</mark> (<mark style="background: #FFF3A3A6;">remove low-degree nodes</mark>,<mark style="background: #FFF3A3A6;"> choose spill candidates</mark>)
3. <mark style="background: #FFF3A3A6;">Reinsert nodes</mark> and <mark style="background: #FFF3A3A6;">assign registers</mark>
4. <mark style="background: #FFF3A3A6;">Spill live ranges </mark>that cannot be coloured with $K$.


##### Example
1. Build a graph
2. Estimate global spill costs (annotate the graph)
![](Pasted%20image%2020260118213207.png)
3. Simplify the gaph
	Assume $K$ physical registers (4 in this example.)
	<mark style="background: #FFF3A3A6;">Repeatedly find a node </mark>with $degree < K$, <mark style="background: #FFF3A3A6;">starting with those that have the lowest degree</mark>, and push onto a stack. 
	![](Pasted%20image%2020260118213404.png)
	If <mark style="background: #FFF3A3A6;">no node with lower degree exists</mark>, <mark style="background: #FFF3A3A6;">pick node</mark> with **lowest spill cost** each time, mark as spill candidate, remove. <mark style="background: #FFF3A3A6;">LOWEST DEGREE ON STACK FIRST, IF EQUAL, THEN LOWER SPILL COST GOES FIRST.</mark>
4. Rebuild the graph
	<mark style="background: #FFF3A3A6;">Pop nodes</mark> from stack<mark style="background: #FFF3A3A6;"> one by one</mark>
	<mark style="background: #FFF3A3A6;">Reinsert each node into graph, assign it colour different from its neighbours</mark>
<mark style="background: #FFF3A3A6;">	If no register available, uncoloured, must be spilled. 
5. Inserting spill code
	For every variable that couldn't get a register, allocate space in memory
	Insert:
	- **Store** to memory immediately after definition
	- **Load** from memory before every use</mark>
