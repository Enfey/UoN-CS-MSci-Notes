# Optimisation in Compilers
> An optimising compiler is a compiler designed to generate code that is optimised in aspects such as program execution time, memory usage, code size, and power consumption. Optimisation is generally implemented a sequence of optimisation transformations which transform either an IR or generated code for the target backend to produce a semantic equivalent which is optimised in some way.
## Constraints
Optimisation is limited by a number of factors. Producing perfectly optimal code is not possible since optimising one aspect often degrades performance for another. Thus optimisation is generally a collection of heuristic methods for improving resource usage. There is the obvious constraint that we can only introduce optimisations which preserve semantics and that we can prove such optimisations would not affect the semantics of the source program. 

## Scope of optimisation
Optimioation can be applied to different sized sections of the program. Choosing to optimise a smaller scope generally means simpler optimisation, but also means that the impact of that optimisation is limited.

| Scope           | Involves                                     |
| --------------- | -------------------------------------------- |
| Local           | Single basic block                           |
| Regional        | Connected blocks, less than a full procedure |
| Intraprocedural | An entire procedure                          |
| Interprocedural | Several procedures.                          |

### Local Optimisation
These use information local to a basic block. Since basic blocks contain no branches, these optimisations require minimal analysis, reducing time and storage requirements.
#### Local Value Numbering
A compiler optimisation that aims to find multiple instances of equivalent expressions (i.e., those that yield the same result), and then replace them with the first occurrence to eliminate redundant calculations. We assign a unique number to each operation and remember these associations. A psuedo algorithm is detailed below.

For each operation $T_{i}= L_i \ Op \ R_i$ 
- Let $V_1$ = $lookup(L_i)$ and $V_2 = lookup(R_i)$
- Create table entry for $T_i$ 
	- Will eventually get a value number assigned.
- Create key $k = \langle V_1, Op, V_2 \rangle$ 
	- Uniquely identify the computatiojn
- If table has key $k$:
	- Get name $X = lookup(k)$ and rewrite operation as $T_i = X$ 
- If the table does not have key $k$:
	- Create table entry for $k$, generate new value, and store $T_i$ and the value at $k$.
For example, for a sequence of the following:
$T_1 = a+b$ 
$T_2 = a + b$ 
$T_3 = T_1 * c$ 

We keep an initial symbol table:

| Name | Value |
| ---- | ----- |
| a    | 1     |
| b    | 2     |
| c    | 4     |
1. $V_1 = lookup(a)$ and $V_2 = lookup(b)$
2. Create table entry for $T_1$ - $T_1$ is the name, and its value is the abstract identifier meaning 'the value produced by $a+b$' that is unique
3. Create key $k = \langle V_1, Op, V_2 \rangle$ - this is $\langle 1, +, 2 \rangle$ 
4. The table does not have key $k$ therefore we insert the key, name, and value as necessary, and our table now looks like the following:

| Key                       | Name  | Value |
| ------------------------- | ----- | ----- |
|                           | a     | 1     |
|                           | b     | 2     |
|                           | c     | 4     |
| $\langle 1, +, 2 \rangle$ | $T_1$ | 3     |
We would do the same for $T_2$ to realise that we have the key $k$. Thus, we rewrite as $T_2 = T_1$ 

In practice, we usually separate tables such that there is a symbol table containing name to value numbers, and an expression table containing key to value number/representative operation mapping. 

##### Local value numbering extensions
- You can use known identities to simply operations. 
	- $b := a + 0$ simplifies directly to $b := a$ without expression table lookup needed.
- Commutative operations share value numbers
	- $a :=b * c$ should receive the same value as $a := c * b$
	- To achieve, normalise operands by storing the key as smaller value numbers first so both expressions get the same value number.
If both operands are constants, we may be able to compute the result directly and use this in place of variable lookups.
	If $a:= 2 + 3$ simply mark $a$ as having the constant value $5$ and then future uses of $a$ can then substitute the literal value 5, rather than referring to that variable/expression. This is **constant folding**.

![](Pasted%20image%2020251216051440.png)
This algorithm misses redundancies across blocks by resetting the value table at every block boundary. 
### Regional Optimisation
These operate in a region in the CFG that contains multiple blocks. Loops, trees, paths, extended basic blocks. 
#### Superlocal value numbering
Extends LVN across multiple basic blocks along a single control-flow path.

Formally an $EBB$ is a maximal set of blocks $B_1...B_n$ where each $B_i$ except $B_1$ has exactly one predecessor and that block is in the $EBB$.
![](Pasted%20image%2020251216052810.png)
We define $SVLN$ over $EBBs$ rather than arbitrary $CFG$ regions becuause $EBBs$ have a single entry point and contain no internal join points. As a result, there is no ambiguity about reachability; along any path within an $EBB$, all previously encountered instructions on that path have definitely executed. Therefore, there is also no need for $\phi$ functions which are required only at control-flow joins. Thus, an $EBB$ contains some desirable properties such as unambiguous execution order and the safe propagation of value information. An $EBB$ may be the loop header and parts of the loop body, or a block containing a logical test together with one of its successor blocks, provided those blocks have a single ancestor.


