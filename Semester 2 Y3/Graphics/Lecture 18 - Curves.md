## Why use curves?
- The naive approach to representing any smooth shape is to approximate the geometry with many small triangles.
- This works for rendering, but for any degree of animation, this is incredibly wasteful.
	- Imagine trying to define the shape of a car bonnet via thousands of vertices
	- Moving the bonnet's curvature would require adjusting thousands of vertices simultaneously. 
	- Note that triangles are still the final representation. 
- Curves solve this by letting us define complex smooth shapes with just a handful of *control points*, which then have the smooth detail between them filled in. 
- We can use curves to define the mesh of 3D objects too, permitting smooth deformation during animation. 


## Bézier Curves
- A **Bézier Curve** is a smooth curve defined by **control points** where:
	- The curve always starts at the first control point
	- The curve always end at the last control point.
	- Interior control points **pull** the curve toward them without the curve necessarily passing through them. 
	- The curve is always contained within the **convex** hull of the control points.
- There needs to be at least two control points:
	![](Pasted%20image%2020260325180429.png)
- Defining more control points pulls the curve:
	![](Pasted%20image%2020260325180449.png)
	![](Pasted%20image%2020260325180504.png)
	- This bottom Bézier curve - a cubic Bézier curve is the most commonly used form. 
	- With 4 points, we get an S-curve capability as the figure shows.
	- The first control point sets the start, the last sets the end, and the middle two control the tangent directions at each end.
- A curve is rendered firstly by an algorithm generating points along the curve which is specified using the control points. 
- These points are then joined and and rendered using straight lines. 
- Each point is generated using an **evaluation**
- The number of generated points, and therefore the number of small lines rendered, is a parameter, with more evaluations resulting in a smoother curve and also a more computationally expensive process. 
	![](Pasted%20image%2020260325180941.png)

### De Casteljau's algorithm for Bezier Curves#
- Points along a curve can be produced via this algorithm:
```C
EvaluateBezierCurve(ctrl_points, num_evaluations)
{
	offset = 1 / num_evaluations
	initialise empty curve
	curve.append(fst(ctrl_points))
	
	for each e in 0..num_evaluations-1
	{
		p = evaluate(offset * (e+1), ctrl_points)
		curve.append(p)
	}
	return curve
}
```
- E.g., 5 control points, 20 evaluations
- offset = 0.05
- for e in 0..19, evaluate 0.05 x e+1 
```C
evaluate (t, P)
{
	Q = P
	while Q has more than 1 element:
	{
		R = []
		for each adjacent pair p1 p2 in Q
		{
			p = ((1 - t) * (p1)) + (t * (p2))
			R.append(p)
		}
		Q = R
	}
	return first element of Q
}
```
- Calculates the generated point given control points $P$ at parameter $t$
- We create a worklist $Q$ of points that we shrink until it only contains one point. 
- We compute the linear interpolation for the adjacent pairs of points in the worklist, and append the resulting interpolated points in R.
- We then move onto R.
- When the loop ends, that single point is the exact point on the curve at parameter $t$.


### Example execution of De Casteljau's algorithm
- We begin with three control points, and set the number of evaluations to 5.
	![](Pasted%20image%2020260325184210.png)
- The algorithm will compute points on this curve for offsets $\frac{1}{5}$, $\frac{2}{5}$, $\frac{3}{5}$, $\frac{4}{5}$, $1$.
- First evaluation $t = \frac{1}{5}$:
	- We linearly interpolate along the control points to yield points $AB$ and $BC$ which are $1/5$ of the way from $A$ to $B$ and $B$ to $C$, respectively.
	- We then linearly interpolate between those two points, yielding a new point $ABC$ which is $1/5$ of the way from $AB$ to $BC$ which becomes the first point on the bezier curve as there is no other point to linearly interpolate with. 
		![](Pasted%20image%2020260325184802.png)
	- This then continues e.g., 
		![](Pasted%20image%2020260325184821.png)
		![](Pasted%20image%2020260325184836.png)
		![](Pasted%20image%2020260325184844.png)
