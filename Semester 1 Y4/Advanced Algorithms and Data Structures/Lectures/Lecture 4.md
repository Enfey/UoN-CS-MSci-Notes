### Dictionary ADT
- Also known as an **associative array** or a **map**
- Represents a collection of $(key, value)$ pairs, where the keys are *unique*
	- $(k_1, v_1), (k_2, v_2), ... (k_n, v_n)$ 
- Has the following operations:
	- $search(k):$ returns value $v$ associated with key $k$
	- $insert(k, v):$ creates an entry for value $v$ associated with key $k$
	- $delete(k) :$ removes the entry indexed by $k$
- We have seen that search trees can implement this ADT and augment it to speed up particular operations in the prior lecture.
- **Hash tables** are an alternative method to implement this ADT.

## Comparing Hash Table and Search Tree Computational Efficiency

### Worst case Complexity 
- $n$ keys

| Operation | (Balanced) Search tree | Hash Table |
| :-------- | :--------------------: | ---------: |
| search    |      $O(log \ n)$      |     $O(n)$ |
| insert    |      $O(log \ n)$      |     $O(n)$ |
| delete    |      $O(log \ n)$      |     $O(n)$ |

### Average Case complexity

| Operation | (Balanced) Search tree | Hash Table |
| :-------- | :--------------------: | ---------: |
| search    |      $O(log \ n)$      |     $O(1)$ |
| insert    |      $O(log \ n)$      |     $O(1)$ |
| delete    |      $O(log \ n)$      |     $O(1)$ |

## Hashing
- A hash function maps an arbitrary key $k$ into an integer within a fixed range.
- It is a function from key space $K$ to range $0...N-1$: $$h : K \to \{0, \dots , N-1\}$$
	Where:
	- $K$ is the key space
	- $N$ is the size of the output range.
	- $h$ is the hash function. 
- For example, suppose our keys are alphanumeric strings, with unlimited length $n$, $K = 36^n$ 
	- Because $36 \times 36 \times 36 \dots$ 
- The hash function is a mapping from this key space to some output space constrained by $N$, perhaps $1000$.
### Properties of a good hash function
- **Fast to Compute**
	- Hashing is intended to make accessing entries efficient.
	- If calculating $h(k)$ takes a long time, we undermine the advantage that implementing a Dictionary ADT with this data structure gives us. 
		- If our constant time hash is slower than an alternative concrete implementation whose efficiency is dependent on $n$, then it may be better to use those implementations, particularly where $n$ is small.
- **Deterministic**
	- The same key $k$ must always produce the same hash.
	- $h(k_1) = h(k_2)$ **iff** $k_1 = k_2$
	- Otherwise, we would not be able to find or delete existing entries in the hash table. 
- **Avalanche effect**
	- Intuitively, small changes to the input should produce substantially different hash outputs. 
	- For example, two similar strings, ideally, should produce very different hash values. 
	- If we have $h$ that consistently only maps to $N / p$ where $p$ is a small portion of the output space, we are likely to get collisions and increase the cost of the operations on the hash table. 
	- We seek to distribute entries as uniformly as possible among our hash table, and thus our output space. 
		- We want it so that if the input is *uniformly distributed* that the output space reflects this
	- By ensuring that we our outputs differ on small changes on the input we can enforce this.
- We do not necessarily choose cryptographic hash functions which maintain properties such as pre-image resistance and collision resistance, as these are often much more expensive to compute
	- It does however, depend on the concrete use case.


## Hash Table
### Hash Table Basics - Creation
- Suppose we create an array $A[0...N-1]$ that is initially empty.
	- We choose $N = 8$ 
		![](Pasted%20image%2020261102162354.png)
- **Sentinels**
	- "Empty" $(\emptyset)$ is usually implemented by a **sentinel**
	- A **sentinel** is a specific value that corresponds to no valid key for that slot/output value from $h(k)$
	- Also known as a flag value, or a dummy element
	- Some examples include:
		- $-1$
		- $NULL$
