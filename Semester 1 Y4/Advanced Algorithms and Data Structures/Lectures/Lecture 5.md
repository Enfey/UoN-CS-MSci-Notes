## Priority Queue ADT
- Sometimes known as called a **Heap** (note that the memory heap is not an instantiation of this ADT)
	- A heap is the most common implementation of the priority queue ADT, so much so they are often used interchangeably.
- This ADT defines a collection of *unique* **keys,** *optionally* paired with **values.**
	- Unique $KV, V$ optional
- We have no interest in searching this structure.
- We want to add elements, and peek at/extract the *minimum*(**min-heap**) or *maximum*(**max-heap**) element efficiently.
- We have the following operations:
	- $empty()$ - instantiates a new priority queue
	- $isEmpty()$ - tests whether the priority queue is empty
	- $insert(k)$ - inserts a new key $k$ 
	- $min()$ - returns the smallest key currently in the queue - peek
	- $extract()$ - *removes* and returns the smallest key

### Intuition and Use Cases
- Picture a physical pile:
	![](Pasted%20image%2020261112205721.png)
- Placing something on top is easy, and taking off the top item is easy. 
- Retrieving a specific item is hard. 
- A better example is hospital triage. 
	- Patients arrive in any order, and the most urgent is always seen next, without any need to search for a specific patient in the ordering. 
- Thus we would queue the patients as they order with respect to their priority, which becomes their keys e.g., severity level.
- Some examples of real world applications of this include:
	- Service industry in general e.g., table management.
	- Dijkstra(and other graph algorithms) - always expand the closest unvisited vertex next, and this is maintained in a priority queue.
		- Many algorithms use a priority queue to explore the most promising path.
	- Live scheduling algorithms
		- CPU task scheduling specifically uses priority queues to decrease latency in execution, minimising the latency from when a process first had execution to when it last has execution, rather than in arrival order specifically.
	- Packet forwarding
		- Routers use priority queuing for quality of service, forwarding high-priority data packets e.g., unordered streaming packets or VoIP, ahead of torrenting or standard file downloads.

### Extra Operations
- We typically define a few extra abstract operations:
	- $merge(PQ_A, PQ_B)$ 
		- Combines two compatible priority queues into one e.g., the separate scheduling lists of a multi-core architecture for more efficient dispatch of jobs.
	- $decreaseKey(pointer, newKey)$
		- Lowers the key of an element already in the priority queue.
		- In a min-heap, this makes the element more urgent, vice-versa for a max-heap/PQ.
		- This is useful in some algorithms/scenarios
			- E.g., we have some cpu job who no longer performs interrupts/IO; more latency is now forgiveable, so we decrease the key (max-heap) to make the job less urgent.
				- Obviously speaking theoretically here.
		- A pointer to the node to change the key of is *required* to make this operation efficient in practice, as there is no efficient random access for the typical heap implementation, which would make this operation $Θ(n)$

### Naive Realisations
- Some naive instantiations of a PQ could be:
	- **Unordered List**
	- **Ordered List**
	- **Balanced Binary Search Tree**
#### Unordered list PQ
- We would have
	- $empty()$ - $[ \ \ ] \to \Theta (1)$  
	- $isEmpty()$ - test if the list is $[\ \ ] \to \Theta(1)$ 
	- $insert(k)$ - append $k \to \Theta(1)$
	- $min(k)$ - search list for minimum key $\to \Theta(1)$
	- $extract(k)$ - find minimum, repair link (deletion) $\to \Theta(1)$ 
- This is essentially a lazy approach, where no order according to the priorities/keys is preserved on insertion, and we defer the actual semantics of priorities to $min$ and $extract$ which gives $\Theta(1)$ for what are the most common operations.
#### Ordered list PQ
- In this case:
	- The **minimum** is now the the first/head of the list: $min(k)$ and $extract(k)$ become $\Theta(1)$ 
	- However, insert must now put the new element in the right place, walking through the list, $\Theta(n)$ 
		- In an array we can find the position in $O(log \ n)$, but shifting the elements along is still $\Theta(n)$
		- In a linked list we cannot binary search, so its $\Theta(n)$ anyway, but has more storage constraints.
