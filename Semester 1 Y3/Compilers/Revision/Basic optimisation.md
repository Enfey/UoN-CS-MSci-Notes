> Optimising compilere <mark style="background: #FFF3A3A6;">generate code</mark><mark style="background: #FFF3A3A6;"> optimised</mark> in aspects such as <mark style="background: #FFF3A3A6;">execution time,</mark> memory <mark style="background: #FFF3A3A6;">footprint</mark>, code size, <mark style="background: #FFF3A3A6;">power consumption. </mark>Generally implemented as<mark style="background: #FFF3A3A6;"> sequence of optimisation transformations</mark> to <mark style="background: #FFF3A3A6;">transform</mark> either <mark style="background: #FFF3A3A6;">IR</mark> or <mark style="background: #FFF3A3A6;">generated code</mark> for the target backend, producing a **semantic equivalent** which is optimised in some way.


## Constraints
***Can only introduce optimisations where we can prove the optimisation would not alter the semantics of the source program***

***Perfectly optimal code is not possible; optimising one aspect often degrades performance for another***
	Generally leads optimisation to be a collection of heuristic methods for improving resource usage. 
<mark style="background: #FFF3A3A6;">PERFECTLY OPTIMAL NOT POSSIBLE, MUST PROVE CANNOT ALTER SEMANTICS OF SOURCE PROGRAM. </mark>

## Scope of Optimisation
<mark style="background: #FFF3A3A6;">- Optimisation can be applied to different sized sections of the program</mark>
<mark style="background: #FFF3A3A6;">- Smaller scopes generally simpler to optimise, but more limited in impact. </mark>

| Scope           | Involves                                   |
| --------------- | ------------------------------------------ |
| Local           | Single basic block                         |
| Regional        | Connected blocks, less than full procedure |
| Intraprocedural | Full procedure                             |
| Interprocedural | Several procedures                         |


### Local optimisation
- <mark style="background: #FFF3A3A6;">Use info local</mark> to <mark style="background: #FFF3A3A6;">basic block</mark>
- <mark style="background: #FFF3A3A6;">Since basic blocks</mark> contain<mark style="background: #FFF3A3A6;"> no branches,</mark> these<mark style="background: #FFF3A3A6;"> optimisations </mark><mark style="background: #FFF3A3A6;">require</mark> <mark style="background: #FFF3A3A6;">minimal analysis,</mark> <mark style="background: #FFF3A3A6;">reducing time</mark>+<mark style="background: #FFF3A3A6;">storage requirements</mark>. 

#### Local Value Numbering
- <mark style="background: #FFF3A3A6;">Aims</mark> to find <mark style="background: #FFF3A3A6;">multiple instances </mark>of<mark style="background: #FFF3A3A6;"> equivalent expressions</mark>(those that yield the same result), <mark style="background: #FFF3A3A6;">replacing them</mark> with the <mark style="background: #FFF3A3A6;">first occurrence</mark>
- E<mark style="background: #FFF3A3A6;">liminates redundant calculations</mark>. 
- We assign a <mark style="background: #FFF3A3A6;">unique number</mark>/value to to track when they change. 

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
- If both operands are known constants, we may be able to <mark style="background: #FFF3A3A6;">compute the result directly</mark>, <mark style="background: #FFF3A3A6;">use this in place of variable lookups</mark>
	- If $a:= 2 + 3$ simply mark $a$ as having the constant value $5$ and then future uses of $a$ can then substitute the literal value 5, rather than referring to that variable/expression. This is **constant folding**.
	- <mark style="background: #FFF3A3A6;">By assigning value numbers to constants and expressions, when an operations operands have constant values when looked up(constant propagation), can evaluate the expression at compile time, and replace with the resulting constant.</mark>



### Regional optimisation
<mark style="background: #FFF3A3A6;">Operate </mark>in a <mark style="background: #FFF3A3A6;">region</mark> in the <mark style="background: #FFF3A3A6;">CFG</mark> that contain <mark style="background: #FFF3A3A6;">multiple blocks</mark>, loops, paths, trees, extended basic block particularly. 

#### Superlocal Value numbering and EBBs
Extend $LVN$ <mark style="background: #FFF3A3A6;">across multiple basic blocks along</mark> a single control-flow path. 

Formally, an $EBB$ is a <mark style="background: #FFF3A3A6;">maximal set of blocks</mark> $B_1 \dots B_{n}$ where each $B_i$ except $B_1$ has <mark style="background: #FFF3A3A6;">exactly one predecessor,</mark> and<mark style="background: #FFF3A3A6;"> that block</mark> is <mark style="background: #FFF3A3A6;">in the</mark> $EBB$.<mark style="background: #FFF3A3A6;"> Each block</mark> with $>1$ predecessor <mark style="background: #FFF3A3A6;">starts an</mark> $EBB$. 

They are a collection of basic blocks <mark style="background: #FFF3A3A6;">representing one path of execution</mark>; e.g., the block containing a logical test, and the <mark style="background: #FFF3A3A6;">block that will be executed if the test is true,</mark> or the <mark style="background: #FFF3A3A6;">block relating to a loop condition</mark>, and the block(s) relating to the loop body (ofc, providing single ancestor)

