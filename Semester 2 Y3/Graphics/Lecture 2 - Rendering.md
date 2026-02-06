# Rendering
> Rendering is the process of automatically generating a 2D image or video from 3-dimensional monitors, which need to be rendering to a 2-dimensional display. Final stage of the 3D pipeline. 

Graphics programmers start with models, constructed via 3D geometry, which then need to be rendered to a 2D display. Models are constructed using *primitives*, including points, lines, and $n$-sided polygons. We focus on models made of triangles, as graphics hardware is optimised for rasterising triangles, and any complex shape can also be broken down into triangles. 

In the below figure, can see triangles repeatedly rendered as a wireframe, by drawing only the lines connecting the points making the triangle. Large, realistic models may consist of hundreds of thousands or millions of triangles.
	![](Pasted%20image%2020260205235054.png)
	Model constructed from many triangles.

## Terminology
- Triangles are made from 3 points, each of which is a called a **vertex**.
- Each vertex has a position in 3D space and so is represented as a **vector**.
	- Recall that, informally, a vector is an element of vector space, satisfying addition and scalar multiplication axioms. Essentially a point in space and can be represented geometrically. 
	- Can either be row-major or column-major form
	- We can visualise vectors as arrows from the origin of a vector space, the end of the vector represents the vertex position in that space. 
- A computers display is made up of a pixels, the smallest addressable element in a raster image. 
- A triangle is rendered in OpenGL by projecting the 3 vertices from 3D to the 2D screen, then shading the pixels inside the triangle on the screen. 
- A fragment is a potential pixel generated during rasterisation; it represents a *piece of triangle* that covers a pixel-sized area on the screen, it is not yet a pixel. 
	- Vertices define shapes, triangles cover areas, those areas become fragments if they are pixel sized
	- Prior to rasterisation, there are triangles in screen space, and no pixels have been touched yet
	- During rasterisation, the GPU checks which pixels are inside each triangle, and for every covered pixel, it generates a fragment
	- Usually one fragment per pixel per triangle, but multiple fragments can target the same pixel, each overlap producing a fragment, with the GPU deciding which fragment wins. 
- A colour buffer is a section of memory that stores colour values for pixels that will be used at each pixel on the display
- Historically, the term shading referred to the method used to determine the colour of fragments, overtime, the term shading encompassed more concepts. Thus, a shader is a program that is executed on the graphics hardware.
	- We are concerned with vertex and fragment shaders, not tesselation and geometry shaders.


## Rendering pipeline OpenGL
- Rendering includes the following 5 main steps:
	- Specify vertex data
	- Execute a vertex shader to calculate each vertex position
	- Assemble triangles from the vertices
	- Rasterise the triangles to create fragments
	- Execute a fragment shader to calculate the fragment colours. 
		![](Pasted%20image%2020260206000912.png)

### Vertex Specification
- Vertices are stored in memory, usually as an array
```C
float vertices[] = {
  -1.f,  0.f,  0.f, // v0
   0.f, -1.f,  0.f, // v1
  -1.f, -1.f,  0.f  // v2
};
```
- Yields:
	![](Pasted%20image%2020260206005726.png)
	Since working in 2D, no depth, and the z-value can be ignored. 


### Vertex shader
- Runs once per vertex
- Calculates the position of each vertex. This will be covered in more detail for later on, for now, just assume the vertex shader simply returns the vertex positions each just as they were set during the vertex specification
- Later, will apply transformations e.g., rotation, scaling, projection. 

### Triangle Assembly
- A triangle is constructed from 3 vertices, it is the simplest 2D shape, in 3D, it is guaranteed to be planar i.e., the 3 vertices exist on the same plane. 
	- This is intuitively obvious, given the definition of a plane in linear algebra; two linearly independent vectors span a 2D subspace and that subspace is a plane, taking one vertex/vector a a point on the plane from which the other two can be added to yield the vertex plane. 
- Triangle assembly is the process of taking vertices and organising them into triangles. 
	- The order matters.
		![](Pasted%20image%2020260206010533.png)

### Rasterisation
- This stage determines which pixels are covered by each triangle, by converting triangles into fragments
- *Rasterisation* is the process that calculates which pixels on the screen are contained in a triangle that is being rendered, so fragments can be generated. 
	*For each pixel y in Y dimension*
	*For each pixel x in X dimension*
		*For each triangle t*
		*If pixel x, y is inside triangle t*
			*Calculate the fragment colour*
#### Barycentric co-ordinates
- Barycentric co-ordinates are a way of describing the position of a point relative to a triangle, rather than in global (x, y) co-ordinates.
	- Barycentric co-ordinates are 3 co-ordinates $\alpha$, $\beta$, $\gamma$ 
- Suppose we have a triangle constructed from 3 vertices $A, B, C$ and a point $P$
- We can describe $P$ using barycentric co-ordinates, which tell us how much influence each vertex has on the position of $P$.
- The barycentric co-ordinate $\alpha$ 
	- Corresponds to the normalised linear distance of $P$ between the line $BC$ and the point $A$
- The barycentric co-ordinate $\beta$ 
	- Corresponds to the normalised linear distance of $P$ between the line $AC$ and the point $B$
- The barycentric co-ordinate $\gamma$ 
	- Corresponds to the normalised linear distance of $P$ between the line $AB$ and the point $C$
- Each barycentric co-ordinate is at most 1, with the constraint that $\alpha + \beta + \gamma = 1$
	- If $0 \le \alpha \le 1 \ and \ 0 \le \beta \le 1 \ and \ 0 \le \gamma \le 1$ , then the point is inside the triangle
	- We calculate $\alpha, \beta, \gamma$ using the **implicit line equation** as follows:
		$line(A, B, P) = (B_y - A_y)P_x + (A_x - B_x)P_y + B_xA_y - A_xB_y$ 
		Where $A$ and $B$ are start and end points of the line segment, and $P$ is a point.
	- Yields a single scalar value:
		- = 0 $P$ lies exactly on the line
		- > 0 $P$ lies on one side of the line
		- < 0 $P$ lies on other other side
		- Essentially a **half-space test**
	- We can can calculate $\alpha, \beta, \gamma$:
		- $\frac{line(B,C,P)}{line(B, C, A)} = \alpha$
		- $\frac{line(A,C,P)}{line(A, C, B)} = \beta$
		- $\frac{line(A,B,P)}{line(A, B, C)} = \gamma$

##### Appendix A:
![](Pasted%20image%2020260206012318.png)
##### Appendix B:
Cover at another point

### Fragment shader
- Executes on the GPU and computes the final colour of a fragment
- Fragment shaders receive data that has been interpolated from the triangles vertices using barycentric co-ordinates 
- Typical inputs include fragment position, depth value, colour, texture co-ordinates. 