- Our table may be initialised, and our output space is postitive integers, therefore $-1$ would be a suitable sentinel as it will never be the output of $h(k)$ 
### Hash Table Basics - Insert
- The insertion operation is defined as: $$insert(k,v): A[h(k)] \leftarrow (k, v)$$
- We place $(k,v)$ into a 'bin' of sorts in $A[N-1]$, where the index is governed by $h(k)$ 
- Example:
	- $insert('David', 42)$
	- $insert('Susan', 67)$
	- $h(David) = 2$
	- $h(Susan) = 7$ 
	![](Pasted%20image%2020261102162838.png)
### Hash Table Basics - Search
- The search operation is defined as:$$search(k): A[h(k)]$$
- To search the value corresponding to $k$, we look up bin/index $h(k)$ of $A$
- If the returned value is **sentinel**, then the key $k$ has not yet been inserted into the hash table, and does not exist
	- This should be reflected when returning from such a function
- Example:
	- $search('David')$
	- $search('John')$
	- $h(David) = 2$
	- $h(John) = 3$ 
	![](Pasted%20image%2020261102163131.png)
	There is a sentinel at $k(John)$ and thus this key/a key at that also hashes to that index has not yet been inserted into the hash table.

### Hash Table Basics - Delete
- The deletion operation is defined as: $$insert(k, v) : A[h(k)] \leftarrow \emptyset$$
- We set bin $h(k)$ of $A$ to the sentinel value (we represent generally with $\emptyset$)
- Example:
	- $delete('David')$
	- $h(David) = 2$ 
	![](Pasted%20image%2020261102164215.png)
	We have updated the index that $h(David)$ yields to the sentinel value $-1$ for this implementation. 
### Collisions
- When we have two distinct keys $k_1$ and $k_2$ that yield the same hash, we say that we have a **hash collision**:$$k_{1} \neq k_{2} : h(k_{1}) = h(k_{2})$$
- Resolving collisions generally affects all operations in practice, and the structure of the hash table itself. 
- This is because collisions are unavoidable in practice. 
	- Especially when dealing with large $K$, and finite output space; by the pigeonhole principle, multiple keys must map to the same output. 
- To begin with, we can choose a suitable $h$ with good avalanche characteristics to preserve the distribution property(uniformity where input is uniform), but this can only do so much.
- There are two main families of collision resolution, which augment the hash table itself:
	- **Separate chaining**
	- **Open addressing**

#### Collision Resolution Strategies - Separate Chaining
- This a collision resolution technique where each slot in the table points to a **bucket**
- A **bucket** is a secondary data structure that acts as a container for keys that collide at $A[h(k)]$
	- This secondary data structure is typically a linked list, but can be a search tree, a secondary hash table, etc.
##### Table Creation (linked lists)
- The hash table is initialised as standard, with sentinel values in every bin.
- When we perform an initial insertion at $A[h(k)]$ we instantiate our linked list and set $A[h(k)]$ to contain a pointer to the head of our linked list, and set the head of the linked list to contain $(k, v)$ 
##### Collisions
1. We instantiate a new node containing $(k, v)$
2. We link this new node to the existing chain by updating the $next$ pointer of the new node to the existing head pointed to by $A[h(k)]$
3. Set $A[h(k)]$ to contain the pointer to the new list head.

##### Operations
- $insert(k,v) : A[h(k)].insert(k, v)$ 
- $search(k) : A[h(k)].search(k)$ 
- $delete(k) : A[h(k)].delete(k)$ 
- We hash the key to find the bin as normal
- Then defer to the secondary data structure held in that slot (if any, we may need to instantiate on first insertion) 
- Perform the ordinary operation on that bin's secondary data structure.
- Example to $search(k_2)$ 
	1. Calculate $h(k_2)$ which yields index $2$
	2. Access the linked list associated with bin $2$
	3. Perform the search operation on the linked list associated with bin $2$ (linear search)

##### Analysis
- A natural objection is that linked list searches/traversals(deletion) are $O(n)$ thus may degrade the performance of search and deletion operations
- In the worst case of course, all $k$ keys could hash to the same bin, resulting in $O(n)$ (or $O(k)$) search or deletion
- This assumed a particularly poor distribution of keys, and resultant poor hash function which does not preserve the properties we want.

