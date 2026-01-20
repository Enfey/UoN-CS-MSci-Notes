><mark style="background: #FFF3A3A6;"> Family</mark> of <mark style="background: #FFF3A3A6;">static program analysis techniques.</mark> <mark style="background: #FFF3A3A6;">Compute information</mark> about <mark style="background: #FFF3A3A6;">how values</mark> + <mark style="background: #FFF3A3A6;">variable definitions</mark> <mark style="background: #FFF3A3A6;">propagate program</mark> by <mark style="background: #FFF3A3A6;">analysing </mark>$CFG$ paths. Can <mark style="background: #FFF3A3A6;">determine properties</mark> such as **reachability**, **liveness**, and **available expressions**, enabling <mark style="background: #FFF3A3A6;">safe and effective optimisation. </mark>


## Liveness analysis
A <mark style="background: #FFF3A3A6;">data-flow property of variables</mark>, answers the question:
	*"Is the value of this variable needed?"*

At <mark style="background: #FFF3A3A6;">each instruction</mark>, <mark style="background: #FFF3A3A6;">each variable </mark>in the program is <mark style="background: #FFF3A3A6;">either live</mark> or <mark style="background: #FFF3A3A6;">dead.</mark> 

A variable is **live** if it <mark style="background: #FFF3A3A6;">holds a value</mark> that <mark style="background: #FFF3A3A6;">may be needed</mark> in the<mark style="background: #FFF3A3A6;"> future </mark>(<mark style="background: #FFF3A3A6;">or</mark> its <mark style="background: #FFF3A3A6;">value</mark> will be <mark style="background: #FFF3A3A6;">read</mark> <mark style="background: #FFF3A3A6;">before</mark> the <mark style="background: #FFF3A3A6;">next time</mark> the <mark style="background: #FFF3A3A6;">variable</mark> is <mark style="background: #FFF3A3A6;">written to</mark>).

If a <mark style="background: #FFF3A3A6;">variable</mark> is **not live**, is <mark style="background: #FFF3A3A6;">value is no longer needed,</mark> can be s<mark style="background: #FFF3A3A6;">afely discarded. </mark>

<mark style="background: #FFF3A3A6;">Backward data-flow problem</mark>, flow from successor -> predecessor.
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
<mark style="background: #FFF3A3A6;">Computing for all blocks, iterative process. </mark>
1. Compute $UEVars(n)$ and $VarKill(n)$ for each basic block $n$
2. $\forall n \ , \ LiveOut(n) = \emptyset$ 
3. $\forall n$
	1. <mark style="background: #FFF3A3A6;">Compute </mark>$LiveOut(n)$
	2. <mark style="background: #FFF3A3A6;">Record whether the definition changed</mark>
4. If any block's $LiveOut$ set changed, <mark style="background: #FFF3A3A6;">return to step 3.</mark>
5. <mark style="background: #FFF3A3A6;">Iterate to fix point</mark>
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
<mark style="background: #FFF3A3A6;">Would not recompute, as though changed, its successor has not changed, and is last in chain. </mark>



### Uses and limitations of liveness analysis
#### Finding uninitialised variables
<mark style="background: #FFF3A3A6;">If </mark>a <mark style="background: #FFF3A3A6;">variable</mark> was **live** on entry but<mark style="background: #FFF3A3A6;"> never assigned a value, it may be used uninitialised.</mark>

If create a <mark style="background: #FFF3A3A6;">synthetic predecessor</mark> block $B_{init}$, <mark style="background: #FFF3A3A6;">preceding all entry blocks </mark>in the $CFG$, we can work out which variables are used uninitialised.
- Create $B_{init}$
- Set $succ(B_{init})$ <mark style="background: #FFF3A3A6;">equal to set of entry points</mark> in the $CFG$
- Compute $LiveOut(B_{init})$ 
- If $LiveOut(B_{init}) \neq \emptyset$, <mark style="background: #FFF3A3A6;">live outputs</mark> are<mark style="background: #FFF3A3A6;"> uninitialised before use</mark> in the $CFG$.
$B_{init}$ <mark style="background: #FFF3A3A6;">represents all variables needed immediately at program start by definition.</mark>
#### Eliminating store operations
<mark style="background: #FFF3A3A6;">If a variable is dead after</mark> after a store, i.e., <mark style="background: #FFF3A3A6;">it will never be read again, </mark>the<mark style="background: #FFF3A3A6;"> store operation is unnecessary</mark>, can be eliminated. 

- <mark style="background: #FFF3A3A6;">If the store is the last instruction in the block</mark> $n$, t<mark style="background: #FFF3A3A6;">hen for a variable</mark> $x$, if $x \notin LiveOut(n)$, then <mark style="background: #FFF3A3A6;">can be safely eliminated. </mark>
-<mark style="background: #FFF3A3A6;"> Cannot remove mid-level stores</mark>(unless computing instruction-level liveness)


### Limitations
<mark style="background: #FFF3A3A6;">- Liveness analysis</mark> <mark style="background: #FFF3A3A6;">propagates info</mark> along $CFG$ edges, thus, <mark style="background: #FFF3A3A6;">if block is unreachable</mark>, its <mark style="background: #FFF3A3A6;">effect on liveness</mark> is<mark style="background: #FFF3A3A6;"> incorrectly included</mark> if not handled
	- <mark style="background: #FFF3A3A6;">Can just reorganise the order of optimisations.</mark> 
	E.g., block may kill a variable, but if block never executes, the variable is never killed. 

## Related techniques
### Available expressions
> <mark style="background: #FFF3A3A6;">An expression</mark> is **available** at a block $n$ if it has <mark style="background: #FFF3A3A6;">already been </mark>computed along <mark style="background: #FFF3A3A6;">**every path to that point**,</mark> and its <mark style="background: #FFF3A3A6;">operands haven't been assigned to</mark><mark style="background: #FFF3A3A6;">/changed since the last time the expression was evaluated.</mark> 

Determines that expression result is the same, and is available to use. 

$AvailIn(n)$ = <mark style="background: #FFF3A3A6;">the set of expressions available at entry of block</mark> $n$

$AvailIn(n) = \bigcap\limits_{m \in preds(n)} DEExpr(m) \cup (AvailIn(m)\backslash ExprKill(m))$ 

<mark style="background: #FFF3A3A6;">Apply forwards</mark>. Sees <mark style="background: #FFF3A3A6;">practical use</mark> in **c<mark style="background: #FFF3A3A6;">ommon subexpression elimination</mark>** - reusing previously computed value of **available expression** rather than re-evaluating.
### Reachable definitions
- <mark style="background: #FFF3A3A6;">A definition</mark> is considered <mark style="background: #FFF3A3A6;">reachable</mark> from a <mark style="background: #FFF3A3A6;">particular program point if:</mark>
	1. <mark style="background: #FFF3A3A6;">The concerned variable</mark> was <mark style="background: #FFF3A3A6;">assigned a value earlier</mark>
	2. <mark style="background: #FFF3A3A6;">There has been no other assignment to that variable along the way</mark>
<mark style="background: #FFF3A3A6;">Can statically determine which definitions reach  a given program point. Forward flowing.
</mark>

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

d1 reaches 5, and d2 also reaches 5, called a <mark style="background: #FFF3A3A6;">may analysis</mark>

