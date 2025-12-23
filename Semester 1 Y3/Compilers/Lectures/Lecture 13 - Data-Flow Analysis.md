# Data-Flow Analysis
> Data-flow analysis is a family of static program analysis techniques that compute information about how values and variable definitions propagate through a program by analysing paths in its $CFG$. By modelling how information flows along control flow edges and at joins, the compiler can determine properties such as reachability, liveness, and available expressions, enabling safe use and effective optimisations by analysing the $CFG$ and its various points. 

## Liveness analysis
Liveness is a data-flow property of variables answering the question: 'Is the value of this variable needed?'. 

At each instruction, each variable in the program is either live or dead; a variable is **live** at some point if it holds a value that may be needed in future, or equivalently, if its value may be read before the next time the variable is written to. If a variable is **not live**, its value is no longer needed and can be safely discarded.

Liveness analysis is a backward data-flow problem, meaning information flows from successors to predecessors. 
### Upward-Exposed Variables
> A variable is **upward-exposed** in a **basic block** if it is **used before being defined within that block**.

The variable must have been defined in a previous block in scope, or it is being used uninitialised. 

The upward-exposed variables of a block $n$ are represented as $UEVars(n)$. 

$$UEVars(n)={\text{variables used in block n before any assignment to them}}$$
### Killed variables
> A variable is killed in a block if it is **defined/assigned a new value** in that block.

So-called because if the variable was live before this point, it isn't alive anymore. 

Killed variables in a block $n$ represented as $VarKill(n)$ 

$$VarKill(n) = \text{variables defined/assigned a value in block n}$$
#### Killed and Upward-Exposed
$B:$
	$x = x+1$
	$x = 5$
$x \in UEVars(B)$
$x \in VarKill(B)$
### LiveOut Sets
> $LiveOut(n)$ is the set of variables that are **live at the exit** of a basic block $n$.

$$LiveOut(n) = \bigcup\limits_{m\in succ(n)} UEVars(m) \cup(LiveOut(m) \backslash VarKill(m) )$$
The set $succ(n)$ is the set of basic blocks that immediately follow after $n$.

Thus the variables that are live at the exit of a basic block $n$ are all the variables that are needed at the entry of any successor block $m$. 
- **"If m uses a variable before defining it, it's live in n ($UEvar(m)$)".**
- **"If m's $LiveOut$ includes a variable that isn't redefined in m, it's live in n".**
#### Computing Live Outputs
Computing the $LiveOut$ sets for all blocks is an interative process. 
1. Compute $UEVars(n)$ and $VarKill(n)$ for each basic block $n$
2. $\forall n \ , \ LiveOut(n) = \emptyset$ 
3. $\forall n$
	1. Compute $LiveOut (n)$
	2. Record whether the definition changed
4. If **any block's** $LiveOut$ set changed, return to step 3.
5. Iterate to fixed point.
Must be iterative because a CFG can contain cycles.
#### Example
	$x := \text{call getint}$
	$y := 5$
$loop: \text{if y <= 0 goto end}$
	$x := x + x$ 
	$y := y – 1$ 
	$goto \ loop$ 
$end: \text{param x}$
	 $\text{call putint}$

Thus $B_1$ = 
	$x := \text{call getint}$
	$y := 5$
$B_2$ = 
	$loop: \text{if y <= 0 goto end}$
	  $x := x + x$ 
	  $y := y – 1$ 
$B_3$ = 
	$end: \text{param x}$
	  $\text{call putint}$

| Block | $UEVars$    | $VarKill$   |
| ----- | ----------- | ----------- |
| B1    | $\emptyset$ | $\{x, y\}$  |
| B2    | $\{x, y\}$  | $\{x,y\}$   |
| B3    | $\{x\}$     | $\emptyset$ |

Initially, all $LiveOut$ sets are empty:

| Block | $LiveOut$   |
| ----- | ----------- |
| B1    | $\emptyset$ |
| B2    | $\emptyset$ |
| B3    | $\emptyset$ |
Liveness analysis is a backward data-flow problem, meaning information flows from successors to predecessors. This means we start with $B_3$.

