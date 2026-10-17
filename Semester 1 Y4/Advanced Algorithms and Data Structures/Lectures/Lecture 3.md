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
- **Linked list**
- **Doubly linked list**
- **Hash table**
- **Graph**

![](Pasted%20image%2020261017192144.png)



### Balanced Trees

#### BST Height

#### Lemma for Number of Nodes in a Tree

#### Tree height

#### Balanced Trees



### 2-3 Tree

### Tree rotations

### Red-black trees
