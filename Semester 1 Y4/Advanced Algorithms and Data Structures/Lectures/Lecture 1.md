## Algorithm Complexity
- Algorithmic complexity is concerned with the **measure** of resources, most often **time** (computation speed) and **space** that an algorithm requires to run. 
- We can perform two things to measure this complexity with regard to **time**:
	- **Measuring** how long a particular implementation takes to execute
		- Gives an exact measurement in milliseconds for a particular execution
		- Specific to input and size, and is not implementation/machine agnostic.
	- **Analysing** the algorithm to understand its complexity (classification of execution time as a function of its input $n$)
		- Gives a 'loose' bound the algorithm subscribes to.
		- This is machine, implementation, and input agnostic and so the result generalises beyond the local result the former would give, at the cost of losing precision in time measurement. 
### Types of Complexity:
- **Worst-case time complexity**
	- The maximum possible time/executions taken/performed over any possible input $n$
	- Useful for guaranteed upper bound by taking the most punishing input possible and is usually the simplest and most useful measurement to characterise algorithms by. 
- **Average-case time complexity**
	- The expected time over a typical input. 
	- Useful when the worst case is rare/not expected to occur in a practical setting. 
	- Algorithms often differ in average and worst-time complexity.
- **Amortised Complexity**
	- Average cost per operation over a *sequence* of distinct operations
	- Useful when expensive operations are rare, but necessary e.g., dynamic array resizing and copying, append time complexity remains $O(1)$ even under the occasional copy+resize as the cost is amortised.
- **Space complexity**
	- How much memory an algorithm requires, typically bounded by time complexity as you cannot touch more memory cells than you have time steps but is a meaningful concern it its own right.

### Running times
- We define $T(n)$ as the time the algorithm takes to run on inputs of size $n$ 
	- Different inputs of the same size $n$ can take different times (sorted vs unsorted list)
	- Unless stated otherwise, we generally refer to the worst case for a particular $n$ 
	- We do not mean the *exact running time* but instead take a measure of elementary computational steps that we can generalise.
### Measuring input and time
- **Input size** $n$ 
	- Refers to the number of **bits** the input takes in memory.
- **Running time** 
	- Refers to the number of basic hardware operations executed. 
- However, complexity theory is usually more relaxed than this and is concerned with permitting us to generalise.
	- **Input size**
		- Measured by **memory locations** occupied e.g., list by its length $n$, a tree by its number of nodes $n$, a graph by its vertices $v$ etc.
	- **Running time** 
		- Measured by assuming that certain elementary operations e.g., passing a pointer, performing arithmetic take constant time (which is often false (e.g., operands may be big and not fit in a machine word and may actually compile to multiple instructions)) - but can actually be bounded)
- The *exact* running time is not important, but the **Complexity Class** is. 
	- How the running time changes as a function of $n$ e.g., linear, quadratic, exponential. 
	- Yields a classification that generalises and can be used as a comparative result. 


## Big-O Notation
- The notation $f(n) = O(g(n))$ means that:
	- The function $f(n)$ grows **at most** as $g(n)$ 
- There is a constant $c$ and a threshold $n$ such that: $\forall n \geq n_{0}$, $0 \leq f(n) \leq cg(n)$ 
	- Scale $g$ by any fixed factor $c$ to prove/disprove the equivalence/faster growth with regard to $g$ 
	- After a certain size $n_0$ this holds; we are generally concerned with the complexity as $n$ tends toward infinity (long-term performance).
		-  Allows to ignore weird/inefficient behaviour of an algorithm for small inputs e.g., startup overhead. 
- Big $O$ $f(n) = O(g(n))$, there is a constant $c$ and a number $n_0$ such that $\forall n \geq n_0 , 0 \leq f(n) \leq cg(n)$ 
- Big $\Omega$ $f(n) = \Omega (g(n))$, there is a constant $c$ and a number $n_0$ such that $\forall n \geq n_0 , 0 \leq cg(n) \leq f(n)$ 
- Big $\Theta$ $f(n) = \Theta(g(n))$ there are constants $c_1, c_2$ and $n_0$ such that $\forall n \geq n_0, 0 \leq c_1g(n) \leq f(n) \leq c_2g(n)$ 

