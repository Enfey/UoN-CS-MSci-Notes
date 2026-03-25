## Fundamental problem
- A modern game scene may contain millions of triangles.
- In ray tracing, for every pixel on the grid at least one primary ray is fired, which for 1080p amounts to 2 million rays per frame.
- If we naively tested every single triangle:
	- `2,073,600 rays × 1,000,000 triangles = 2,073,600,000,000 intersection tests`
- At 60fps this is 124 trillion tests per second, which is completely impossible in real time.
- This is the $O(n)$ complexity we seek to avoid, and the one that the use of a BVH - a spatial data structure used to accelerate rendering, can reduce to $O(log n)$.

## BVH
- A *Bounding Volume Hierarchy* or BVH is a spatial data structure used to accelerate rendering. 
- A BVH can be applied to the rasterisation pipeline through a method called **View Frustrum Culling** where only the geometry inside the view frustrum is rasterised.
- A BVH can be applied to ray tracing to accelerate ray tracing by reducing the number of ray triangle intersection tests.
- Additionally a BVH can be applied to general collision detection between objects. 
- In a BVH a *Bounding Volume* is used to contain geometry
	- Examples of bounding volumes are:
		- Spheres
		- Axis-Aligned Bounding Box AABB
		- Oriented Bounding Boxes
	- The motivation is to use the bounding volume is an intersection test which is computed much more quickly than testing the enclosed geometry, with the idea that a failed intersection test with a bounding volume with necessarily fail intersection tests with all enclosed geometry.
		- There are some false positives due to the bounding box not being 'filled' with geometry entirely, and having gaps, so some intersections intersect the box but none of the enclosed geometry
		- Bounding volumes behave much like a bloom filter in that there are no false negatives.
- A BVH has a tree structure:
	- The tree has a root node, internal nodes, and leaf nodes
	- Each node is a bounding volume which contains all of the geometry in its subtree of bounding volumes (or just simply the geometry if it is 1 level above the leaf nodes)
	- The leaf nodes represent the actual geometry of the scene. 
	![](Pasted%20image%2020260325010618.png)![](Pasted%20image%2020260325010622.png)
	- A tree where internal nodes have at most $k$ children is called a $k$-ary tree. Binary trees are exceedingly common for a BVH.

## Bounding Volume Hierarchy Construction
- An **axis-aligned bound box** AABB has the property that all faces have a nromal that coincide with the standard basis vectors, corresponding to the world axes.
- Thus, the box is never rotated, its faces are always parallel to the co-ordinate planes - if it were to be tilted or rotated the face's normals  would no longer align with the basis vectors and it becomes an OBB instead.
- Because the box is axis-aligned and never rotated we only need 2 corner points to fully define it.
	1. $ftl$ = front top left = $(min_x, max_y, max_z)$ 
	2. $bbr$ = back bottom right = $(max_x, min_y, min_z)$ 
		![](Pasted%20image%2020260325012509.png)
	These sit on opposite diagonal corners of the box. Once we know these two points, every other vertex and face position is completely determined via these two points. 
		$Width  = bbr.x - ftl.x$
		$Height = ftl.y - bbr.y$
		$Depth  = ftl.z - bbr.z$
- To build an AABB which tightly encloses the geometry inside, we simply take the maximum extents in the $x, y$ and $z$ dimensions of all the enclosed vertices.
	$ftl.x = \text{minimum x across all vertices}$ 
	$ftl.y = \text{maximum y across all vertices}$ 
	$ftl.z = \text{maximum z across all vertices}$ 
	$bbr.x = \text{maximum x across all vertices}$ 
	$bbr.y = \text{minimum y across all vertices}$ 
	$bbr.z = \text{minimum z across all vertices}$ 
- The binary BVH construction pseudocode is shown below, which shall be called with all of the geometry to be rendered as a parameter and return a pointer to the root node of the constructed BVH. 
```
1. Create tight AABB bounding all geometry
2. If geometry count < threshold
	   Add geometry to leaf node
	   return leaf node
3. Split into left, right at the middle of the longest axis
4. BVH.L = Construct_BVH(left)
5. BVH.R = Construct_BVH(right)
6. Add BVH-l and BVH-r to child nodes.
```


### Bounding Volume Hierarchy Traversal
- To use a BVH in a ray tracing pipeline, the traversal begins by testing an intersection of the ray with the root bounding volume. 
- If the ray misses this bounding volume, then it must miss all geometry contained inside, which is why the speed is so evident.
- If the ray intersects, then test against all child nodes of the root in the same manner.
- If the ray misses any bounding volume, then it will miss all geometry in that sub-tree.
- If it hits a bounding volume, then it needs to test ray-intersection against all child bounding volumes until it reaches the leaf nodes.
- When a ray intersects the bounding volume of a leaf node, then the ray is finally tested against the concrete geometry of the scene. 
```
Traverse_BVH(ray, node):
1. If ray misses node's bounding volume -> return miss
2. If node has no children -> return closest triangle intersection
3. Traverse all child BVHs -> return closest intersection
```
- Note that we also track the closest hit distance found so far and use it to prune the search space; if the nearest hit so far is less than the distance of a bounding volume being tested against, then the bounding volume and its subtree are not tested for intersection as they cannot possibly contain a closer intersection. 

### Ray Intersection with Bounding Volume
- Recall the equation for a ray: $$q(t) = o + dt$$
- We can solve for $t$ when testing if a ray intersects with an AABB as follows:
	- We separate the $x, y$ and $z$ components of the ray. 
	- The $x$ component of the point where the ray intersects the plane of the $x$ component of the $ftl$ of the AABB is:$$ftl_{x} = o_{x} + d_{x}t_{0x}$$
		Which can be solved for $t_{0x}$:
		$$\frac{ftl_{x} - o_{x}}{d_{x}} = t_{0x}$$
	- We then solve for $t$ for where the ray intersects the plane of the $bbr$ $x$ component, and likewise for the other components.
	- We essentially determine for the fixed components, the value of $t$ for the infinite plane defined by the fixed component values. 
	- Each pair e.g., $t_{0x}, t_{1x}$ gives us an interval - the range of $t$ where the ray is inside that 'slab' which is defined by the two planes. 
	- The intervals must overlap for the ray to be inside all the slabs at the same time for respective components. This essentially redefines the bounding box in the co-ordinate space.
	- Define:
		- $tmin = max(t_{0x}, t_{0y}, t_{0z})$
		- $tmax = min(t_{1x}, t_{1y}, t_{1z})$ 
		- The ray intersects the AABB if and only if $tmin ≤ tmax$ 
			- That is $tmin$ determines the maximum $t$ value whereby the the ray entered one of the slabs defined by the components of $ftl$ 
			- $tmax$ determines the minimum $t$ value whereby the ray exited one of the slabs defined by $bbr$
		- If it has already left on one plane by the time it is entering another, then the intersection is invalid IDFK.