# Motivation
> Most if not all modern platforms use registers for computations. The number and width of these registers vary, but they are undeniably faster that memory, so optimising the values processor registers contains speeds up program execution.

The question then becomes, *which values do we assign to registers, and how should we do it to maximise efficiency*?

## Live Ranges
Recall that a variable is live at a given point if it will be used before it is reassigned.

Thus:
> A **live range** is a segment of code where a variable is live. That is all the points in the program from the variable's first definition to its last usage. Redefinition instantiates a new live range. By convention, live ranges are often said to begin the instruction after the definition. 

The instruction level formulation for liveness is useful for the rest of this content:
$$LiveIn(i) = Use(i) \cup (LiveOut(i) \backslash Def(i))$$
For an instruction $i$ and a variable $v$, $v$ is live before instruction $i$ if it is used in instruction $i$ or it is live after instruction $i$ and instruction $i$ does not redefine $v$.
## Interference
Two live ranges **interfere** if:
- They are live at the same program point.
- The compiler cannot prove they have the same value.
If two live ranges interfere, the variables they represent must be stored separately, and thus cannot share the same register as the values must be available simultaneously (with the exception of copy operations).
### Calculating Interference
Cannot be arsed with the formal explanation rn. The Simplest way to identify interferences it to:
1. Identify all operations that define variables
2. Label all other live ranges that are live at that point as interfering. 

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
	$t_0$ live ranges are $2-5, 6, 7, 8, 9$
	$t_1$ live range is $3-6$
	$t_2$ live range is $4-7$
	$t_3$ live range is $5-8$
	$a$ has no live range

For the following code:
	$1: a = 5$
	$2: b = a + 1$
	$3: c = b * 2$
	$4: a = 7$
	$5: d = a + c$
	The only variables with overlapping range are $a$ and $c$ with variable $c$ being live at $4,5$ and $a$ being live at $5$.


#### Interference Graphs
Each vertex represents a live range. An edge exists between two vertices if the live ranges they represent interfere. As such, the interference graph for the previous example would simply be: $$V = \{LR_{a1}, LR_{c0}\} \ \text{and} \ E = \{ (LR_{a1}, LR_{c_{0}})\}$$
##### More complex example (Revision)
TODO

### Spilling
> A live range is said to be **spilled** when it cannot be kept in a register due to resource limitations and must be committed to memory. 

A range is **restored** when it is brought back from memory into a register.

A live range is **dirty** if its current value has not previously been written to memory. This forces a store operation. 

A live range is **clean** if its value already exists in memory. This requires no action. 

#### Spill costs
Spilling a live range is costly, but we must estimate *how costly*. **Spill cost** is a heuristic, usually estimated by estimating the cost of a single spil, multiplied by the estimated number of executions/uses of that variable.

We can often take shortcuts to make very rough estimates:
- Assume that every loop executes 10 times
	- Block in a loop has a spill cost of 10, block in a nested loop has a spill cost of 100
- A live range that loads and stores to the same address and does nothing else gets a negative spill cost, as the spill would be redundant; the spill is essentially 'free'.
- A very short range would receive an infinite cost, as the cost for committing variables which are extremely short-lived is high compared to keeping it in a register for the tiny fraction of code that the variable is live.

### Graph Colouring