### Little o notation (strict)
- **Little-o**
	- $f(n) ∈ o(g(n)) \iff \lim_{ n \to \infty } \frac{f(n)}{g(n)} = 0$
		- Does not permit equivalence
		- Defined using limits instead to enforce strictness as $g$ can be multiplied by $c$ to appear strictly greater than $f(n)$ even though they're say, both linear. 
- **Little-**$\omega$ 
	- $f(n) \in \omega(g(n)) \iff \lim_{ n \to \infty } \frac{f(n)}{g(n)} = \infty$
		- Strictly runs faster as $n$ tends to infinity - also does not permit equivalence. 
- **Asymptotic equivalence**
	- $f(n) \sim g(n) \iff \lim_{ n \to \infty } \frac{f(n)}{g(n)} = 1$ 
- Example:$$f(n) = n^2 + n + 4, \quad  f(n) \in O(n^2), \quad f(n) \notin o(n^2), \quad f(n) \sim n^2$$
	$o(n)$ requires sub-linear.


### Notation
- $O(g)$ is a set of functions/complexity class defined by $g$ so it is proper to write $f(n) \in O(g(n))$ 
- However it is more common to write $f(n) = O(g(n))$ or $f(n)$ is $O(g(n))$ even if the functions are not actually asymptotically equivalent. 
- Equivalences:
	- $f(n) = O(g(n))$ and $f(n) = \Omega(g(n)) \iff f(n) = \Theta(g(n))$ 
	- $f(n) = O(g(n)) \iff g(n) = \Omega(f(n))$ ($g(n)$) at least as fast as $f(n)$
		- Same for little-o notation
	- $f(n) = \Theta(g(n)) \iff g(n) = \Theta(f(n))$ 
		- Symmetric.
	- $f(n) = o(g(n)) \implies f(n) = O(g(n))$


### Growing functions
- Complexity analysis focuses on functions that grow with $n$
- The exceptions involve multiple variables and trade-offs:
	- Graph algorithms have two size parameters. In sparse graphs(number of edges much lower than the potential) $\vert E \vert \approx \vert V \vert$, but in dense graphs $\vert E \vert \approx \vert V \vert ^2$ (more vertices are connected)


### Hierarchy of Complexity Classes
- There is an intuitive pecking order demonstrated by the rate of function growth: $1 ≪ log \ n ≪ nᶜ ≪ exp \ n ≪ n!$  
	- Constant, logarithmic, polynomial, exponential, factorial
- $log \ n$ is smaller than **any** polynomial, including powers smaller than 1, $log \ n ≪ n^{\epsilon}, \forall  \epsilon > 0$ 
- The base does not matter and is usually omitted. 
- Exp $n$ is larger than any polynomial - polynomial of infinite degree as the power is not constant. 
- The base **does** matter for exponential complexity and needs to be specified e.g., $2^n \neq \Theta(3^n)$ 
	- This is because in order to make exponentials equivalent for the $\Theta$ equation the constant would need to predicated on $n$; no single $c$ can reduce/increase the ratio between the functions for given $n$ because the constant is applied $n$ times to itself. 

### Composition
- Let $f \in \Theta(F)$ and $g \in \Theta(G)$ 
	- $fg \in \Theta(FG)$ 
	- $f+g \in \Theta(F+G) = \Theta(max(F, G))$ 


![](Pasted%20image%2020260928232031.png)



### Master Theorem
- Many problems are solved using **recursion** rather than iterative looping.
- In particular, they are solved using an algorithmic paradigm known as **divide-and-conquer**
	- Recursively split input into partitions
	- Perform some computation on these partitions and produce further partitions
	- Repeat until some base case is triggered and then aggregate results by unwinding stack and passing back values. 
- For example merge sort:
	- Split list in two halves
	- Recursively sort them
	- Merge the two lists
	- Singleton list considered sorted.
	![](Pasted%20image%2020260928232503.png)

