## Dead Code 
Refers to instructions that do not affect the result of a program. Divided into **2** categories: 
	1. **Useless code** -  is evaluated/executed, but doesn't change the result of the program. 
	2. **Unreachable code** - is code that would potentially affect the result, but exists in a branch that can be **proved** to never run. 

Useless code + increase code size, memory footprint, cache pressure+worsen locality. Code may become unreachable/useless as a consequence of transformations performed by an optimising compiler. 

Detection of dead code, takes form of control flow analysis. 


### Dead code elimination strategy
Two-phase 'mark and sweep' algorithm:
1. **Mark useful operations**
	Start from operations that must be preserved, and work backwards.
2. **Sweep**
	Remove or replace everything that is non-useful i.e., wasn't marked.

#### Useful Operations
 - The following operations are always useful 
	 - **Performing I/O**
	 - **Unconditional branches**
	 - **Setting procedure return value**
	 - **Modifying storage location that may be accessed outside the local scope**
- For an operation $x := y \ op \ z$ 
	- The operations responsible for defining $y$ and $z$ are useful
	- If a block $b$ contains a useful operation, then any conditional branch in a block $c$ that **postdominates** $b$ is useful(removing that branch may prevent execution of useful code; determines which continuation of execution occurs after $b$)

#### Domination
- A block $B_1$ is said to **strictly dominate** a block $B_2$ if it is impossible to reach $B_2$ without passing through $B_1$ first. That is, every path from the start block to $B_2$ passes through $B_1$ first
	"You cannot get to $B_2$ without going through $B_1$"
#### Postdomination
- A block $B_2$ postdominates a block $B_1$ if every path from $B_1$ to the leaf nodes passes through $B_2$ 
	"Once in $B_1$, must eventually pass through $B_2$"

#### Replacing dead operations 
- **Non-Jumps**
	- A dead operation that is not a jump can simply be removed
- **Jumps**
	- Replace with a jump to the operation's closest **postdominator**.


##### Example:
Start with the roots, which are always useful. May be useful to draw as a CFG. 
We propagate backwards from the roots, we usually do this by performing from a worklist of the roots. While the worklist is non-empty:
	1. Take a marked operation $op$
	2. If $op: x := y \ op \ z$ 
		Mark the operations that define $y$ and $z$ as useful, and add those operations to the worklist if not already marked.
	3. Mark the conditional branches in **postdominating blocks** of the the current block as useful, and add to worklist. 

![](Pasted%20image%2020260118164630.png)
start worklist with:
param x
call putint

then: 
x * x

then:
x := call getint

then:
if y <= 0 goto end (conditional in post dominating; falls through)

then:
y := 5
y := y - 1
goto loop (unconditional)


##### Question
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
  goto B4

B4:
  e = d + 1
  f = c * 3
  goto B5

B5:
  return e


return e, e = d + 1, goto b5, d = c * 2, goto b4, c = a +b, a = 1, b = 2, 

#### Useless control flow
After removing some useless operations, we may be able to eliminate some control flow. 
- If a block ends in a branch, but target is always the same, make jump unconditional. **FOLDING A REDUNDANT BRANCH**.
- If a block only contains a jump, merge the block with its predecessor. **REMOVING AN EMPTY BLOCK**
- If a block ends with a jump to another block, and that block **only has one predecessor,** merge it with this block. **COMBINING BLOCK**.
- If B1 jumps to B2, and B2 is empty but ends with a branch, replace the jump in B1 with a copy of the branch/conditional from B2. HOISTING A BRANCH.




