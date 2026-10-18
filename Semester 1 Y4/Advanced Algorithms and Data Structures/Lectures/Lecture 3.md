### Augmentation
- **Augmentation** is modification of a data structure in order to offer additional functionality, or make certain operations more efficient
	- This is an addition not required by the ADT. 
- Consider a dynamic set implemented as a binary search tree. What if we want to support a sum of all keys operation?
	1. Traverse the tree and compute the sum $\to$ $\Theta(n)$ 
	2. Keep a sum variable at the top level/root, updated on insert/delete, on query simply return $sum \to \Theta(1)$ 
#### Augmentation - Minimum in BST
- The **minimum key** in a BST can be found at the bottom left
- Returning this costs $O(h)$ - traverse links down one path = height
- But using augmentation, we can make this more efficient:
	- Keep a $min$ pointer to **minimum**, return on query $\to \Theta(1)$ 
	- $INSERT(k): if \ k < min$, update $min \to \Theta(1)$ 
	- $DELETE(k): if \ k = min$, update $min$ to successor $\to O(h)$
		- Has to be successor(next val strictly greater), no other value that is less than it.
		- This is $O(1)$ where $min$ does not have a right subtree, as its immediate ancestor spawned a left-subtree
		- However, if $min$ has a right sub-tree, its successor is the leftmost node of said right sub-tree.
			![](Pasted%20image%2020261017191949.png)
 - This of course, works for maximum, and is merely reversed on all fronts.
 - This modification is so common that BSTs are generally said to have $O(1) min/max$ 

#### Augmentation examples
- **Linked lists**
	- **Tail pointer**
		- Enables $O(1)$ append
	- **Doubly Linked List**
		- Previous pointer enables $O(1)$, though is still $O(n)$ to traverse the list to find the node if searching by value
		- If directly given the node it is $O(1)$, compared to $O(n)$ in singly linked-list as you do not have to search from the head in order to find the prev node whose next pointer you must reassign to curr's next pointer.
- **Hash table**
- **Graph**

![](Pasted%20image%2020261017192144.png)



### BST Height
- The **height** of a BST is crucial to all of its operations. 
- All of its operations are bounded by $O(h)$ 
- On average, $h$ is $O(log n)$ as if the binary tree as on average each comparison will half the problem space $n$
	- Specifically, the height is minimum for that $n$ when the tree is **complete** - all levels are filled except the lowest level, which is left-filled.
- In the worst case, it can be up to $O(n)$ 
	![](Pasted%20image%2020261017223535.png)


#### Lemma for Number of Nodes in a Tree
- We ask: What are the number of nodes in a ***complete*** tree of degree $d$?
	- Degree of a tree is the maximum degree a node has within the tree.
	- The following example uses $d = 3$ 
- $n = 1$
- $n = 1+3$
	![](Pasted%20image%2020261017223915.png)
- $n = 1 + 3 + 3^2 ...$ 
	![](Pasted%20image%2020261017223942.png)
- $n = \sum_{i=0}^{h-1}d^i$ 
	- Degree/branching factor to the power of the level summed, starting from 0 to account from root node.
		- We already know that $b^j$ OR $d^i$ gives the number of nodes at level $i$ / $j$
	- We can write this summation as:
		$S=1+d^1+... +d^{h-1}$
		$dS = d + d^2 + ... +d^h$
		$dS - S = d^h -1$ 
		$dS-1S = d^h - 1$(so we can factor $S$ out)
		$S(d-1) = d^h - 1$
		$S = \frac{d^h-1}{d-1}$ 
- Setting $n$ to 2 gives us a unique result in that the bottom half of the fraction resolves to $1$ giving us $2^h-1 = n$, where $n$ is the number of nodes in the tree.


#### Tree height
- Thus according to the above lemma a ***complete*** BST of height $h$ has $2^h-1$ nodes.
- A $BST$ of $n$ nodes then, has at least $log$ $n$ height
	- $n = 2^h-1$
	- $n+1 = 2^h$ 
	- $log_2(n+1) = h$ 
- We want to get as close as possible to this.
- A tree is called **balanced** if its absolute difference between the left and right subtrees for every single node is at most $1$
	- Each node has auxiliary info $d$ (or at least, on paper it does) which determines the height difference of its left and right subtree
		- If this exceeds one for any node, then the tree is **not balanced.**
- A balanced tree strives to reduce $h$; we modify the tree-shape on insertion/deletion to maintain **some invariant**

### Balanced Trees
- There exist a lot of different variants of that maintain an invariant to ensure the tree-shape is balanced throughout operations that mutate the tree.
	- AVL trees, red-black trees, 2-3 trees, B trees etc.
- All have different strategies, the differences are:
	- **Ease of implementation**
	- **Practical performance**
	- **Historical usage**

### 2-3 Tree
- A **2-3 Tree** is a **ternary tree**(can have up to 3 child-nodes) where nodes are either:
	- **2-nodes**: contain 1 key and 2 children
	- **3-nodes**: contain 2 keys and 3 children
	- **Terminal**: have no children and one or two data elements.
![](Pasted%20image%2020261017230521.png)


#### Defining property of 2-3 Tree
- **All terminal nodes are at the same level**: the tree is thus **balanced**
	- Note that it may not be complete as this would require all nodes to have the same amount of children i.e., all nodes either by 3 nodes or 2 nodes.
