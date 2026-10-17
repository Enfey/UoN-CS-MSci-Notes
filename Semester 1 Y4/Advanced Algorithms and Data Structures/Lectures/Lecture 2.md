## Sorting
- Sorting of array elements (random access is assumed)
- Requires order defined on the elements e.g., numerical, lexicographical, custom etc
- Useful for a lot of things e.g., binary search. 
### Quicksort - divide and conquer sort
- Steps:
	- Choose an element called the $Pivot$ 
	- **Partition:** Modify the array s.t.
		- Elements $\lt$ $pivot$ are sent to the **left**
		- Elements $\gt$ $pivot$ are sent to the **right**
		- $Pivot$ is therefore in its final position (everything strictly greater or smaller than it has been moved, but naturally those sub-arrays have not been sorted yet)
	- Recursively apply this strategy to the left and right sub-arrays

```python
def quicksort(array, start, end)
	if end - start > 1:
		p = partition(array, start, end)
		quicksort(array, start, p-1)
		quicksort(array, p+1, end)
```

#### Quicksort: Partitioning
- Note that numerous variants exist to choose the pivot.
- Here we choose the "Lomuto partition scheme" where the pivot is always the last element in the sub-array
	- The loop for this variant simply ensures elements that are strictly smaller than the pivot are moved to the left, and a 2nd point $i$ keeps track of the boundary between elements smaller than the pivot and larger than it, simply swap the pivot at the end of the loop to $i+1$.
	- If an element is larger keep it where it is, otherwise swap with $i$ and increment $i$ to move the boundary forward.

```python
def partition(array, start, end)
	pivot = array[end]
	i = start
	for j in range(start, end):
		if array[j] < pivot
			array[i], array[j] = array[j], array[i]
			i+=1 # has been sorted this turn, marks the
			# elements smaller than the pivot
	array[i], array[end] = array[end], array[i] # swap pivot directly in front of the elements it is strictly larger than to attain its final position.
	return i
```


### Properties of sorting algorithms

#### In place-ness
- An algorithm is generally considered **in-place** if it modifies its input directly, without requiring substantial additional storage.
	- The lomuto partition variant of quicksort is considered in-place via its two pointer approach to tracking elements smaller than the pivot rather than instantiating separate sub-arrays to recursively invoke the algorithm on. 
	- Naturally though, there is a small memory cost for recursive-calls and subsequent deep unwinding but this is often negligible compared to what the memory cost would be if not in-place.
- An algorithm is considered **out-of-place** if it creates additional new working copies of the input, or maintains other large data structures to perform its work/sorting.
	- In conventional merge sort, we recursively divide arrays and then marge sorted positions
	- To perform the merge step we usually allocate a temporary array, requiring $\Theta(n)$ extra space for merging, as well as recursion stack space.

#### Stability
- A sorting algorithm is said to be **stable** if element with equal sorting keys retain their original relative order.
- This is not as simple as doing strict comparisons within the algorithm.
- Quicksort is generally not considered to be stable. 
	- For the lomuto scheme we defined above, the reason for this is the final swap. 
	- If we have an array where all elements are smaller than the pivot already, the inner loop will perform no work.
	- The swap at the end however, will swap with $i$, which if we haven't moved (or some similar scenario where $i$ has an equivalent that it is next to) then the pivot will move to where $i$ was.
		- This reverses their relative order:
			![](Pasted%20image%2020261016221503.png)


#### Efficiency - Quicksort
- Worst case: $O(n^2)$ 
- Average case: $O(n \ log \ n)$ 
- Small standard deviation $O(n)$ 

- Best case: approximately balanced partitions.
	- Suppose we have $n$ elements and each pivot splits the remaining elements approximately evenly.
	- At each level, we get two similarly sized recursive problems
	- This recurrence is approximately: $T(n) = 2T(n/2) + cn$ 
		- Using the master theorem, $a = 2, b = 2, c = log_2(2) = 1, n^1 = 1$ thus case 2, therefore, set $k = 0$, n log n trust.