$B_3:$ 
$LiveOut(B_3) = \emptyset$ as $B_3$ has no successors.

$B_2:$
$succ(B_2) = \{B_2, B_3\}$ $LiveOut(B_2) = UEVars(B_2) \cup (LiveOut(B_2) \backslash VarKill(B_2)) \cup UEVars(B_3) \cup (LiveOut(B_3) \backslash VarKill(B_3))$$LiveOut(B_2) = \{x, y\} \cup (\emptyset \backslash \{x, y\}) \cup \{x\} \cup (\emptyset \backslash \emptyset)$ - as the $LiveOut$ set changes $(\emptyset \to \{x, y\})$ we go back to step 3, and calculate it again
$LiveOut(B_2) = \{x, y\} \cup (\{x, y\} \backslash \{x, y\}) \cup \{x\} \cup (\emptyset \backslash \emptyset)$ - now that it has converged, we compute $B_1$ 

$B_1 :$ 
$succ(B_1) = \{ B_{2} \}$
$LiveOut(B_1) = UEVars(B_2) \cup (LiveOut(B_2) \backslash VarKill(B_2))$ 
$LiveOut(B_1) = \{x, y\} \cup (\{x, y\} \backslash \{x, y\})$ 
$B_1$ only needs to be computed once since $B_2$ has converged, and no further iteration is required for it. If we went forwards we'd have to backtrack and recalculate $B_1$ again. 

Only just realised I should've split B2 as it has 2 exits, but I cannot be arsed redoing the calculation lol, you get the gist.
### Uses and limitations of liveness analysis
#### Finding uninitialised variables
>If a variable is **live on entry** but it was never assigned a value, it may be used uninitialised. 

If we create an imaginary block $B_{init}$ that precedes all entry blocks in the $CFG$ we can work out which variables are used uninitialised:
- Create new block $B_{init}$ with no content
- Set $succ(b_{init})$ equal to the set of entry points in the $CFG$
- Compute $LiveOut(b_init)$
If $LiveOut(B_{init}) \neq \emptyset$ the live outputs are uninitialised before use in the $CFG$. $LiveOut(B_{init})$ represents all variables needed immediately at program start, so if any such variable has no assignment before its first use, it's uninitialised. 

#### Eliminating Store Operations
> If a variable is dead after a store, i.e., it will never be read again, storing it is unnecessary and the instruction can be eliminated.

If the store is the last instruction in the block n, then for a variable $x$, if $x \notin LiveOut(n)$ then it can be safely eliminated. Though if the store appears mid block, and we aren't computing instruction level liveness, we cannot safely eliminate the store as it may be reused within that block.

#### Limitations
##### Assumes that all basic blocks can be reached.
> Liveness analysis propagates information along all CFG edges, thus if a block is unreachable, its effect on liveness is incorrectly included if not handled. 

A block may kill a variable, but if that block can never be executed, the variable never actually gets killed!


## Related Techniques
### Available expressions
> An expression is available at a program point if it has already been computed along every path to that point, and its operands have not been redefined since the last computation.

This means the expression is safe to use without re-evaluation. 

Let $AvailIn(n)$ = the set of expressions available at entry of block $n$ 

$AvailIn(n) =\bigcap\limits_{m \in preds(n)} DEExpr(m) \cup (AvailIn(m) \backslash ExprKill(m))$

This applied forwards, unlike liveness analysis, and is also relatively straightforward to compute. It sees practical use in **common subexpression elimination** - reusing the previously computed value of an **available expression** rather than re-evaluating. 
	![](Pasted%20image%2020251216214343.png)


### Reachable definitions
> A definition is **reachable** at a program point if 
> 1. The variable was assigned a value earlier
> 2. There has been no other assignment to that variable along that way. 

We can statically determine which definitions can reach a given program point. The analysis, much like available expression analysis, is forward-flowing. 

The equation is very similar to other data flow equations. See [here](https://en.wikipedia.org/wiki/Reaching_definition#:~:text=reach%20d3.-,As%20analysis,-%5Bedit%5D) for more.