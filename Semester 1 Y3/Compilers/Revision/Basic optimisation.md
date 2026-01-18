> Optimising compilere generate code optimised in aspects such as execution time, memory footprint, code size, power consumption. Generally implemented as sequence of optimisation transformations to transform either IR or generated code for the target backend, producing a **semantic equivalent** which is optimised in some way.


## Constraints
***Can only introduce optimisations where we can prove the optimisation would not alter the semantics of the source program***

***Perfectly optimal code is not possible; optimising one aspect often degrades performance for another***
	Generally leads optimisation to be a collection of heuristic methods for improving resource usage. 


## Scope of Optimisation
- Optimisation can be applied to different sized sections of the program
- Smaller scopes generally simpler to optimise, but more limited in impact. 

| Scope           | Involves                                   |
| --------------- | ------------------------------------------ |
| Local           | Single basic block                         |
| Regional        | Connected blocks, less than full procedure |
| Intraprocedural | Full procedure                             |
| Interprocedural | Several procedures                         |


### Local optimisation
- Use info local to basic block
- Since basic blocks contain no branches, these optimisations require minimal analysis, reducing time+storage requirements. 

#### Local Value Numbering
- Aims to find multiple instances of equivalent expressions(those that yield the same result), replacing them with the first occurrence
- Eliminates redundant calculations. 
- We assign a unique number/value to to track when they change. 

For each operation  $T_{i}= L_i \ Op \ R_i$ 
- Let $V_1 = lookup(L_i)$ and $V_2 = lookup R_i$ 
- Create table entry for $T_i$
	- Will be given a value number eventually
- Create key $k = \langle V_1, Op, V_2 \rangle$
	- Uniquely identify the particular operation with those operands, see if can reuse
- If table has key $k$:
	- Get $name$ $X = lookup(k)$ and rewrite operations as $T_i = X$ 
- If the table does not have key $k$
	- Create table entry for $k$, generate new value, and store $T_i$ and the value at $k$. 

##### Example
![](Pasted%20image%2020260118133341.png)


| Key     | Name | Value |
| ------- | ---- | ----- |
|         | b    | 1     |
|         | c    | 2     |
| 1, +, 2 | a    | 3     |
|         | d    | 4     |
| 3, -, 4 | b    | 5     |
| 5, +, 2 | c    | 6     |
| b       | d    | 7     |
Rewritten as:
- a:= b + c
- b := a - d
- c := b + c
- d := b

In practice, generally maintain separate tables; symbol table containing name to value numbers.


##### Example 2
$t1 := x * x$
$t1 := t1 * x$
$t1 := t1 * 2$
$t2 := x * x$
$t2 := t2 * 3$
$t3 := t1 + t2$
$t4 := t3 + x$
$y := t4 + 1$

| Key        | Name | Value |
| ---------- | ---- | ----- |
|            | x    | 1     |
| <1, *, 1>  | t1   | 2     |
| <2, *, 1>  | t1   | 3     |
|            | 2    | 4     |
| <3, * 4>   | t1   | 5     |
| 2          | t2   | 6     |
|            | 3    | 7     |
| <2, *, 7>  | t2   | 8     |
| <5, +, 8>  | t3   | 9     |
| <9, +, 1>  | t4   | 10    |
|            | 1    | 11    |
| <10, + 11> | y    | 12    |
REUSE THE ORIGINAL LOCAL VALUE, INSTEAD OF CREATING NEW TABLE ENTRY, LOOK AT VALUE 8, see how it reuses 2 instead, in exam, wouldnt enter a table entry, see algorithm. 


##### Local value numbering extensions
- **Can use known identities to simplify operations**
	- E.g., $b := a + 0$ simplifies to $b := a$ without needing to utilise the table for 0
- **Commutative operations (yield same, no matter how ordered)**
	- **Should receive the same value.**
	- To achieve, normalise operands, by storing key as smaller value number first, so both expressions get the same value number/rewrite.
- If both operands are known constants, we may be able to compute the result directly, use this in place of variable lookups
	- If $a:= 2 + 3$ simply mark $a$ as having the constant value $5$ and then future uses of $a$ can then substitute the literal value 5, rather than referring to that variable/expression. This is **constant folding**.
	- <mark style="background: #FFF3A3A6;">By assigning value numbers to constants and expressions, when an operations operands have constant values when looked up(constant propagation), can evaluate the expression at compile time, and replace with the resulting constant.</mark>



### Regional optimisation
Operate in a region in the CFG that contain multiple blocks, loops, paths, trees, extended basic block particularly. 

#### Superlocal Value numbering and EBBs
Extend $LVN$ across multiple basic blocks along a single control-flow path. 

Formally, an $EBB$ is a maximal set of blocks $B_1 \dots B_{n}$ where each $B_i$ except $B_1$ has exactly one predecessor, and that block is in the $EBB$. Each block with $>1$ predecessor starts an $EBB$. 

They are a collection of basic blocks representing one path of execution; e.g., the block containing a logical test, and the block that will be executed if the test is true, or the block relating to a loop condition, and the block(s) relating to the loop body (ofc, providing single ancestor)

We define $SVLN$ over $EBBs$ rather than arbitrary $CFG$ regions, EBBs have single entry, no internal joins, no ambiguity about reachability; along any path in EBB, can be usre that all previously encountered blocks on that path have executed. Safe propagation of **value information** is permitted. 