##### Average Case
 - $Rank$ of pivot is its index in the sub-array
 - If pivot has rank $i$, the left subarray **size** is also $i$ 
 - If pivot has rank $i$, the right subarray size is $n-i-1$(exclude the pivot itself)
 - Therefore, the recursive work for a pivot of rank $i$ is $T(i) + T(n-i-1)$ 
 - IF we assume that every pivot $rank$ is equally likely, we have $1/n$ 
 - The expected recursive cost is then the average of all possibilities:
	 ![](Pasted%20image%2020261016232234.png)
	 We can simplify this however, as $T(i) + T(n-i-1)$ are actually just each others inverse. 
	 ![](Pasted%20image%2020261016232349.png)
	 So their sums are equal, and we can combine them
	 ![](Pasted%20image%2020261016232427.png)

- The intuition behind this average case, is that the rank of the pivot is random in each sub-array in the average case, but in particular, is between the 25th and 75th percentiles with $p=0.5$, 75-25 = 50
- At most, 75% of the elements in one sub-array
- If a good pivot has probability $p = 0.5$ then the expected number of attempts until we get one is $1/0.5 = 2$ 
	- At most, we generally reduce the size to $3/4$ x $n$ 
- The depth/height of the tree is determined by finding how many good partitions it takes to reduce the array of size $n$ down to $1$ element. 
	- A good partition leaves us with at most $3/4$ x $n$
	- We do this $k$ times
	- $n(\frac{3}{4})^k = 1$ 
		![](Pasted%20image%2020261016234422.png)
		![](Pasted%20image%2020261016234430.png)
	- Which leaves us with $n = (\frac{4}{3})^k$ 
	- By inverting the square we are left with $log_{\frac{4}{3}}n$ 
	- This is technically an upper bound on the remaining size after $k$ good partitions, rather than necessarily the exact size. 
- $O(n)$ at each level from the scan/partition itself
- Total cost $O(n log n$)

##### Worst case
- Partitioning array of size $n$ examines approximately $n -1$ elements where $p$ is chosen as the last element. After partitioning, recursively partition subsequent arrays which require $n-2$ comparisons, $n-3$ comparisons, as we slowly obtain $rank$(s) for the pivot(s). 
- As such we eventually reach array with one element, therefore the total number of comparisons is $(n-1) + (n-2) + (n-3) + ... + 2 + 1 + 0$ 
- We can express this as $\sum^n_{i=1}(n-i)$ 
	- Where the increase in $i$ expresses the removal of $p$ and subsequent less comparisons.
- We can express this as a multiplication ![](Pasted%20image%2020261017001134.png) which also makes it easier to derive the time complexity (dont even worry about it lad)

#### Choice of Pivot
- We can choose any element as the pivot, this can be random, fixed e.g., middle element, or some other scheme.
- However,m there is always a worst-case permutation regardless of the pivot chosen
- Certain partition schemes are good for avoiding certain scenarios. 
- In practice, we generally use the median-of-three pivot, where we look at the first, middle, and last elements of the subarray.
	- Compare first, middle, and last elements, and swap them so they are ordered with respect to one another.
	- Naturally, this changes the partitioning.
		- We swap the chosen median with the last element.
	- If sorted arrays are frequent, this is better strategy, as we do not blindly select the last element and perform $n$ work $n$ times, never once dividing the problem set, the resultant recursion tree is left-balanced.
- Choosing the **exact median** means we can always split exactly in half and achieve $O(n log n)$ every time, however, it is heavy to compute and often not done in practice. 



#### Comparison and Non-Comparison sorts
- **Comparison sorts**
	- With $T(n)$ comparisons, the permutation of distinct outcomes is $2^{T(n)}$ (binary decision) 
	- There are $n!$ possible correct permutations
		- $n(n-1))(n-2)(n-3) ...$ 
	- To ascertain the correct ordering of $n$, the permutation of distinct outcomes/comparisons must exceed or equal the number of possible correct permutations **in all cases.**
		$2^{T(n)}\geq n!$
		If we take the logarithm base 2 of both sides, we can simplify the LHS to $T(n)$:
		$T(n) ≥ log_{2} n! = Ω(n log n)$ (stirlings approx.)
- Common complexity/lower bound for comparison based sorting algorithms. 
- Non-comparison sorts can do better under certain circumstances, but are not suitable for all data types.

### Stability, In-Placeness, and Efficiency
- It is not easy to achieve all 3: **stability, in-placeness, and efficiency**
	- Quicksort vs mergesort
	- Quicksort is not stable, mergesort is stable
	- Quicksort is in-place, mergesort is not
	- Quicksort has $\Theta(n\log n)$ average time complexity with merge sort having the same
	- Quicksort has $\Theta(n^2)$ worst time complexity, with merge sort having $\Theta(n \ log \ n)$ 
	- Neither satisfy all 3.
