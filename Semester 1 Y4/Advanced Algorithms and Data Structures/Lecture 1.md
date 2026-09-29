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
 - **Substitution Method**
	 - 
 - **Recursion Tree Method**
 - **Master Method**