- This is the eager approach to ordering, where we maintain structure by keeping the $min$ element(s) at the head(assuming we have duplicate keys)
	- The lecture says assume we do not, but in practice we always have duplicates keys.
- The cost then is now placed on initial insertion, as it must maintain this structure to ensure that $min$ and $extract$ are $\Theta(1)$

#### Balanced binary search tree PQ
- Insert, minimum, and extract can be done in $O(log \ n)$
	- Keeps the full sorted order of every element at all times so the complexity is merely the tree height, which is $log \ n$ where the tree is balanced.
- However, since we're not interested in searching the PQ(ideally), we'd like to see if we could do better than this, especially for $min$ and $extract$
	- Though we could always of course, augment the BST to keep a pointer to the $min$ element
	- Insertions still remain $\Theta(log \ n)$ as they sit in the far left subtree, $d = h$ 
- The tree is overkill considering we don't need traversals, successors/predecessors. 
	- The ADT is relatively simple.
- Can we do better on **min/extract**?

### Using heaps to sort lists
- We can envision two operations:
	1. $listHeap$ 
		- Insert every element into the list into a heap.
			![](Pasted%20image%2020261112223139.png)
	2. $heapList$
		- Call $extract()$ until the list is empty, $append()$ each result to a new list.
			![](Pasted%20image%2020261112223155.png)
- Combining these two operations effectively sorts a list $L$.
- In practice heapsort is usually done a bit differently, slightly more efficient and in-place without creating 2 separate data structures that retain the same information.
	- But this works and has optimal complexity.
- The sorting algorithm you get though, depends on the priority queue implementation.
	- Of course one can use heaps, but say you use an unordered list, each insertion will be $\Theta(1)$ whereas with an ordered list it would be $\Theta(n)$
		- The extractions are technically not needed when we use an ordered list, preservation of the order has been done by $insert(k)$; $listHeap$ is the identity, much like $heapList$ is the identity for unordered lists.
	- So the underlying implementation of the PQ affects this algorithms real-world performance.
- The cost is $n$ inserts + $n$ extracts (append is dominated):
	![](Pasted%20image%2020261112223717.png)

## Binary Heaps
### Tree Completeness
- There are several definitions pertaining to a tree being complete.
	1. **Perfect** trees
		- All internal nodes have 2 (or whatever the arity is) children, and all leaf nodes are at the same depth $d$
	2. **Complete** trees
		- Every tree level is full, except possibly the last level, which is filled from the left.
- Every *perfect* tree is also a *complete* tree
- For both perfect and complete trees, we have $h = \Theta(log \ n)$ 
- We require that heaps are **complete** when represented as trees. They need not be perfect.

### The Heap Property
- **Heap property**: the key of a given node is $\geq$ it's parent's key
- This is for a min-heap; for a max-heap the key of a node is $\leq$ its parent's key.
- A valid min-heap can be seen below:
	![](Pasted%20image%2020261112225405.png)
- A valid-max heap can be seen below:
	![](Pasted%20image%2020261112225411.png)
- A heap which violates the heap property can be seen below:
	![](Pasted%20image%2020261112225436.png)
	- And is in fact a perfect binary search tree.
- An incomplete tree can be seen below:
	![](Pasted%20image%2020261112225515.png)
- Thus a binary heap preserves **both** *completeness* and *the heap property*.
	- The heap property is deliberately simple; this has the following consequences:
		- **Partial ordering**- two siblings can be in either order providing they exceed or equal the parent. Because of this, searching for a key is $\Theta(n)$ as a comparison does not halve the search space.
		- **Deeper does not mean bigger** - in the valid min-heap, $5$ is a grandchild of the root, yet it is smaller than 7. The heap property is still preserved because $5$ exceeds $4$.
- A heap is sorted **top to bottom**, whereas a BST is sorted **left to right.**


### Implementation as an array
- **Binary Heaps** can be implemented as arrays, with parent-child links made implicit by indexing, instead of maintaining pointers.
- We write the nodes out level by level, left to right
	- Because the tree is complete, there are no gaps, so $n$ keys fill $A[0..n-1]$ exactly.