- **All keys are in sorted order**
	- Keys in 3-nodes are sorted
	- Keys in left-subtree are smaller
	- Keys in right-subtree are larger
	- For 3-nodes, keys in the middle subtree are between the two keys. 
	![](Pasted%20image%2020261017230753.png)
	$x < y$
	$\forall a \in A: \ a < x$ 
	$\forall b \in B: \ x \lt b \lt y$
	$forall c \in C: \ c \gt y$ 
- We naturally have the same sorting order imposed on a BST for a 2-node, where keys $\lt$ than key $k$ are in the left-subtree, and vice-versa for the right subtree.
- Also naturally, the child of a 3 node may be another 3 node, or a 2 node.

### Why 2-3 Tree properties establish $h = O(log n)$ 
- Relatively easy to see:
	- Maximum:
		- The maximum amount of keys in a tree of height $h$ would mean all nodes are 3-nodes, so each node has 2 keys, perfect binary tree.
		- $n^{max} = 2 \times \frac{3^h-1}{3-1} = 3^h-1$ 
	- Minimum:
		- The minimum amount of keys in a 2-3 key would require that all nodes are 2 nodes, i.e., have 1 key and 2 children, and thus devolve to a regular BST
		- $n^{min} = \frac{2^h-1}{2-1} = 2^h-1$
- $n = 2^h-1$ can be rearranged to $h = log_2(n+1)$, but this equality naturally only holds for the minimum. 
- We need a more robust way of expressing the tree height for this structure with varying nodes.
- We say that $2^h-1 \leq n \leq  3^h-1$:
	- $2^h-1 \leq n$ 
	- $2^h \leq n+1$
	- $h \leq log_2(n+1)$ 
		WE say here that given $n$ keys the tree cannot grow taller than a certain height e.g., 12 keys, h is at most 3.7, so at most 3.
		If every node is a 2-node then we reach the maximum possible height for that number of keys when we reach equality.
		If some nodes can be 3-nodes, the tree may accommodate more keys without increasing its height.
	- We also have the opposite, saying the tree must have a certain height given $n$ keys, but that  is not important here.
- Worst case height is that we have all 2 nodes, best case is all 3 nodes.

### 2-3 Trees: Operations

#### Searching
- Searching is very similar to BST:
	- 2-nodes behave the same way as in a BST, LR comparison
	- For 3-nodes, we also need to consider that the searched key may be between $x$ and $y$, recurse to middle child
		- Also must check both keys
		- So there is 2 extra checks done on a 3-node
- Cost: in both cases, one level traversed each step, so complexity is $O(h)$ but naturally, the structure preserves that $h$ is roughly $log n$.
	![](Pasted%20image%2020261018020751.png)

#### Insertion
- Again, similar to a BST but we must restore the invariant i.e., protect the balance of the tree. 
- We add the key in the **correct terminal node**
	- If the node was a 2-node, make it a 3-node and we're done.
		![](Pasted%20image%2020261018021224.png)
	- If the node was a 3-node, temporarily make it a 4-node
		![](Pasted%20image%2020261018021901.png)
##### Insertion: 3 node - Splitting
- 3-node terminal nodes that we temporarily turn into 4 nodes must be split
- There are 2 cases:
	1. Parent is a 2 node
		- We move the middle key of the 4 node to the parent node, making the parent node a 3-node.
		- We also move the element that maintains the 'in-between relationship' to be the middle child of the new 3 node we have created
			![](Pasted%20image%2020261018022647.png)
			For the right-subtree, we would move the middle key up regardless to get $py$ but instead placing $x$ as the middle child because $z$ exceeds $y$ and therefore doesn't preserve the relative ordering rule.
	2. Parent is a 3 node:
		- Move the middle key of the 4-node to the parent, making it a 4 node. 
		- Continue this action until we reach a 2 node, which in the worst case is the root.
			![](Pasted%20image%2020261018023418.png)
			Would be the same for right subtree; promote y to get $pqy$ and then make $x$ the middle child of $qy$ as we know it is less than $y$ but must be greater than $q$.
		- This case is recursive unlike case 1, so case 1 becomes our base case.
		- The only way the height of the overall tree can increase is case 1 being applied to the root (which is always a 2 node)

We ask the following questions:
1. Is the invariant conserved?
	- Yes, there are only 2 nodes and 3 nodes with this strategy with the insertion ensuring the sorting order is respected, and all terminal nodes remain at the same level. 
2. What is the cost? 
	- In the worst case, insertion requires us to apply case 2 up until the root and is thus $O(h)$ and because we are preserving the invariant $O(logn)$ 

#### Lazy Deletion
- First idea is a lazy approach
	- Search for the key and mark it with a special tag ("tombstone") signifying a key has been deleted here
	- Such keys are ignored in future searches
	- When inserting, can replace tombstone with key to be inserted if it fits
- Pros:
	- Does not require modifying the structure, which is cheap
	- Future insertions may be cheaper/easier to compute
	- Very easy to implement, just augment the search with some replacement code
- Cons:
	- Does not reduce the size of the tree, if you have a lot of deletions too, you can speed up searches perhaps(technically) too as you can avoid the tombstone checks.

#### Laziness
- The notion of Laziness is deferring the completion of some operation or computing of a result until it is absolutely necessary
- This a strategy employed in certain data structures for amortised cost guarantees, and better performance in some cases
- Lazy deletion in **2-3 trees** is *sometimes* efficient in practice, but theoretically breaks our complexity bounds.
#### Eager Deletion


##### Bubble-up


### Last words on 2-3 Trees

### Tree rotations


### Red-black trees
