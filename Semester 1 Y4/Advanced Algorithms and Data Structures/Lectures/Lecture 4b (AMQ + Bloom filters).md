## Approximate Membership Query
- An **Approximate Membership Query** (AMQ) stores a set $S$  without storing the elements themselves.
	- It is a lossy representation of $S$
	- It is an example of **approximation** in a data structure.
- The focus of this structure is **space-efficiency**
- The information/space reserved is enough to perform one operation: $search(x)$, which answers whether of not $x \in S$ 
- The cost of a space-efficient representation of $S$ is that the answers to $search(x)$ are not always correct:
	- **There are no false negatives**, if $search(x)$ returns $false$ then we say that $x \notin S$ guaranteed.
		- Equivalently if $x \in S$ we always say yes.
	- **There are false positives**, with probability $\epsilon$
	- We may say yes to $search(x)$ when actually $x \notin S$

| $search(x)$ | $x \in S$ | $x \notin S$ |
| ----------- | --------- | ------------ |
| $true$      | always    | $\epsilon$   |
| $false$     | never     | $1-\epsilon$ |
- WE have no false negatives: $search(x)$ will never return false where $x \in S$ (but may return true when $x \notin S$, which is our false positive.)
### Rationale
- We can use AMQ's as a **proxy** to govern accesses to some other data structure, typically one whose accesses are slow
	1. Perform the lookup in the AMQ first, who is usually small enough to fit in fast memory.
	2. If it says definitely not, we know the item is not in
	3. If it says yes, the item may be in our secondary data structure, so perform the full access (false positive may be possible)
- Time is thus only wasted for false positives, which is what we seek to minimise in AMQ implementations
- The space and computational efficiency of this approach is massive:
	- A bloom filter needs around 10 bits per element for a 1% false-positive rate, **irrespective of how large the underlying elements are**
- Typical applications include:
	- Distributed systems
	- Expensive/remote memory and having to fetch said memory
	- Networking(routing, caching etc)

### Sketches
- AMQ's and Bloom filters are an example of a **sketch**
- A **sketch** is a compact data structure that 'summarises' a large dataset and can answer certain questions about the data without storing/processing the data directly.
- AMQ's are one kind of a *sketch* that test set membership.
- Some further examples are:
	- **Count-min sketch**
		- Estimates how frequently items occur; it can overestimate but never underestimate.
	- **HyperLogLog**
		- Estimates the number of distinct elements using only a few kilobytes.
	- **MinHash**
		- Estimates similarity between large sets/documents
	- **Space-Saving sketch**
		- Tracks most frequent items.

## Bloom Filter - Structure and Insertion
- A bloom filter is a concrete implementation of an $AMQ$
- A bloom filter is an array of $m$ bits, all initialised to $0$
- There are $k$ different hash functions $(h_1, h_2, ..., h_k)$ that each turn a key $x$ into a position in the array.
- We define insertion like so:
	- $insert(x) = h_1(k) = A_1, h_2(k) = A_2, ..., h_k(x) = A_k$ 
	- Example:
		- $insert(3)$
			- $h_1(x) = 2$ 
				![](Pasted%20image%2020261104000649.png)
			- $h_2(x) = 6$
				![](Pasted%20image%2020261104000734.png)
			- $h_3(x) = 4$
				![](Pasted%20image%2020261104000753.png)
	- Example(cont.)
		- $insert(y)$
			- $h_1(y) = 7$
				![](Pasted%20image%2020261104000828.png)
			- $h_2(y) = 3$ 
				![](Pasted%20image%2020261104000846.png)
			- $h_3(y) = 4$
				![](Pasted%20image%2020261104000914.png)
		- Note that we have a collision at position $A[4]$; this is where we get our false positives from, which we will see below. It is okay to have collisions; for each element to have its own array bits would significantly increase the space requirements of a bloom filter, so we take care choosing our hash functions. 
		- We just leave the bit as $1$ under collision. 

