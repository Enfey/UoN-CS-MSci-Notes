## Why Transparency is sometimes required and sometimes not
- Sometimes an object may behave as transparent in real life, but rendering it opaque does not visually break realism
- For example, it is possible to avoid rendering the transparency and instead use opaque surfaces; in the figure below the visor on the helmet can be drawn opaque, and the viewer doesn't really notice a problem because it looks the same as in real life. 
	![](Pasted%20image%2020260218212637.png)
- However, there are times when it will look unnatural to render a transparent surface as it were opaque. Transparency becomes necessary when:
	- Intended to see objects behind the surface clearly.
	- The camera moves; when the camera moves **parallax occurs**
		- Introduced during perspective projection during the perspective divide, nearer objects move more on the screen than distant objects when the camera moves, giving us depth perception.
		- With correct transparency, through the glass, we see the glass surface, the objects behind it, and the objects behind shift differently from the glass as the camera moves preserving the depth relationship
		- Without correct transparency, the background may disappear entirely so the geometry behind is not seen at all, fragment does not contribute to pixel colouring.
		- Something else that may happen is incorrect blending, the glass and the background may be blended in the wrong order, or the blending is mathematically incorrect, the final pixel colour for example may combine glass and a tree as if they were at the same depth. 
- We need a flexible method for rendering a transparent surface when it is required. 

## Alpha blending
- Colours can be represented with Red, Green, and Blue components, and an additional component called Alpha, then a colour is represented using 4 component RGBA vectors.
- The alpha co-ordinate is used in a technique called **Alpha Blending**, which simulates how much colour passes through a transparent surface. 
- Alpha Blending can be used in the rendering pipeline can be used in the rendering pipeline (we discuss in terms of rasterisation for now) after the $z-buffer$ test has past, which means the fragment being rendered is the one closest to the camera, and the colour of the fragment will replace the colour stored in the colour buffer.
- For Alpha Blending, the following formula is used for copying the fragment colour to the colour buffer $$c_{b} = a_{s}c_{s}+(1-a_{s})c_{d}$$
	where $c_b$ is the updated colour stored in the colour buffer, $a_s$ is the alpha co-ordinate of the current fragment, $c_s$ is the colour of the current fragment, and $c_d$ is the old colour in the colour buffer.
- Definitionally if $a_s$ is set to $1.0$ then the formula simplifies to full replacement of the colour in the colour buffer with the new colour. If $a_s$ is  less than $1.0$ then a blending of the two colours is the result. 

### Render Order
- For alpha blending to correctly render transparent objects, the triangles must be rendered
	1. All opaque triangles first
	2. All transparent triangles back-to-front
- This is necessary; all opaque are rendered, contributing to the colour buffer, and then can alpha blend with the values in the colour buffer.
	- If transparent surfaces were rendered first, later opaque fragments behind them would fail the depth test, and would result in missing geometry. 
- By back to front, we mean:
	![](Pasted%20image%2020260218222118.png)
	- Each transparent layer then blends onto the accumulated result behind it, simulating light passing through multiple layers to correctly shade the pixel. 
		- If we went front to back, it blends on the background(where blending is not commutative) and the opaque fragments will not pass the z test
		- Even if we drew opaque triangles first, and then did front to back for the transparent fragments like below:
			![](Pasted%20image%2020260218222401.png)
			The fragments of the 4th render will not pass the z test, and its colour will not be blended with the transparent fragments/opaques in front/behind, respectively.
- Because blending depends on ordering, we must sort trapsrent surfaces by distance from the camera before rendering. 


### Primitive sorting
- Primitive sorting sorts the triangles using the distance between the centroid of each triangle and the camera position.
- Object sorting sorts by the centroid of the entire object mesh (can be incorrect if object spans a large depth range).
- It should be noted that rendering of opaques comes first, and that this is generally enforced explictly by the renderer
- So we only need to sort the the transparent geometric primitives back-to-front. 
- For a triangle with vertices:
	$A(x_A, y_A, z_A), B(x_B, y_B, z_B), C(x_C, y_C, z_C)$
	- The triangle centroid is:
		$$
\begin{aligned}
cen_x &= \frac{x_A + x_B + x_C}{3} \\
cen_y &= \frac{y_A + y_B + y_C}{3} \\
cen_z &= \frac{z_A + z_B + z_C}{3}
\end{aligned}
$$
	- This yields the average position of the triangle as a single vector. 
	- We then compute the distance to to the camera, and sort the triangles in descending order. 
	- We reorder the index buffer, issuing draw calls using the sorted index order to draw transparent triangles in the correct order. 
- Centroid sorting assumes geometry is entirely in front or behind one another; if there are cyclical intersecting relationships, there is no sorting order that can satisfy a particular geometry, which is why transparency remains a difficult problem. 