###### Assume uniform hashing
- We assume that keys are uniformly distributed over key space $K$
- Assuming our hash function maintains the desired properties, this implies a uniform distribution of hash output across $N$ bins
- Therefore, for any arbitrary key, the probability of hashing to any particular bin $i$ is: $$p = \frac{1}{N}$$
- For example, if we have 10 bins, the probability of key landing in bin 3 is: $$ p = \frac{1}{10} = 0.1$$
###### Modelling bin occupancy with a binomial distribution
- We now want to establish **how many keys we can expect to find in a particular bin**
	- Let $X$ represent the number of keys that land in a particular bin
	- We have $n$ keys being hashed, and we want to model the probability of each possible size of occupancy: $$ X \in \{0,1,2, \dots , n\}$$
- A **binomial distribution** counts the number of successes in a fixed number of independent yes-no trials.
- Each hash operation represents one trial:
	- Number of trials: $n$, amount of keys.
	- Success: key $k$ hashes to bin $X$
	- Probability of success: $p = 1/N$
	- Failure: the key hashes to any other other bin, with probability $1-p$ 
- We have a fixed number of independent trials, each with the same probability of successes, so we can model the occupancy of keys $X$ in a bin with a binomial distribution: $$ X \sim \text{Binomial}\left( n, \frac{1}{N} \right)$$
	This describes the **probability** of a particular bin receiving $0..n$ keys from key space $K$; it applies equally to any individual bin under uniform hashing.

###### Expected Value/Mean
- The **expected value** tells us the average number of keys we would expect to find in a particular bin
- In a binomial distribution this is: $$\mu = np$$
- Substituting $p = 1/N$: $$ \mu = \frac{n}{N}$$
- This is the expected occupancy of any particular bin. 
	- This makes sense; we have $N$ bins, and we divide $n$ keys into those bins, uniformly.
- However, every bin will have a different occupancy; this is just the average. 
	- We want to know how much occupancy will vary across the hash table.

###### Variance
- Variance measures how spread out possible bin occupancies are around their expected value.
- For a binomial distribution: $$\sigma^2 = np(1-p)$$
- Thus: $$\sigma^2 = \frac{n}{N}\left( \frac{N-1}{N} \right)$$
	*Substituting N/N for 1*
- Therefore: $$\sigma^2 = \frac{n(N-1)}{N^2}$$
- Since $N$ is typically large, we can approximate $(N-1)/N$ as $1$: $$\sigma^2 \approx \frac{n}{N}$$
- This tells us how much the expected occupancy is expected to float around the mean, which is defined as $\frac{n}{N}$.
- A larger variance indicates a greater spread of possible occupancies among the bins in the hash table.
- We can also calculate the standard deviation of the binomial distribution:
	![](Pasted%20image%2020261102183917.png)
	Don't worry about how we calculate the probability of $n$ amount of keys occupying the bin $i$.
	We calculate the SD by taking the square root of the variation of the mean. 
###### 99.7%
- For sufficiently large $n$, the binomial distribution which represents how many keys we can expect to find in bin $i$, can be approximated using a normal distribution
- For a normal distribution:
	- 68% of outcomes are found within one S.D of the mean
	- 95% of outcomes are found within one S.D. of the mean
	- 99.7% of outcomes are found within one S.D. of the mean
- We can use this to understand the likely occupancy of a particular bin $i$
- Suppose that we have $n = 100$, and $N = 10$, we thus have $\mu = 10$ and $\sigma \approx 3.16$ 
- Therefore:
	- 68% of outcomes fall between 10-3 and 10+3 i.e., 7-13 keys in bin $i$
	- 95% of ooutcomes fall between 10-6 and 10+6...
- **Therefore, 99.7% of bins have $\mu \pm 3 \sigma = \frac{n}{N} \pm 3\sqrt{\frac{n}{N}}$ keys.**  
	- FUCKING LOOK MATE.

###### Implications for time complexity
- The expected number of keys is $\mu = n / N$ 
- If the number of keys grows proportionally with the number of bins, then $n = O(N)$, it grows no faster than a constant multiple of the number of bins: $$n \leq cN$$
- Therefore: $$\frac{n}{N} \leq c$$
- AND FINALLY, WE HAVE: $$\frac{n}{N} = O(1)$$