##### Algorithm
1. Identify $EBBs$ and add their entry blocks to a work list
2. For each block in work list:
	Initialise a value-number table at the $EBB$ entry and run value numbering on the block, producing a hash table, which is then extended.
	For each of the blocks successors:
		If this block is the only ancestor, apply $SVLN$ to that block using the shared table as the starting point.
		If the successor has several ancestors, add it to the work list to be processed linearly.
At conditional branches, you copy the table and propagate independently down each successor.
Furthermore, we kill on redefinitions, that is assign a new value number and remove previous mappings for the old name.


##### Example
1. Identify $EBB$
	$B_1:$
		$T_1 = a + b$
		$\mathrm{if\ (c)\ goto\ } B_{2} \mathrm{\ else\ } B_{3}$
	$B_2:$
		$T_2 = a + b$
		$d = T_2 * e$
	$B_3:$
		$T_3 = a+b$
		$f = T_{3} - e$
	Thus $EBB$ = $\{B_{1},B_{2}, B_{3}\}$ = $worklist$
2. Initialise the VN table at the EBB entry and process $B_1$

| Key                       | Name  | Value |
| ------------------------- | ----- | ----- |
|                           | $a$   | $1$   |
|                           | $b$   | $2$   |
|                           | $c$   | $3$   |
| $\langle 1, +, 2 \rangle$ | $T_1$ | $4$   |

3. Propagate copy of table down each each successor as we are at conditional branch. We then process $B_2$ as it appears next in the work list. As key $\langle 1, +, 2 \rangle$ exists, we rewrite $T_2$ to be $T_2 = T_1$. 

| Key                       | Name  | Value |
| ------------------------- | ----- | ----- |
|                           | $a$   | $1$   |
|                           | $b$   | $2$   |
|                           | $c$   | $3$   |
| $\langle 1, +, 2 \rangle$ | $T_1$ | $4$   |
|                           | $e$   | $5$   |
| $\langle 4, *, 5 \rangle$ | $d$   | $6$   |
4. Propagate the table from step 2 and process $B_3$. As key $\langle 1, +, 2 \rangle$ exists, we rewrite $T_3$ to be $T_3 = T_1$. 

| Key                       | Name  | Value |
| ------------------------- | ----- | ----- |
|                           | $a$   | $1$   |
|                           | $b$   | $2$   |
|                           | $c$   | $3$   |
| $\langle 1, +, 2 \rangle$ | $T_1$ | $4$   |
|                           | e     | 5     |
| $\langle 4, -, 5 \rangle$ | f     | 6     |

#### Loop unrolling
Loop transformation technique that attempts to optimise a programs execution speed at the expense of its binary size, an approach known as the space-time tradeoff. The transformation can be taken either manually by the programmer or by an optimising compiler. On modern processors, loop unrolling is often counterproductive as the increased code size can cause more cache misses. Unrolling a loop literally means turning a loop into linear code, and this is only possible if the number of iterations is static. 

The goal of loop unrolling is to increase speed by reducing or eliminating repeated instructions that control the loop such as pointer arithmetic and 'end of loop' tests on each iteration.
##### Advantages
- Minimisation of branch penalty
- If the statements in the loop are independent of each other, the statements can potentially be executed in parallel. 
- If the reduction in executed instructions compensates for any performance reduction caused by code size explosion, then it is worthwhile. Consequently, loop unrolling is applied selectively to loops that are identified as satisfying this trade-off. 

##### Disadvantages
- Increased binary sizes; problematic in resource constrained environments, thus this form of optimisation should be limited. 
- Reduced code readability where this optimisation is performed by hand. 

##### Examples
![](Pasted%20image%2020251216175935.png)
![](Pasted%20image%2020251216175941.png)
![](Pasted%20image%2020251216180021.png)
![](Pasted%20image%2020251216180025.png)
Halves branches, increment, and comparison, and improves ILP. 

## Order of Optimisations
The order of compiler optimisations matters as certain optimisations being placed eartlier can enable, disable, or change the profitability of later optimisations. Applying them in a poor order may miss opportunities or even undo previous improvements, while a good order maximises effectiveness and preserves correctness. 

One optimisation may create opportunities for another:
- Constant propagation is the process of substituing the values of known constants in expressions at compile time. 
	![](Pasted%20image%2020251216180516.png)
	Yields:
	![](Pasted%20image%2020251216180529.png)
- This further enables constant folding, which is the process of recognising and evaluating constant expressions at compile time. 
One optimisation may destroy information needed by another:
- The advanced optimisation known as code motion, if extremely aggressive, may inhibit value numbering optimisations. 

We generally apply optimisations that simplify the IR and expose structure first. Examples include constant propagation, constant folding, and dead code elimination. 