### Recursive Equations for Time Complexity
- Let us determine the Time Complexity $T(n)$ of this algorithm
- A singleton list $n=1$ will return in constant time: $T(1) = c_0$ 
- For longer lists, the algorithm will perform several steps:
	- Determining the middle of the list takes constant time: $d$ 
	- Splitting the list into two halves takes linear time: $c_1n$ 
	- Auxiliary function $merge$ also takes linear time: $c_2n$ 
	- Finally, the two recursive calls take time $T(n/2)$ because they each take half the size of the list of size $n$ 
- We are left with $T(n) = 2T(n/2) + cn + d$ (where $c = c_1 + c_2)$ 

#### Simplifying the equations
- If length $n$ is not even, then the splitting is not even so we are left with sublists where $n$ differs so to be precise:
	- $T(n) = T([n/2]) + T([n/2]) + cn + d$ 
	- This is a common and safe approximation but naturally does not change the growth rate since the difference will be at most $1$.
- Exact value of constant factors are not relevant, we are concerned with the larger component that dominates so for $cn+d$ we just need to consider that it is **linear** and we can rewrite the equation with this in mind:
	- $T(1) = \Theta(1)$
	- $T(n) = 2T(n/2) + \Theta(n)$ 

### Solving Recursive Equations
- Three methods to solve a recursive equation that expresses algorithm runtime recursively in terms of $n$:
	- **Substitution Method**
		- Proof technique based on induction
		- We make a guess on how quickly the recurrence grows, and substitute that in the original recurrence to determine whether it is mathematically consistent
		- May suspect merge sort has a runtime of $O(n \ log\  n)$ 
			- This method allows us to demonstrate that this is indeed an upper bound. 
	- **Recursion Tree Method**
		- Instead of guessing the complexity and proving it, we expand the recurrence into a tree that represents all the recursive function calls made during execution.
		- Observe how total work accumulates as the recursion progresss.
		- The idea is that the total runtime is the sum of the work performed at every node in the recursion tree, where a node is an invocation of the recursive function. 
	- **Master Method**
		- General mathematical result/theorem that allows us to determine the asymptotic complexity of certain recurrence relations without explicitly expanding the recursion tree or constructing an inductive proof.
		- Depends on the form of the equation however. 

#### Substitution Method
- Say we start with $T(n) = 2T(n/2) + n$ 
- Suppose we believe that $T(n) = O(n \ log_2 \ n)$ 
- By definition of the $O$ notation there exists positive constants $c$ and $n_0$ such that $T(n) \leq c \ n \ log_2 n$ for every $n \geq n_0$ 
- We are claiming that beyond some sufficiently large input size, the running time is always bounded by a constant multiple of $n \ log \ n$ 
- To verify this:
	- $T(n) = 2T(n/2) + n \leq c \ \frac{n}{2} log_2 \frac{n}{2} + n$ 
	- $T(n) \leq 2(c \ \frac{n}{2} \log_2 \frac{n}{2}) + n$
	- $T(n) \leq cn \ log_2 \frac{n}{2}+n$
	- $T(n) \leq cn \ (log_2 n-1) + n$ 
	- $T(n) \leq cn \ log_2 n - cn + n$ 
	- $T(n) \leq cn \ log_2n + (-c+1)n$ 
- If we choose $c = 1$ then the last term evaluates to $0$ 
	- $T(n) \leq cn \ log_2n$

- The base case is more problematic
- We have $T(1) = 1$, we can't prove $T(1) \leq c1 \ log \ 1 = 0$ 
- We can choose any starting point $n_0$ however:
	- For example:
		- $T(2) = 2T(1) + 2 = 4 \leq c2 log_2 2 = 2c$, pick $c \geq 2$ 


Work more on the inductive proofs.

#### Recursion Trees
- We construct a tree of recursive calls, labelled with arguments.
- The root is $T(n)$. For divide and conquer e.g., merge sort, we have 2 calls e.g., 2 children with arguments $n/2$ 
	![](Pasted%20image%2020261016004938.png)
