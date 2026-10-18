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
	- Specifically, it is balanced within 0 levels as every single leaf node ends at $h$. if this is not true, then the tree is not a proper 2-3 tree.
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
- First idea is a **lazy** approach
	- Search for the key and mark it with a special tag ("**tombstone**") signifying a key has been deleted here
	- Such keys are ignored in future searches
	- When inserting, can replace tombstone with key to be inserted if it fits
- Pros:
	- Does not require modifying the structure, which is cheap
	- Future insertions may be cheaper/easier to compute
	- Very easy to implement, just augment the search with some replacement code
- Cons:
	- Does not reduce the size of the tree, if you have a lot of deletions too, you can speed up searches perhaps(technically) too as you can avoid the tombstone checks.
		- Thus, this operation does not actually maintain the height property we require for search to stay roughly $log$ $n$, as it relies on amortising the cost over an equivalent amount of insertions.

#### Laziness
- The notion of Laziness is deferring the completion of some operation or computing of a result until it is absolutely necessary
- This a strategy employed in certain data structures for amortised cost guarantees, and better performance in some cases
- Lazy deletion in **2-3 trees** is *sometimes* efficient in practice, but theoretically breaks our complexity bounds.
#### Eager Deletion
- There are a number of ways we can perform *eager* deletion, only cover one option here
- Similar to BST deletion, but just like with insertion, we may break the invariant(s), and we must therefore ensure we preserve it:
- Note that by invariants, we refer not only to the tree height, but also to the node structure, the sorted order, and that the tree is height balanced.
	- IF:
		- **Key in terminal 3-node**: make it a 2-node.
		- **Key in terminal 2-node**: create a temporary placeholder node (we cannot directly delete, as the parent is required to have 2 children), we call this node a ***bubble***
		- **Key in internal node:** like in BST deletion, we find the immediate predecessor (which will be terminal), delete the predecessor from the terminal and replace the key in parent with the predecessor to preserve the ordering property.


##### Bubble-up
- **Case 1:**
	- Parent is a **2-node**, sibling is a **2-node**
	- The solution is to redistribute.
	- We combine the sibling and parent to form a 3-node by moving the parent down into the sibling:
		![](Pasted%20image%2020261018180907.png)
		Naturally this process begins at a terminal node where $A$, $B$ and $C$ will be empty subtrees.
		- This is valid because we know that $A \lt p \lt B \lt s \lt C$, and this redistribution preserves that logic
			- Note this is naturally symmetric.
		- We 'push the bubble-up' the tree until we can find a place where we can eliminate it with the following cases.
	- Only case where bubble moves up; if the bubble becomes root, merely eliminate it ($h$ decreases?)
- **Case 2:**
	- Parent is a 2-node, sibling is a 3-node:
		![](Pasted%20image%2020261018181205.png)'
		- Simply redistribute to respect the order, replacing the bubble by taking out whatever from the 3-node can become the parent 2-node, and moving the original parent to where the parent is.
- **Case 3:**
	- Parent is a 3-node, sibling is a 2-node. 
	- Parent has 3 children:
		![](Pasted%20image%2020261018181327.png)
		- Preserve ordering of $A \lt p \lt B \lt s \lt c \lt q \lt Z$ 
- **Case 4:**
	- Parent is a 3-node, sibling is 3-node
	- We have $A \lt p \lt B \lt s \lt C \lt r \lt D \lt q \lt Z$ 
		![](Pasted%20image%2020261018182646.png)
MOVE NEXT LOWEST/NEXT HIGHEST THING TO BUBBLE AND PRESERVE ORDERING/INVARIANTS ACROSS REST OF THE TREE

##### Questions
- Is the invariant conserved?
	- yes, we respect the sorting order of the respective internal and terminal nodes, the terminal nodes are kept at the same level and we ensure we only have 2 and 3 nodes in the tree.
- The cost remains $O(log n$), our worst case is applying case $1$ $h$ times, up to the root, $O(h$) operations. 