##### Algorithm
1. Identify $EBBs$ and add their entry blocks to a work list
2. For each block in a work list
	1. Initialise a value-number table at the $EBB$ entry, and run value numbering on the block, producing a hash table, and save this hash table
	2. For each of the blocks successors
		1. If this block is the only ancestor for the successor, apply $SVLN$ to the successor block using the shared hash table(if conditional branch, copy table and propagate independently to each successor)
		- If successor has several ancestors, add to work list if not already in from step 1, and process linearly.

##### Example
1. Identify EBB.
	$B_1:$
		$T_1 = a + b$
		$\mathrm{if\ (c)\ goto\ } B_{2} \mathrm{\ else\ } B_{3}$
	$B_2:$
		$T_2 = a + b$
		$d = T_2 * e$
	$B_3:$
		$T_3 = a+b$
		$f = T_{3} - e$
	Thus $EBB = \{B_1, B_2, B_3\}$, $worklist = \{B_1\}$
2. Initialise $VN$ table at $EBB$ entry $B_1$ 


| Key     | Name  | Value |
| ------- | ----- | ----- |
|         | a     | 1     |
|         | b     | 2     |
| <1 + 2> | $T_1$ | 3     |
|         | c     | 4     |

3. For each successor where $B_1$ is the only ancestor: propagate $VN$ table, given conditional, propagate independently
	3.1 $B_2$ 

| Key       | Name  | Value |
| --------- | ----- | ----- |
|           | a     | 1     |
|           | b     | 2     |
| <1 + 2>   | $T_1$ | 3     |
|           | c     | 4     |
|           | e     | 5     |
| <3, *, 5> | d     | 6     |

And for $B_3$ 

| Key      | Name  | Value |
| -------- | ----- | ----- |
|          | a     | 1     |
|          | b     | 2     |
| <1 + 2>  | $T_1$ | 3     |
|          | c     | 4     |
|          | e     | 5     |
| <3, - 5> | f     | 6     |




### Loop unrolling
- Loop transformation technique, attempts to optimise program execution speed at expense of binary size (space-time tradeoff)
- Can be done manually, or by optimising compiler
- Often counterproductive, modern processors, increased code size, cache pressure/subsequent misses.
- **Unrolling a loop means turning a loop into linear code**
	Reduces the num of operations to complete the loop, but increases code size
- Unrolling only possible if number of iterations is static.
- Goal = increase speed by reducing or eliminating repeated instructions that control the loop e.g., pointer arithmetic, 'end of loop tests' on each iteration.

#### Advantages
- Minimisation of branch penalty
- If the statements in loop are independent of each other, can execute in parallel
- If reduction in executed instructions compensates for any performance reduction caused by code size explosion; generally use heuristics to identify loops that satisfy this trade-off.

#### Disadvantages
- Increased binary size; problematic, resource contrained envs, should limit this optmisation
- Reduced code readability

![](Pasted%20image%2020260118143210.png)

This is full unrolling, but one may choose to unroll partially, to say, halve the branch penalty and improve ILP with fully exploding code size. 
![](Pasted%20image%2020251216180025.png)

#### Order of optimisations
- The order of compiler optimisations mattersc
- Certain optimisations being placed earlier can enable, disable, or change the profitability of later optimisations
- A poor ordering may miss opportunities, or even undo improvements.
- A good ordering maximises effectiveness and preserves correctness(naturally).

E.g., constant propagation (potentially via LVN lookup), is the process of substituting the values of known constants in expressions at compile time.
![](Pasted%20image%2020251216180516.png)
becomes:
![](Pasted%20image%2020251216180529.png)
This enables constant folding; process of recognising + evaluating constant expressions at compile time. 

![](Pasted%20image%2020260118143210.png)
E.g., this exposed opportunity for constant propagation, then folding, and then dead variable elimination. 

One optimisation may destroy information needed by another:
- The advanced optimisation known as code motion, if extremely aggressive, may inhibit value numbering optimisations. 

Apply optimisations that simplify IR first e.g., constant propagation, constant folding, DCe. 





## Practice q's
1. Unroll this TAC
B1:
	i0 = 0
	goto B2
B2:
	t1 = a + b
	t2 = t1 * c
	t3 = a + b
	t4 = t3 * c
	i1 = i0 + 1
	i0 = i1
	t1 = a + b
	t2 = t1 * c
	t3 = a + b
	t4 = t3 * c
	i1 = i0 + 1
	i0 = i1
	t1 = a + b
	t2 = t1 * c
	t3 = a + b
	t4 = t3 * c
	i1 = i0 + 1
	i0 = i1
	t1 = a + b
	t2 = t1 * c
	t3 = a + b
	t4 = t3 * c
	i1 = i0 + 1
	i0 = i1
B4:





2. SVLN

B1:
  a = 1
  b = 2
  if a < b goto B2 else goto B3

B2:
  c = a + b
  goto B4

B3:
  c = a - b
  goto B4

B4:
  d = c * 2
  if d > 0 goto B5 else goto B6

B5:
  e = d + 1
  goto B7

B6:
  e = d - 1
  goto B7

B7:
  return e


EBBs = {B1, B2, B3}, {B4, B5, B6}, {B7}

| Key     | Name | Value |
| ------- | ---- | ----- |
|         | a    | 1     |
|         | b    | 2     |
| <1 + 2> | c    | 3     |
B1-> B2

| Key     | Name | Value |
| ------- | ---- | ----- |
|         | a    | 1     |
|         | b    | 2     |
| <1 - 2> | c    | 3     |
B1->b3


| Key     | Name | Value |
| ------- | ---- | ----- |
|         | c    | 1     |
|         | 2    | 2     |
| <1 * 2> | d    | 3     |

3. SSA conversion