- It is of course, still possible that keys concentrate in a few bins. 
- This is known as **key clustering**. 
- This is generally due to poor distribution of $n$ or a hash function with poor avalanche characteristics that does not preserve the uniform distribution property. 
- If every key hashes to the same bin $i$, that bin's secondary data structure contains all $n$ keys, thus operations are governed by the worst-case time complexity of the deferred operations on that secondary data structure
	- E.g., arbitrary deletions in linked list are $O(n)$, and so we inherit that.
- If worst-case performance is a concern, we use a different secondary data structure with better worst-case performance such as a balanced search tree.


##### Rehashing
- If $n/N$ becomes too large, as our input space will exceed our output space and will make collisions inevitable.
	- Of course as described prior, this also tells us the expected number of keys $n$ that will appear in some arbitrary bin, as we divide the keys $n$ by the output space $N$.
- The solution to this is to instantiate a new hash table, with greater capacity e.g., double the capacity.
- Elements are then **re-hashed** into this new table. 
- The average cost of rehashing is $\Theta(n)$ as we must rehash every key.
- This is mostly relevant for **Open Addressing**

#### Collision Resolution Strategies - Open Addressing
- Open addressing is a hash table collision resolution method where all data elements are stored directly inside the single dimensional array $A$, rather than in secondary data structures.
	- This has the benefit of no pointers (and perhaps better space efficiency as we do not have wasted entries that simply hold a pointer - each entry is doing something)
	- $n + N$ pointers vs $n$ 
- Collision resolution is done through **probing**:
	- Suppose we want to insert a key $k$
		1. Inspect the bin $h(k)$ 
		2. If the bin is available, store the key there
		3. If the bin is occupied, inspect the next candidate in the **probing sequence**
		4. Repeat steps 2-3 until inserted.
- There are different **probing strategies** we can take, which will yield a probing sequence given initial bin $h(k)$ 
##### Linear probing
- We define the $i-th$ element in the sequence as $h(k) + ic (mod \ N)$ 
	- Typically have $c = 1$, so just traverse bins linearly, where $i$ represents the number of probe attempts made. $$h(k) + i (mod \ N)$$
##### Open Addressing - Insertion - Linear Probing
![](Pasted%20image%2020261102212303.png)
![](Pasted%20image%2020261102212330.png)![](Pasted%20image%2020261102212339.png)

- Simple enough, just mutate the insertion operation to scan linearly with a modulus across the flat array until a bin is found that can hold key $k$

##### Open Addressing - Search
- We define search normally as $search(k) = A[h(k)]$, but we must perform equality checking as per normal.
- If there has been a collision at $A[h(k)]$ then $k$ may still be in the hash table due to the linear probing insertion method
- We use the same linear probing method to scan entries and perform comparison checks
- If we run into a **sentinel** and do not see key $k$, we know that $A$ does not contain key $k$
	![](Pasted%20image%2020261102212837.png)
	![](Pasted%20image%2020261102212842.png)
- However, there is a problem, that we will describe in deletion, that applies for both deletion and searches.

##### Open Addressing - Deletion
- Recall that searching follows the same probing sequence used during insertion for some key $k$
- Now suppose we have a hash table like below:
	![](Pasted%20image%2020261102213404.png)
	- Imagine that David, Anne, and Rob all have $h(k) = 2$ 
	- They are stored at adjacent bins per the probing sequence.
	- Now we delete Anne:
		![](Pasted%20image%2020261102213501.png)
	- When we perform $search(Rob)$, we will encounter a sentinel when probing from $A[2]$ and conclude that $Rob$ is not in the hash table, when he was placed in bin 4.
- An empty bin prematurely terminates a search, which has implications for both search and deletion, as the searched-for-key may exist further along the probing sequence.
	- But may not be discoverable due to prior deletions. 

##### Open Addressing - Eager deletion
- The solution is to rearrange elements after deletion such that we do not leave an empty bin in the middle of a probing sequence
- Given $k$, let $@k$ be the index of the bin that holds $k$; $@k$ belongs to $k's$ probing sequence
	- $@k$ not necessarily equal $h(k)$, collisions may have displaced the key
- **Invariant** - in a cluster, there must be no vacant slot between $h(k)$ and $@k$
	- Otherwise the lookup for $k$ will fail. 
