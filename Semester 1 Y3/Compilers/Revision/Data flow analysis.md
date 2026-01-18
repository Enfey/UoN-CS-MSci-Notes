> Family of static program analysis techniques. Compute information about how values + variable definitions propagate program by analysing $CFG$ paths. Can determine properties such as **reachability**, **liveness**, and **available expressions**, enabling safe and effective optimisation. 


## Liveness analysis
A data-flow property of variables, answers the question:
	*"Is the value of this variable needed?"*

At each instruction, each variable in the program is either live or dead. 

A variable is **live** if it holds a value that may be needed in the future (or its value will be read before the next time the variable is written to).

If a variable is **not live**, is value is no longer needed, can be safely discarded. 

Backward data-flow problem, flow from successor -> predecessor.
## Upward-exposed
> A variable is **upward-exposed** in a basic block if it is used in before it is defined (potentially in that block). 

An upward-exposed variable must have been defined in a previous block, or it is being used uninitialised. 

The upward-exposed variables of a block $n$ are represented $UEVars(n)$ 


$UEVars(n)$ = variables used in block $n$ before any assignment to them. 

### Killed variables
> A variable is **killed** in a block if it is **defined/assigned** a new value in said block.

So called, because if the variable was live before this point, not anymore!

Killed variables in a block $n$ represented as $VarKill(n)$

### Examples
$B:$
	$x = x+1$
	$x = 5$
$x \in UEVars(B)$
$x \in VarKill(B)$

### LiveOut Sets
> $LiveOut(n)$ is the set of variables that are live at the exit of a basic block $n$

$$LiveOut(n) = \bigcup\limits_{m\in succ(n)} UEVars(m) \cup(LiveOut(m) \backslash VarKill(m) )$$
Ian may write set diff as line over the thing to difference. 

Thus, the variables live at the exit of a basic block $n$ are the variables needed at the entry of any successor block $m$. 
- "If m uses a variable before defining it, it's live in $n$, $UEVars(m)$"
- "If m's $LiveOut$ includes a variable that isn't redefined/killed in $m$, its live in $n$"

#### Computing Live Outputs
Computing for all blocks, iterative process. 
1. Compute $UEVars(n)$ and $VarKill(n)$ for each basic block $n$
2. $\forall n \ , \ LiveOut(n) = \emptyset$ 
3. $\forall n$
	1. Compute $LiveOut(n)$
	2. Record whether the definition changed
4. If any block's $LiveOut$ set changed, return to step 3.
5. Iterate to fix point
Iterative, CFG can contain cycles. 


##### Example
	$x := \text{call getint}$
	$y := 5$
$loop: \text{if y <= 0 goto end}$
	$x := x + x$ 
	$y := y – 1$ 
	$goto \ loop$ 
$end: \text{param x}$
	 $\text{call putint}$

Becomes:

B1:
	x:= call getint
	y:= 5
B2:
	$loop: \text{if y <= 0 goto end}$
	$x := x + x$ 
	$y := y – 1$ 
	$goto \ loop$ 
B3:
	$end: \text{param x}$
	 $\text{call putint}$

B1 UEVars = $\emptyset$
B1 VarKill = $\{x, y\}$

B2 UEVars = $\{y, x\}$
B2 VarKill = $\{x, y\}$

B3 UEVars = $\{x\}$
B3 VarKill = $\emptyset$


START BACKWARD, BACKWARD DATA FLOW PROBLEM.

B3 LiveOut = $\emptyset$, no successors. 

B2 Succ = $\{B2, B3\}$
B2 LiveOut = $\{x\} \cup (\emptyset \backslash \emptyset) \bigcup \{x, y\} \cup(\emptyset \backslash \{x,y\})$, $\to$ $\{x, y\}$, changed from emptyset, so recompute
B2 LiveOut = $\{x\} \cup (\emptyset \backslash \emptyset) \bigcup \{x, y\} \cup(\{x,y\} \backslash \{x,y\})$ 
Converges, can now do B1


B1 Succ = \{B2\}
B1 LiveOut = $\{y, x\} \cup (\{x,y\} \backslash \ {x, y})$, $\to \{x,y\}$
Would not recompute, as though changed, its successor has not changed, and is last in chain. 



### Uses and limitations of liveness analysis
#### Finding uninitialised variables
If a variable was **live** on entry but never assigned a value, it may be used uninitialised.

If create a synthetic predecessor block $B_{init}$, preceding all entry blocks in the $CFG$, we can work out which variables are used uninitialised.
- Create $B_{init}$
- Set $succ(B_{init})$ equal to set of entry points in the $CFG$
- Compute $LiveOut(B_{init})$ 
- If $LiveOut(B_{init}) \neq \emptyset$, live outputs are uninitialised before use in the $CFG$.
$B_{init}$ represents all variables needed immediately at program start by definition.
#### Eliminating store operations
If a variable is dead after after a store, i.e., it will never be read again, the store operation is unnecessary, can be eliminated. 

- If the store is the last instruction in the block $n$, then for a variable $x$, if $x \notin LiveOut(n)$, then can be safely eliminated. 
- Cannot remove mid-level stores(unless computing instruction-level liveness)


### Limitations
- Liveness analysis propagates info along $CFG$ edges, thus, if block is unreachable, its effect on liveness is incorrectly included if not handled
	- Can just reorganise the order of optimisations. 
	E.g., block may kill a variable, but if block never executes, the variable is never killed. 

## Related techniques
### Available expressions
> An expression is **available** at a block $n$ if it has already been computed along **every path to that point**, and its operands haven't been assigned to/changed since the last time the expression was evaluated. 

Determines that expression result is the same, and is available to use. 

$AvailIn(n)$ = the set of expressions available at entry of block $n$

$AvailIn(n) = \bigcap\limits_{m \in preds(n)} DEExpr(m) \cup (AvailIn(m)\backslash ExprKill(m))$ 

Apply forwards. Sees practical use in **common subexpression elimination** - reusing previously computed value of **available expression** rather than re-evaluating.
### Reachable definitions
- A definition is considered reachable from a particular program point if:
	1. The concerned variable was assigned a value earlier
	2. There has been no other assignment to that variable along the way
Can statically determine which definitions reach  a given program point. Forward flowing.


E.g., 
1: x := 10    ← d1
2: x := 20    ← d2
3: y := x

D2 reaches line 3, d1 reaches line 2 and is then killed. 


1: x := 10    ← definition d1
2: y := x

d1 reaches line 2


1: if (...) goto L1
2: x := 10    ← d1
3: goto L2
L1:
4: x := 20    ← d2
L2:
5: y := x

d1 reaches 5, and d2 also reaches 5, called a may analysis

