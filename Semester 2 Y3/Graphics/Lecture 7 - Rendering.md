# Rendering pipeline
- We divide the **general rendering pipeline** into 4 stages:
	1. **Application(CPU)**
		Software that runs on the CPU, this a program written in C/C++ with a `main()` function. There is code in the application stage e.g., `glNamedBufferStorage()` which copies vertices to the GPU memory for the next stage, vertex processing. The application stage is where user input is handled, and object and camera positions are calculated.
	2. **Vertex processing(GPU)**
		Responsible for processing each vertex independently. In this stage, vertex positions are transformed from model space to world space and view space, and projected to clip space co-ordinates. This happens in the vertex shader. Triangles are clipped if they are not completely visible and perspective divide transforms vertex positions into normalised device co-ordinates. The $x$ and $y$ components are transformed to screen co-ordinates and the $z$ component is stored in a depth buffer, and these 3 values are passed to the rasterisation stage. 
	3. **Rasterisation(GPU hardware)**
		Responsible for calculating all the pixels that are inside the triangles which are being rendered. A fragment is a candidate pixel generated during rasterisation. A fragment contains the screen space co-ordinate, depth from the depth buffer, and interpolated attributes such as colour, but it is not yet the final pixel, because multiple fragments can map to the same pixel, only one will survive via depth testing with the $z$ buffer. We have not seen any implementation of OpenGL's rasteriser because it is implemented on dedicated hardware on the GPU.
	4. **Pixel processing(GPU)**
		The pixel processing stage is responsible for processing individual fragments. The fragment shader computes a colour which is then merged with the colour buffer. Merging involves calculating which fragments are hidden behind others via $z$ buffer testing, and only storing the colour for the visible fragment. 

## Application stage
- Vertex specification happens in the application stage which is running in software on the CPU
- Recall that during vertex specification the programmer creates an array of vertices for OpenGL to render and provides information about how the vertices are passed as input variables to the vertex shader.
	- This is done in tandem with the VAO which determines how the interpret the VBO array which is present in the GPU memory after being copied. 
- At some point in this pipeline, some vertices, a camera, and object positions have been specified. 
	![](Pasted%20image%2020260214233208.png)
	These appear in **model space**.
## Vertex processing
- The vertex processing stage consists of the following substages:
	1. **Vertex Shader**
	2. **Vertex post-processing**

### Vertex Shader
- In the vertex shader, vertices are transformed from model space to world space using the model matrix.
	![](Pasted%20image%2020260214234141.png)
- Model space vertices are then subsequently transformed to view space using the view matrix.
	![](Pasted%20image%2020260214234250.png)
- Finally, vertices are transformed to clip space using the projection matrix. Vertices here are now in a cube called a view volume, which represents what is visible to the camera. 
	![](Pasted%20image%2020260214234256.png)

### Vertex Post-processing
- This occurs immediately after the vertex shader has been executed. 
- Vertex post-processing includes:
	1. **Primitive assembly**
	2. **Clipping**
	3. **Perspective divide**
	4. **Viewport transformation**
#### Primitive assembly 
- Earlier stages process vertices independently; we turn these into structured primitives that we can render. 
- We have a stream of transformed vertices after the vertex shader $v_1 ... v_n$ we group these into geometric primitives based on the **draw mode**
- For example, if one draws in OpenGL with `GL_TRIANGLES` then every 3 vertices $v_{1}, v_{2}, v_{3}$ form one triangle.
- Another option would be to use `GL_TRIANGLE_STRIP` where every new vertex forms a triangle using the prior 2, reusing vertices. 
- Typically use an **index buffer** as vertices may not be sent in order, index buffer is used during primitive assembly to fetch corresponding vertices and build them.
#### Clipping
- Clipping is the process of removing vertices that are not visible. There are 3 cases
	1. The triangle is completely inside the view volume
	2. The triangle is completely outside the view volume
	3. The triangle intersects the view volume.
- In the below figure, the view volume is drawn with a red triangle completely outside the view volume, a blue triangle completely inside the view volume, and a green triangle which interescts the view volume.
	![](Pasted%20image%2020260215003850.png)
- The view volume is now drawn with the red triangle deleted, blue triangle unchanged, green triangle deleted, and **two new vertices added** where the clip was made, shown as black dots.
	![](Pasted%20image%2020260215003844.png)

#### Perspective divide
- Recall that perspective division transforms a vertex to normalised device co-ordinate (NDC) space via division by $w$ which is just the inverse of the depth component $z$ prior to applying the projection matrix (the projection matrix yields this result)
- Our objects now have perspective via this division(the green triangle will look smaller), and we are rid of the homogeneous component which is implicitly $w = 1$ now.
	![](Pasted%20image%2020260215004149.png)
#### Viewport transformation
- Recall that vertices in NDC space are transformed to screen space using the viewport transformation
- After vertex postprocessing, the triangle vertices are in 2D screen space with an additional $z$ component used for depth testng, and a homogeneous co-ordinate of $1$.
	![](Pasted%20image%2020260215004628.png)
	These triangles are subsequently rasterised. 

## Rasterisation
- In the rasterisation stage, we perform the process which calculates which pixels on the screen are contained in a triangle that is being rendered and each pixel inside a triangle is called a fragment. 
- There will be multiple fragments tied to one pixel where fragments overlap. This is resolved in pixel processing.
- The rasterisation pseudocode is roughly:
	*For each pixel y in Y dimension*
	*For each pixel x in X dimension*
		*For each triangle t*
		*If pixel x, y is inside triangle t*
			*Produce fragment*
- Determining pixel membership equations to executing the implicit line equation, for naive solutions, for each triangle.

## Pixel Processing
The pixel processing stage is made up of the following substages:
1. Fragment shader
2. Merging

### Fragment Shader
- Rasterisation produces a list of fragments, which are processed by a fragment shader to compute the colour of each fragment. 
- It is common for the fragment shader to sample texture colours or calculate the amount of light reaching a fragment from a light source, which will be covered later. It is further common to interpolate attributes e.g., colour for the fragment based on barycentric co-ordinates, though this is done in the GPU implicitly. 
### Merging
- Only the parts of the triangles visible to the viewer should be displayed. 
- *Merging* combines the fragment colour produced by the fragment with the colours stored in the colour buffer(usually only has a clear/background colour)
	- If blending is enabled, the new fragment colour is mathematically combined with the existing pixel colour e.g., alpha blending. 
	- For the case below, simply replace the existing pixel value if 'visible'
- The below figure shows what happens when a red triangle is rendered first and then a blue triangle rendered second. The actual geometry is shown in the image below that one.
	![](Pasted%20image%2020260215010203.png)
	![](Pasted%20image%2020260215010220.png)
- Visibility is resolved using the $z$ buffer. The $z$ buffer is the same size as the colour buffer, and each element is initialised to $+∞$.
	- If the $z$ value of a new fragment (interpolated via barycentric co-ordinates and sent to the z buffer(it should be noted that fragments arrive one at a time from the rasteriser, that is, this is carried out during the rasteriser loop, not after it, so we continually compared)) is less than the $z$ value currently stored in the $z$ buffer, then the colour in the colour buffer is updated with the fragment colour, and also replace the $z$ value in the $z$ buffer for the pixel
	- If the $z$ value of a new fragment is greater than the $z$ value currently stored in the $z-buffer$, this means the current fragment is behind a fragment which was previously computed, nothing changes in either buffer.
	- ![](Pasted%20image%2020260215011449.png)