- When we perform a deletion, we probe linearly in that cluster.
- The deleted key's bin becomes the **gap** or vacant slot. 
- We step through the following bins, at each step key $k$ changes:
	- While the bins are occupied, if the gap $g$ is between $h(k)$ and $@k$, the invariant is broken for $k$.
		- Move $k$ into the gap, and $k's$ old bin becomes the new gap.
		- If we don't do this, $k$'s lookup never passes through the gap because it encounters a sentinel.
	- Otherwise, leave alone, as there is no gap $g$ between $h(k)$ and $@k$
![](Pasted%20image%2020261102220144.png)
![](Pasted%20image%2020261102220148.png)
![](Pasted%20image%2020261102220153.png)
![](Pasted%20image%2020261102220200.png)


###### Why must we check whether gap is between $h(k)$ and $@k$ 
- Let $h(a_*) = 2$ and $h(b_*) = 4$, and we're deleting $a_2$ 
	![](Pasted%20image%2020261102220324.png)
- If we naively shifted everything to the left, and then tried to delete say, $b_1$:
	![](Pasted%20image%2020261102220451.png)
- The operation would return as not found. 
- When $a_2$ is deleted we probe linearly for that gap and check that gap $g$ is not sitting between $h(b_1)$ and $@b_1$.
- The gap is not sitting between this as $h(b_*) = 4$ and $@b_1 = 3$ thus must have come from wraparound earlier and so is never accessed by probing from $A[2]$.
- The correct procedure for resolving gap $g$: 
	![](Pasted%20image%2020261102220839.png)
Because $g = A[3]$, $h(a_*) = 2$, and $@a_3 = 6$ 



##### Open Addressing - Lazy Deletion
- This is the alternative to the eager shifting described above.
- When we delete a key $k$ from the hash table, we do not move anything
- We simply mark the bin as a **tombstone** (which is distinct from a sentinel)
	- Lookups treat a tombstone as occupied, and thus do not terminate probing sequences.
		- Thus the gap problem from before never occurs. 
- On insertion, we can *reuse* a tombstone; if an insert, or an insert's probe hits a tombstone, the key $k$ can go there. 
- Deletion is made cheap and easy, and also works for any probing scheme, whereas eager deletion **only works for linear probing**.
- **However**:
	- Lazy deletion never gives back a truly empty bin so occupancy never decreases
	- Over time, the table fills with tombstones.
	- This means that lookups for missing keys that only stop at sentinels may go on longer
	- We also are likely to need more re-hashing, as the occupancy is not decreasing, as we are unlikely to have an equal amount of insertions that map to existing tombstones, and we know that re-hashes are $O(n)$ 

##### Open Addressing - Clustering
- A **cluster** is a sequence of occupied bins in a row. Specifically this is **primary clustering** defined below.
	![](Pasted%20image%2020261102222414.png)
- Clusters tend to snowball:
- The bigger a cluster gets, the more likely it is to grow.
- This makes lookups slow as we degrade to (often linear) probing scans for large portions of the hash table.

###### Clustering Continued
- Clustering is quite common in Open Addressing implementations
- We can use **other probing strategies** to help.
- **Quadratic Probing**
	- The $i-th$ element in the probing sequence is $h(k) + c_1i + c_2i^2 \ (mod \ N)$ 
	e.g., pick $c_1 = 0, c_2 = 1$ 
	'Erratic' jumps make primary clusters less likely.
- **Double Hashing**
	- The $i-th$ element in the sequence is $h(k) + ih'(k) \ (mod \ N)$ 
- *Secondary clustering*:
	- For $x, y : h(k) = h(y)$, we still have the same probing sequence, because the probing sequence (barring double hashing) depends only on $h(k)$'s result.
		- Double hashing is predicated on an entirely different hash function, which should be furthter designed to reduce collisions. 
	- In words: *keys with the **same** home bin follow the exact same probe sequence, because the sequence is dependent only on $h(k)$ (not true for double hashing)*
- *Primary clustering*:
	- Keys with **different** home bins end up sharing the same probe path.
	- Any $h(k)$ that lands in the cluster, when using linear probing will usually join the cluster 
		![](Pasted%20image%2020261102223219.png)


###### Clustering and Cache Locality


##### Probing sequences are Permutations


### Open Addressing vs Separate Chaining