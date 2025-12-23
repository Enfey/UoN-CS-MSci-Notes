# Dead Code Elimination
## Dead Code
> Dead Code refers to instructions that do not affect the result of the program. This can be divided into **2** categories:
> 	1. **Useless code** is evaluated/executed, but doesn't change the result of the program.
> 	2. **Unreachable code** is code that would potentially affect the result of the program, but exists in a branch that can be proved to never run. 

Useless code and unreachable code increase code size, memory footprint, and can cause unnecessary use of the CPU's instruction cache. It should be noted that code may become unreachable/useless as a consequence of transformations performed by an optimising compiler.

Detection of unreachable and useless code take the form of control flow analysis.

### DCE strategy
At a high-level, it takes the form of a two-phase 'mark and sweep' algorithm:

1. **Mark useful operations**
	Start from operations that must be preserved, and work backwards.
2. **Sweep**
	Remove or replace everything that is non-useful i.e., wasn't marked.
#### Useful operations
The following operations are always useful:
- Setting a procedure return value
- Performing I/O
- Modifying a storage location that may be accessed outside the local scope.
- Unconditional branches
For an operation $x := y \ op \ z$
- The operations that define $y$ and $z$ are useful
- If block $b$ contains a useful operation, then any conditional branch in a block $c$ that **postdominates** $b$ is useful, because that branch determines which continuation of execution occurs after $b$, and thus affects the observable behavior caused by the useful operation. Removing this branch may prevent execution of useful code.

#### Domination(Strict)
A block $B_{1}$ is said to **strictly dominate** a block $B_{2}$ if it is impossible to reach $B_2$ without passing through $B_1$ first. That is every path from the start block to $B_2$ passes through $B_1$ first. 
	"*You cannot get to $B_2$ without going through $B_1$*"

#### Postdomination
A block $B_2$ **postdominates** a block $B_1$ if every path from $B_1$ to the leaf nodes passes through $B_2$. 
	"*Once in $B_1$, must eventually go through $B_2$*"

#### Replacing dead operations
- **Non-Jumps**
	A dead operation that is not a jump can simply be removed
- **Jumps**
	Replace with a jump to the operation's closest **postdominator.**

##### Worked Example
**Example 1:**
	$x := \text{call getint}$ 
	$y := 5$
	$i := 0$ 
$loop: \text{if y <= 0 goto end}$
	$x := x + x$ 
	$y := y – 1$ 
	$i := i + 1$ 
	$goto \ loop$ 
$end: \text{param x}$
	 $\text{call putint}$


Start with the roots, which are always useful. In our case, those are the following instructions:
	$x := \text{call getint}$ 
	$goto \ loop$ 
	$\text{call putint}$
We then propagate backwards from this; we usually do this by performing from a worklist of the roots. While the worklist is not empty:
	Take a marked operation $op$
	If $op: x := y \ op \ z$ 
		Then mark the operations that define $y$ and $z$ as useful, and add them to the worklist if not already marked. This is the **data dependence rule**.
	Mark the conditional branches $br$ in **postdominating** blocks of this block as useful, and add to worklist.
	If a conditional branch is marked useful, then the definition of the operands that appear in its comparison are marked as useful, and added to the worklist.
![](Pasted%20image%2020251216235228.png)
![](Pasted%20image%2020251216235233.png)


##### A mechanically worked example with postdominating blocks with control flow and replaced jumps
TODO.

#### Useless control flow
##### Folding a redundant branch
> If a block ends in a branch, but both the true and false targets are the same, replace the branch with an unconditional jump to the closest **postdominator.**

![](Pasted%20image%2020251217023755.png)

##### Combining blocks
> If a block has only one predecessor, then can be merged with its predecessor.

![](Pasted%20image%2020251217023801.png)


##### Removing an Empty Block
> If a block contains just a jump that forwards control to another block, then it can be merged into its predecessor (or vice versa). We simply merge if the purpose of a block is to provide indirection.

![](Pasted%20image%2020251217023806.png)


##### Hoisting a branch
> If a block contains no computation and only control logic, merge that block with another, typically predecessor and alter the sucessor nodes of the predecessor to match.


![](Pasted%20image%2020251217023824.png)
##### Example
![](Pasted%20image%2020251217023901.png)