### Last words on 2-3 Trees
- Efficient and elegant solution to balance search trees
- Technically, this is not a BST, but is what we call a **Multiway Search Tree**
	- A BSt stores one key per node, with arity 2, and this is strict
	- 2-3 keys permit 3-nodes also. 
	- This makes the search space split inconsistently, with more comparisons (but generally more 'divisions' of said search space as a result, and with less $h$)
	- It is called *multiway* because while the ordering principle is the same as BST, a node can direct us to more than two subtrees and naturally this flexibility is one part of how 2-3 trees maintain balance in operations. 
		- As they can 'adopt' additional keys beyond what a BST can with the same amount of nodes with say, repeated insertions. 
- There is a 1:1 mapping between 2-3 trees and **Left Leaning Red-Black trees**, which we will cover shortly.
- 2-3 trees can be generalised $(a, b)$ trees, but 2-3 is the simplest and good enough for the properties we would like.
	- We have between $(a, b)$ children, and a node with $k$ children contains $k-1$ keys.
	- We would increase the branching factor because we can store more keys within fewer levels; naturally better given the upper bound for tree ops is $O(h)$ 
- Can be configured to match system cache.


### Tree rotations
- A ***tree rotation*** is an operation on an edge/link in a tree
	![](Pasted%20image%2020261018185940.png)
	A rotation is **right** around a key where the shift is right (not the link) and vice versa.
- Naturally, the BST invariant must be preserved
- Here that would mean preserving $A < B < C < D < E$ 
- This gives the tree a different shape.
- This operation can affect the height of the tree, for the example above:
	- $left = h(D)= max(h_B, h_E) + 1 = max(h_A + 1, h_C + 1, h_E)+1$ 
		- Replace $h_B$ with its own height formula.
	- $right = h(B) = max(h_A, h_D) + 1 = max(h_A, h_C  + 1, h_E+1) +1$
- The expansion is similar, except for the fact that the height of A has changed, and the height of E has changed. We change the height of the respective subtrees by one for any given rotation. 
	- Rotation cost = $O(1)$ and we can use rotations to maintain balance to enforce $O(log n)$

### Red-black trees
- A ***red-black tree*** is a BST with **additional colouring rules**
- Each node contains:
	- A key, as in an ordinary BST
	- References to its left and right children, as in a BST
	- A **colour**: red or black
- The ordinary BST ordering property still applies:
	- $LeftSubtree < Node < RightSubtree$ 
- Additional colours help us maintain balance during insertion and deletion, without augmenting the node structure severely.
- A red-black tree satisfies the following properties:
	1. Every node contains an extra colour value: **Red** or **Black**
	2. The root (and leaves) are **black**
	3. The **children of a red node are black**(no consecutive red nodes)
	4. For a given node, **every path from it to a leaf has the same number of black nodes.**
- **Black-height** = # of black nodes in any path from node $n$ to leaf $l$.

#### Red-Black Tree Example
![](Pasted%20image%2020261018195015.png)
- All root-leaf paths contain 2 black nodes: thus $black-height = 2$
- It is important to note that a **leaf** in the formal red-black tree definition is actually a **NIL** node, representing an absent child rather than a terminal node containing a key. 
	- The small black dots represent those NIL leaves.
- We are permitted only $Black \to Black$, $Black \to Red \to Black$.
	- This enforced alternation ensures a loose balance of the tree.
	- The shortest path a to a leaf node consists entirely of black nodes, but naturally, we have to preserve the black height from the root, so we cannot just only add black nodes everywhere.
		- Operations maintain this balance and make correct judgments on red/black placement to maintain the black-height invariant. 
#### Red-Black Tree Height
- For a red-black tree with height $h$
	1. No two red nodes are consecutive, so every root-to-leaf path has at least $h /2$ black nodes $\to b \ge h/2$ 
		- Along the longest path, at least approximately half of the nodes must be black, given the alternating colour rule and that we start from a black node.
	2. A tree with maximum black height $b$ contains at least $2^b-1$ nodes.
		- Just stick with this, the black height is the lower bound we're defining, as in, the lower bound is all black nodes as $h$.
		- The tree isn't perfect either so we can't just directly use $2^h-1$, branches can have different lengths so long as the black height for the given node(s) is equal.
- Thus:
	- $n \ge 2^b-1$ 
	- $n+1 \ge 2^b$ 
	- $n + 1 \ge 2^{h/2}$ 
	- $log_2(n+1) \ge h/2$ 
	- $2log_2(n+1) \ge h$ 
- Roughly, $h = O(log \ n)$ 