We define $SVLN$ over $EBBs$ rather than arbitrary $CFG$ regions, EBBs have <mark style="background: #FFF3A3A6;">single entry</mark>, <mark style="background: #FFF3A3A6;">no internal joins</mark>, <mark style="background: #FFF3A3A6;">no ambiguity about reachability</mark>; along any path in EBB, can be usre that all previously encountered blocks on that path have executed. Safe propagation of **value information** is permitted. 


##### Algorithm
1. <mark style="background: #FFF3A3A6;">Identify</mark> $EBBs$ and <mark style="background: #FFF3A3A6;">add their entry blocks</mark> to a <mark style="background: #FFF3A3A6;">work list</mark>
2. <mark style="background: #FFF3A3A6;">For each block in a work list</mark>
	1. Initialise a <mark style="background: #FFF3A3A6;">value-number table</mark> at the $EBB$ entry, and<mark style="background: #FFF3A3A6;"> run value numbering</mark> on the block,<mark style="background: #FFF3A3A6;"> producing a hash table</mark>, and <mark style="background: #FFF3A3A6;">save this hash table</mark>
	2. For <mark style="background: #FFF3A3A6;">each of the blocks successors</mark>
		1. <mark style="background: #FFF3A3A6;">If this block is the only ancestor for the successor</mark>, apply $SVLN$ to the successor block using the s<mark style="background: #FFF3A3A6;">hared hash table</mark>(if conditional branch, copy table and propagate independently to each successor)
		- <mark style="background: #FFF3A3A6;">If successor</mark> has <mark style="background: #FFF3A3A6;">several ancestors</mark>,<mark style="background: #FFF3A3A6;"> add to work list</mark> if not already in from step 1, and process linearly.

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
- <mark style="background: #FFF3A3A6;">Loop transformation technique,</mark> attempts to <mark style="background: #FFF3A3A6;">optimise program </mark>execution speed at expense of <mark style="background: #FFF3A3A6;">binary size</mark> (space-time tradeoff)
- Can be<mark style="background: #FFF3A3A6;"> done manually</mark>,<mark style="background: #FFF3A3A6;"> or by optimising compiler</mark>
- Often counterproductive, modern processors,<mark style="background: #FFF3A3A6;"> increased code size,</mark> <mark style="background: #FFF3A3A6;">cache pressure</mark>/subsequent misses.
- **Unrolling a loop means turning a loop into linear code**
	<mark style="background: #FFF3A3A6;">Reduces the num o</mark>f operations to<mark style="background: #FFF3A3A6;"> complete the loop</mark>, but <mark style="background: #FFF3A3A6;">increases code size</mark>
- <mark style="background: #FFF3A3A6;">Unrolling</mark> <mark style="background: #FFF3A3A6;">only possible</mark> if <mark style="background: #FFF3A3A6;">number of iterations</mark> is <mark style="background: #FFF3A3A6;">static.</mark>
- Goal =<mark style="background: #FFF3A3A6;"> increase speed </mark>by <mark style="background: #FFF3A3A6;">reducing</mark> or <mark style="background: #FFF3A3A6;">eliminating repeated</mark> <mark style="background: #FFF3A3A6;">instructions </mark>that <mark style="background: #FFF3A3A6;">control the loop</mark> e.g.,<mark style="background: #FFF3A3A6;"> pointer arithmetic</mark>, 'end of loop tests' on each iteration.

#### Advantages
- <mark style="background: #FFF3A3A6;">Minimisation </mark>of <mark style="background: #FFF3A3A6;">branch penalty</mark>
- <mark style="background: #FFF3A3A6;">If</mark> the <mark style="background: #FFF3A3A6;">statements</mark> <mark style="background: #FFF3A3A6;">in loop</mark> are <mark style="background: #FFF3A3A6;">independent of each other</mark>, <mark style="background: #FFF3A3A6;">can execute in parallel</mark>
-<mark style="background: #FFF3A3A6;"> If reduction</mark> in <mark style="background: #FFF3A3A6;">executed instructions</mark> <mark style="background: #FFF3A3A6;">compensates</mark> for<mark style="background: #FFF3A3A6;"> any performance reduction</mark> <mark style="background: #FFF3A3A6;">caused by code size explosion</mark>; <mark style="background: #FFF3A3A6;">generally use heuristics</mark> to <mark style="background: #FFF3A3A6;">identify loops</mark> that<mark style="background: #FFF3A3A6;"> satisfy this trade-off.</mark>

#### Disadvantages
-<mark style="background: #FFF3A3A6;"> Increased binary size</mark>; problematic, resource contrained envs, should limit this optmisation
<mark style="background: #FFF3A3A6;">- Reduced code readability</mark>

![](Pasted%20image%2020260118143210.png)

<mark style="background: #FFF3A3A6;">This is full unrolling</mark>, but <mark style="background: #FFF3A3A6;">one may choose to unroll partially,</mark> to say, <mark style="background: #FFF3A3A6;">halve the branch penalty </mark>and improve ILP with fully exploding code size. 
![](Pasted%20image%2020251216180025.png)

#### Order of optimisations
- <mark style="background: #FFF3A3A6;">The order of compiler optimisations</mark> <mark style="background: #FFF3A3A6;">matters</mark>c
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