- The tree links become arithmetical(assuming we index from 0):$$A[k] \leq A[2k+1]$$
$$A[k] \leq A[2k+2]$$
- The children of index $k$ are $2k+1$ and $2k+2$
- The parent of index $k$ is $[(k-1)/2]$ 
- The reason we have $2k+1$ is because the $k$ nodes before index $k$ each have 2 children, filling indices $1$ to $2k$, so $k's$ children come right after them at $2k+1$ and $2k+2$ 
	- Note that $k$ is the index.
- The use of an array fits very well:
	- The next free slot in the tree is merely $A[n]$, easily known to us at all times, and is necessary for *insertion*
		- Placing a key here allows the tree to preserve its shape automatically(due to how we index)(providing we preserve the order/heap property).
	- The last node in the tree is $A[n-1]$, easily known to us at all times, and is necessary for *extraction*.

### Implicit Relationships and Data Structures
- There are different definitions:
	- **Explicit**
		- Relationships between elements are encoded directly.
		- E.g., each node has pointer to children.
	- **Implicit**
		- Relationships between elements are encoded indirectly via *some rule*
		- E.g., element ordering, index arithmetic
			- Thus our binary heap implementation is **implicit**
		- Usually this implies they use $O(1)$ extra memory - does not grow (e.g., don't need to add more rules as we run the algo live) (beyond storing the $n$ data) 
			- This is important as if a data structure has implicit relationships. If something is implicit, it is also both succinct and compact.
			- Something with explicit relationships naturally does not use $O(1)$ extra memory and is either succinct or compact.
	- **Succinct**
		- A **succinct** data structure uses $o(M)$ extra memory, with $M$ = "informational-theoretical minimum memory to store the data, while still allowing for efficient operations".
			- Thus the overhead grows strictly slower than this bound, and its share of $M$ tends toward $0$ as $M$ grows.
	- **Compact**
		- A **compact** data structures uses $O(M)$ extra memory
			- The overhead's share stays at some fixed proportion and does not shrink, always wasting the same fraction. 
- For example:
	- **Flat OA Hashmap** (explicit and compact)
		- Keeps everything in singular array is slot, when taken, probes via some probing sequence/scheme, other slots
		- There are no separate nodes or pointers
		- But has to track load factor to avoid long probing sequences, typically 75%
		- The overhead then, stays fixed, as whatever the remaining portion of the load factor is gets filled
		- While it doesn't have a relationship that involves extra memory, the fact this structure is **compact** means it cannot be implicit (as implicit also has consequences for memory overhead).

### Insertion and Extraction
- Both of these operations follow the same pattern when applied to a heap: 
	- Keep the **shape complete first** *then* **repair the ordering** (along a single path through the tree that we have affected)
- **Insertion**
	- Put the new key in the next free slot
		![](Pasted%20image%2020261112235748.png)
	- In an array representation this is $A[n]$ ; the tree keeps its shape automatically, as the shape comes from the array representation.
	- The heap property can fail between the new key and its parent. 
	- $bubble-up(i)$ compares it with its parent, and swaps it if it is smaller (min-heap), repeating until the parent is smaller than the node we're inserting, or we reach the root.
		![](Pasted%20image%2020261112235755.png)
- **Extraction**
	- The minimum is at $A[0]$ 
	- We cannot just remove it, because that would leave a hole.
	- Instead we move the **last** element into the root, which keeps the shape complete. 
	- However, because it came from the bottom that element breaks the heap property as it will be larger than the immediate children of $root$
		![](Pasted%20image%2020261113000505.png)
	- $sift-down$ swaps the new root with its smaller child, repeating until it is $<=$ than its children, or we swap with a leaf node.
		- Swapping with the larger one in any instance would create a heap property violation, as the parent would exceed the other sibling.
	![](Pasted%20image%2020261113013900.png)

#### Complexity of Insertion
- The worst case is still $\Theta(log \ n)$, as if the newly inserted key is smaller than everything, we will traverse the tree height $2^h$, giving us $\Theta(log \ n)$ 
- The average case however is $\Theta(1)$ 
	- We assume that the keys are random however, and are uniformly distributed.
- The intuition:
	- About half of the nodes in a complete tree are leaves.
		- The sum of a tree shows that the number of nodes in a given level is double the prior level
	- Thus the key to be inserted has a 50% probability to already belonging at the leaf level, in which case it does not move.
	- If it does, the same argument applies at the next level; 50% chance it stops there, as the number of nodes only decreases as you go further up the tree comparing to parents to preserve the heap property.
- $Extract$ is still $\Theta(log \ n)$ average however, because we take a key that was already at the bottom, and is thus likely to be large, and we have to sift it downward which is likely to traverse a large portion of the tree, as the larger nodes are naturally concentrated near the bottom...
- Note that while the array itself is compact, a path through it jumps from index $k$ to $2k+1$
	- For a large heap implementation, consecutive levels sit far apart in memory, so each step can result in cache misses quite easily.


### Heapify
- Transforming an arbitrary array into a min-heap in place is called **heapifying**
- This is quite a common operation:
	![](Pasted%20image%2020261113015803.png)
- Everything from $n/2$ onwards is a leaf, and $n/2-1$ yields the first non-leaf element.
	- Picture this leftward, goes onto the level above from where the node $n$ is situated, and then reads leftward.
- **Why do we do sift-down?** 
	- $sift-down$ on its own swaps an element $i$ with its immediate children, swapping with the smaller child if $A[i]$ is bigger.
	- This of course only works if both of $i$'s subtrees are valid heaps
	- Because the leaves are the base case and thus a valid heap (and have no ordering imposed on them) we start from the level above
	- The problem with the ordering can thus only be with that initial parent, and each call to $sift-down$ ensures that $A[i]$ ends up in the correct position
	- We do this continually for all elements that are not-arrays until we have worked through the entire array.
	- This works because the **heap property is local, not global**

#### Example
![](Pasted%20image%2020261113020831.png)
	Arbitrary Array
![](Pasted%20image%2020261113020909.png)
	$6-1 = 5$ 
	$5/2 = 2.5$, round down to index $2$ thus select 8 and invoke $sift-down$
![](Pasted%20image%2020261113030318.png)
	 $5-1 = 4$, $4/2 = 2$ = element we just did so skip
	 $4-1$ = $3$, $3/2$ = 1, so select $3$ and invoke $sift-down$.
![](Pasted%20image%2020261113030658.png)
	$3-1$ skipped, already dealt with index $1$
	$2-1$ = $1$, $1/2$ = $0.5$ = $0$ thus select the root and invoke $sift-down$
- Remember we get the parent of the final node, and then count backwards from that index as that is when the levels above the leaves start that we actually need to sift down from to preserve the heap property.
#### Complexity
- There are roughly $O(n)$ calls to an operation whose worst case is $O(log \ n)$ 
- This is however an overestimate.
	- Consider that 50% of the tree is the leaves, and thus these are indices that $sift-down$ never works on.
	- Each step of sift-down moves the key down one level, which is a constant amount of work.
	- A sift-down on a node of height $h$ costs at most $h$ steps
		![](Pasted%20image%2020261113033134.png)
		- The root node here has height $h$ and thus can take at most $h$ steps
	- There are around $n/2$ calls, and if each cost $log \ n$ steps you'd get $O(n \ log \ n)$ 
	- However, only the root truly has $O(log \ n)$ worst case, and most nodes are concentrated near the bottom and thus can only move a step or two.
	![](Pasted%20image%2020261113033332.png)
		The naive bound would be 7 calls x 3 (h) = 21
		11 is notably less than $n$
	- To generalise: $$T = 1 \frac{n}{4} + 2 \frac{n}{8} + 3 \frac{n}{16}+ \dots$$
	- Each level has half as many nodes as the one below it. With $n/2$ leaves, $n/4$ have height 1 and so on and so forth, and $h$ directly informs the steps $sift-down$ takes.
	- Thus:
		$$=\frac{1}{2} \sum_{i=1}^h i \cdot 2^{-i} \leq n$$
	We have $O(n)$



### Order property in Heaps and BSTs