### Bloom Filters - Search
- If $h_i(x) = 1$ for all $1 \leq i \leq k$ (as in, the number of hash functions idk why $k$ isn't explicitly bound here) then we can conclude that $x \in S$
	- Superficially of course, with probability $\epsilon$ that this is a false positive.
- If there exists at least one $i$ such that $h_i(x) = 0$ we conclude that $x \notin S$
- Either $A[h_1..k]$ is all $1$'s thus $x \in S$, or has at least one $0$ and thus $x \notin S$ 
- Example:
	- $search(z)$ 
		- $h_1(z) = 4$ 
			![](Pasted%20image%2020261104002625.png)
		- $h_2(z) = 3$ 
			![](Pasted%20image%2020261104002658.png)
		- $h_3(z) = 5$
			![](Pasted%20image%2020261104002704.png)

### Bloom Filters - False Positives and Negatives
 - It is entirely possible to conclude $x \in S$ even though $x \notin S$ 
	 - This is because some other element $y$, or a combination of elements $y, z ...$, hashed to those same positions that $x$ would've hashed to, so when checking we naively assert that $x \in S$ when that is not necessarily the case. **False positive**.
	 - We cannot conclude that $x \notin S$ if $x \in S$ because $A[h_i..k(x)]$ would hold 1's if $x \in S$.
- Example:
	- $search(z)$ 
		- $h_1(z) = 6$
			![](Pasted%20image%2020261104003048.png)
		- $h_2(z) = 4$
			![](Pasted%20image%2020261104003102.png)
		- $h_3(z) = 3$ 
			![](Pasted%20image%2020261104003115.png)
		- We assert that $z \in S$ (though really in this scenario we know that $z \notin S$ thus false positive)

### Bloom Filters - False Positives Probability
- Let us assume that $S = \{x_1, \dots , x_n\}$ 
- A false positive happens when $y \notin S$ and yet $\forall 1 \leq i \leq k$, we have $h_i(y) = h_j(x_a)$ for at least one $1 \leq j \leq k$ and at least one $1 \leq a \leq n$ 
	The hashes are equivalent, and this may be caused by more than 1 element (second bound)
- Assuming the input is uniformly distributed(for the hashes) and $h_i$ is *independent* (each behaves as a pseudorandom choice)
	- We have the following
		- $Pr(h_i(y) = h_j(x_a)) = 1 / N$ 
		- $Pr(\exists : h_i(y) = h_j(x_a)) = 1 - Pr(\forall j: h_i(y) \neq h_j(x_a))$ 
			^Take complement of LHS $Pr$
			=$1-(1-1/N)^k$ 
			- This comes from the probability of **collision** which we established first, so we take the complement of that, to get the probability of **no collision**
			- We raise this to the $k$ because there are $k$ hash functions. 
		- $Pr(\exists a, \exists j : h_i(y) = h_j(x_a)) = 1 - Pr(\forall a, \forall j : h_i(y) \neq h_j(x_a))$ 
			$= 1 - (1-1/N)^{nk}$ 
			Same reasoning as above, except we now bound $a$ and there are $n$ $x$'s.
		- $Pr(\forall i, \exists a, \exists j : h_i(y) = h_j(x_a))$
			- Exists for all hashes $i..k$ of $y$ some hash function $j$ over some $x_a$ that is equal !
			$= (1-(1-1/N)^{kn})^k  = \epsilon$ 
- We have this $\epsilon$ predicated on $n$ in the form of $a$ and predicated on the number of hash functions
- This leads us to be able to select an **optimal choice** for the number of hash functions $k$ for a particular choice of $n$, as our false positive rate can be directly derived in inputting those values.
### Choosing an optimal number of hash functions
- We have $N$ bits and $n$ number of items to insert, and we can choose $k$ hash functions
- Changing $k$ pulls the false-positive rate in two opposite directions:
	- More hash functions
		- Helps searches, a non-member $y$ has to find at every one of its $k$ positions a 1, in order to trigger a false positive.
		- Also hurts, as every insert now sets $k$ bits, which makes general collisions more likely as the array fills up faster, so it is more likely that future $y$ will encounter repeated collisions. 
- So $k$ too small is bad, and $k$ too large is bad
	- Too small is not resistant to a collision when it occurs, increasing the false positive rate.
- The best value is somewhere in between, and we can determine this precisely.
- ...