- The underlying difficulty is that these properties can conflict with one another, and if not, the complexity of the algorithm and its correct implementation becomes exceedingly difficult.
- **Adaptiveness**
	- An algorithm is **adaptive** if it can take advantage of ordering that already exists in the input
	- Real-world arrays aren't always completely random, sometimes they're already sorted, nearly sorted, contain sorted sub-arrays etc.
	- Exploiting this will yield a best-case complexity of $\Theta(n)$ 
	- But on a reverse-sorted array, it may need to shift many elements repeatedly(for a comparison-based sorting algorithm), yielding $\Theta(n^2)$
		- Compared with merge sort that is $O(n \ log \ n)$ on already sorted inputs however...
- **Cache-friendliness**
	- Asymptotic complexity doesn't capture this notion particularly well, as here we delve into real-world performance.
	- The notion of spatial locality factors in here, e.g., accessing array in order vs random indices, the former is clearly more cache-friendly because when CPU loads cache line it will pull adjacent data, and have less cache misses.
	- Quicksort has relatively favourable memory access patterns due to its sequential access by partitioning contiguous sections of an array.
	- Merge sort also benefits from sequential access when merging, but there is considerably more data moving and you have no say over where the arrays are placed.
- **Writes vs comparisons**
	- A comparison determines relative ordering of 2 elements $A$ and $B$
		- A subsequent swap typically requires multiple reads and writes.
	- A write modifies the data stored in memory.
	- These operations are naturally, not equivalent
	- Imagine sorting a large collection of records, containing a key and some additional data
	- Key comparison is cheap, but repeatedly swapping them to order them based on key value is expensive depending on the size of the record. 
	- Conversely, if comparing two elements requires an expensive long string comparison or custom comparator, but the data is small(doesn't apply to the long string example obviously) then reducing the number of comparisons may be favourable to reducing the number of writes.
- Blocksort
	- Variant of merge-sort that uses buffers/blocks within the array itself to facilitate merging, rather than requiring a separate array, to achieve an in-place, efficient, stable sorting algorithm.
- Timsort
	- Hybrid sorting algorithm combining ideas from merge sort and insertion sort


### Interpolation Search
- If an array $A$ is sorted, we can use binary search to find the index of value $x$
- Interpolation search is similar to binary search, except instead of checking the middle value of an array, it predicts where the element should be based on its value
- The complexity of binary search: $T(n) = T(n/2) + O(1) \to T(n) = O(\log n)$ (arity 1 tree, branching factor = 1 so the time complexity is just the height of the tree $k$ as the work done is constant)
- If we know the keys in our array however are uniformly distributed, we can make a better guess:
	- Suppose we have a sorted array containing values from 0 to 1000 and we're searching for 750
		- Importantly, uniform distribution doesn't mean equal spacing. There can be random gaps and sequences of equal keys.
	- Estimate where $x$ should be: $\hat{i} = \frac{x-A_{0}}{A_{n-1}-A_{0}}(n-1)$ 
		$A_0$ = smallest value in sorted array
		$A_{n-1}$ = largest value in sorted array
		Numerator tells us how far $x$ is above the smallest value, and the denominator tells us difference between largest and smallest value
		This gives us a percentage that we can turn into an array index by multiplying by $n$
	- Under a suitable random uniform input model, our estimated index is typically within roughly $\sqrt{n}$ positions
	- We get a time complexity of $T(n) = T(\sqrt{n}) + O(1)$
		- Recursive search step is constant, see the interpolation formula above.
		- Now to solve this recurrence:
			- We can actually apply the master theorem, if we can express the problem size shrinking per recursive call as a fraction instead of a square root.
			- Let $t = log_2n$ 
			- $n = 2^t$ 
			- $T(2^t) = T(\sqrt{2^t}) + O(1) = T(2^t/2) + O(1)$ 
				- Via this which i cba writing down myself
					![](Pasted%20image%2020261017013454.png)
			- 
	- $O(log \ log \ n)$ 


### Dynamic Set

### Abstract Data Type

### Implementations


### Binary Search Tree