- The number of nodes at level $j$ is equal to $2^j$, for perfect binary tree.
	- $b^j$ 
- The argument/size of argument at level $j$ is $n/2^j$ 
- The total work at level $j$ is therefore $2^j \times n / 2^j = n$ (the $2^j$ factors cancel)
- As such, there are total of $n$ computation steps at each level.
- The height of the tree:
	- At level $j$ the problem size is $n / 2^j$.
	- If the tree has height $k$, the leaf nodes satisfy $n / 2^k = 1$ 
	- $n = 2^k$
	- $log_2 n = k$
- As such, there are $n$ computation steps per level, with $log_2n$ levels, leaving us with $n \ log \ n$ 
#### Master Method
- Method to determine the time complexity of recursive algorithms without having to expand and solver their recurrence relation manually
- The master theorem applies to recurrence relations of the form $T(n) = aT(n/b) + f(n)$ 
- The number of children for each node of a recursion tree will be $a$, the **branching factor/arity**
- The number of arguments at level $j$ = $n / b^j$ 
- The height of the tree $h = log_b n$ (via the above)
	- Because $b^h = n$ so invert the square with logarithm of $b$.
- The intuition is then:
	*Which levels of the call tree contribute the most work?*
	1. **Bottom Heavy**
		Most of the work takes place at the leaves. Each level does not progressively work more than the previous one
		$a^{log_bn} = n^{log_ba}$ 
	2. **Balanced**
		Every level contributes roughly the same amount of work: number of levels $log_bn$ multiplied by work at each level, which will roughly be $f(n)$ by $b^j = a, \times  n/a = n$
	3. **Top-Heavy**
		Most of the work happens near the root. Each level does progressively less work. Dominated by first few levels $\to f(n)$ 
	- Total work per level:
	- 1 node at level 0 with work $f(n)$ 
	- $a$ nodes at level 1 with work $af(n/b)$ 
	- $a^2$ nodes at level 2, with work $a^2f(n/b^2)$ 
	- $a^j$ nodes at level $j$ with work $a^jf(n/b^j)$ 
	- $a^h$ nodes at level $h$ with work $a^hf(n/b^h) = O(a^h)$, because $f(n/b^h)$ = $f(1)$ = $O(1)$
	- We can use log laws to rewrite $a^h = a^{log_bn}$ 
	![](Pasted%20image%2020261016201734.png)
	Learn these at a later point.
##### Master method: Cases
- Let $c = log_ba$, that is, the 'critical exponent'
- Let $d$ describe the growth of non-recursive work in terms of $n^d$ 
- We have three cases based on $c$ and $f(n)$ 
	1. $f(n) = O(n^{c-\epsilon}) \to T(n) = n^c$ 
	2. $f(n) = \Theta(n^c log^k \ n) (inc \ k=0)$ $\to T(n) = n^c log^{k+1} \ n$ 
	3. $f(n) = \Omega(n^c+\epsilon) \to T(n) = f(n)$
- The total work at level $j$ is $W_j = a^jf(n/b^j)^d$ 
	- $W_j = a^j (n^d/b^{jd})$ 
	- $W_j = a^j(n^d/(b^d)j)$ 
	- $W_j = n^d(a^j/(b^d)j)$ 
	- $W_j = n^d(a/b^d)j$ 

##### Merge Sort
- We have $T(n) = 2T(n/2) + n$ 
- $a = 2, b = 2, f(n) = n$ 
- Calculate $c = log_b(a) = log_2(2) =1$ 
- Therefore $n^1 = n$ leaves, equivalent to $f(n)$ done at the root, and is thus not case $1$ because $f(n)$ grows at an equivalent rate $n^c$, the work done at the leaves.
- Case 2, choose $k=0$, $n^clog^0n = n^1 \times 1 = n$ 
- We know that $f(n) = n$, so case 2 applies as we are tighltly bounded, sub in $c = 1$ and $k = 0$ and get $\Theta(n \ log \ n)$ 