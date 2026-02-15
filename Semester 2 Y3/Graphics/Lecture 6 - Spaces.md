## Projecting from 3D to 2D
- 3D rendering is the process of drawing 3D images on a 2D screen.
- 3D renedering requires vertices, a camera with location and direction, and a display to draw assembled triangles which are visible to the camera. 
	![](Pasted%20image%2020260214185615.png)
- A vertex does not go directly from 3D to 2D, it passes through a series of co-ordinate systems ("Spaces") each with a specific purpose, keeping modeling, scene layout, screen mapping etc clearly separated. This is called **projection**.
 - 3D space has 3 dimensions; a position in this space is specified with a cartesian co-ordinate  $p=(x, y, z)$ which is a vector in $\mathbb{R}^3$ 
 - A display has 2 dimensions; a position in this space is specified with a cartesian co-ordinate $p=(x, y)$ which is a vector in $\mathbb{R}^2$ 
 - Therefore, an important part of rendering is **projection** from 3D to 2D.

## Spaces
- In order to project from 3D to 2D, vertices are transformed through 6 spaces.
	![](Pasted%20image%2020260214190059.png)
	1. **Model Space**
	2. **World Space**
	3. **View Space**
	4. **Clip Space**
	5. **Normalised Device Co-ordinate Space**
	6. **Screen Space**

### Transformations between Spaces
- We transform vertices between the 6 spaces using transformations
	![](Pasted%20image%2020260214190718.png)
	1. Model transformation of vertices from model space to world space. This is achieved using a **model matrix**.
	2. View transformations of vertices from world space to view space. This is achieved using a **view space**
	3. Projection transformations of vertices from view space to clip space. This is achieved using a **projection matrix.** 
	4. Perspective division of vertices from clip space to NDC space. This is achieved by dividing each component by its homogeneous $w$ component, termed **perspective division**.
	5. Viewport transformation of normalised co-ordinates to screen pixel co-ordinates. 
### Model Space
- *Model space*, in 3D, is relative to an individual model constructed from vertices. 
- You may imagine thousands of vertices positioned relative to the centre of an object (where the origin lies) of an object, which can be used to construct a mesh of triangles denoting the object surface. 
- There is no information about where an object is relative to anything in the world, there is only information about the relative positions of vertices which make up the object. 
	![](Pasted%20image%2020260214191841.png)
	Where the origin is at the centre, the position vector for the back left vertex essentially describes left, up, and backward.
- Define object independently of the world (can reuse in many different positions in world space).

### World Space
- World space is a shared co-ordinate system for the entire scene. 
- Positions all the models relative to each other and also positions a camera. The origin of world space can be anywhere. 
- To move from model space to world space, we use the **model matrix**
	The model matrix combines 
	- Translation
	- Rotation
	- Scaling
- For example, if the cube's center should be at (5, 0, 7) in the world space, its model matrix should be composed (quite literally) in a manner to reflect that transformation as a translation. The model matrix will then shift **every vertex** by that amount. 

#### Example
- In the below figure, two cubes are shown at their positions in world space, where the origin of each model space is at that model's position. A camera is shown at its position which will be used in the next space.
- Translation, scale, rotation, achieved via independent model matrices, which transform all the vertices specifying that model.
- In the figure, the green cube is at position $(5, 0, 7)$ and is not rotated or scaled, so the model matrix for the green cube is simply: $$M = \begin{pmatrix} 1 & 0 & 0 & 5 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 7 \\ 0 & 0 & 0 & 1 \end{pmatrix}$$
- As an example, the top back left vertex $v1_{model}$ is multiplied by the model matrix $M$:
$$v_1^{world} = M \times v_1^{model} = \begin{pmatrix} 1 & 0 & 0 & 5 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 7 \\ 0 & 0 & 0 & 1 \end{pmatrix} \times \begin{pmatrix} -1 \\ 1 \\ -1 \\ 1 \end{pmatrix}$$
$$= \begin{pmatrix} -1 \begin{pmatrix} 1 \\ 0 \\ 0 \\ 0 \end{pmatrix} + 1 \begin{pmatrix} 0 \\ 1 \\ 0 \\ 0 \end{pmatrix} + (-1) \begin{pmatrix} 0 \\ 0 \\ 1 \\ 0 \end{pmatrix} + 1 \begin{pmatrix} 5 \\ 0 \\ 7 \\ 1 \end{pmatrix} \end{pmatrix}$$

$$= \begin{pmatrix} \begin{pmatrix} -1 \\ 0 \\ 0 \\ 0 \end{pmatrix} + \begin{pmatrix} 0 \\ 1 \\ 0 \\ 0 \end{pmatrix} + \begin{pmatrix} 0 \\ 0 \\ -1 \\ 0 \end{pmatrix} + \begin{pmatrix} 5 \\ 0 \\ 7 \\ 1 \end{pmatrix} \end{pmatrix} = \begin{pmatrix} 4 \\ 1 \\ 6 \\ 1 \end{pmatrix}$$

![](Pasted%20image%2020260214193554.png)

### View Space
- The *view space* has each vertex of an object at a position from the point of view of the camera. We express the world from the position of the camera. 
	"*Where is each object relative to the camera*"
- The origin of the view space is the position is position of the camera, and the camera looks down/points towards the negative $Z-axis$. 
- The translation and rotation of each object from world space to view space is achieved using a **view matrix**, applied to all vertices specifying each model in the scene. 

#### Example
![](Pasted%20image%2020260214203059.png)
- The above figure shows the 2 cubes in view space, with the camera at the origin pointing down the $-Z$ axis, i.e., away from from us into the page, and the camera is also shown as the origin.
- The **view matrix** (denoted by $V$), configures the camera’s position, orientation, and perspective within the global scene. By applying the view matrix, we effectively convert the coordinates of objects’ vertices from world space to view space, establishing the desired viewpoint for our 3D scene.
- We derive the view matrix often by treating the camera as an ordinary object in world space, and then using the view matrix to define all objects relative to the camera; the origin of this co-ordinate space.the
- We apply the same matrix to all vertices in the world space.
- In world space, the camera was positioned at $(5,5,5)$ and was rotated clockwise $-90^{\circ}$ (we look down, as can see in world space diagram).
- Translation needs to be negated as moving the camera forwards is the same as moving all the models backwards, we essentially compute the relative distance from the camera as origin to all the models in world space, and this is reflected in our matrix. 
- Rotation also needs to be negated, for the obvious reason that rotating the camera in one direction is equivalent to rotating the entire world in the opposite direction when expressing coordinates relative to the camera.
	- If the camera rotates by $\theta$, then expressing the world relative to that camera requires rotating the world by $-\theta$.
	- If the camera turns to the right, the world must appear to turn to the left to keep the camera aligned with the standard view-space axes.

#### View Frustrum
- The below figure displays the cubes in the view space, with the camera at the origin pointing down the -Z axis, and a frustrum, everything outside the frustrum will not be rendered. 
- Notice that the frustrum has a smaller face near the camera, and a larger face further than the camera. 
- The frustrum will be transformed into a cube, and during that process perspective will be added to all the contents of the frustrum. A frustrum is defined using left $l$, right $r$, top $t$, bottom $b$, near $n$, and far $f$ values.
	![](Pasted%20image%2020260214222459.png)
	
### Clip Space
- The transformation from view space to clip space is achieved by using a projection matrix. 
- The intuition for the name is that anything outside the view frustrum is clipped using these rules. 
- After projection, each vertex becomes a 4D co-ordinate. 
	The rules that determine whether a vector is outside the view frustrum are:
		$-w ≤ x ≤ w$
		$-w ≤ y ≤ w$
		$-w ≤ z ≤ w$
- Up to this point the homogeneous component $w$ for a position vector has remained = 1, the -1 in the projection matrix's 3rd column vector, after being applied to a view space vector, yields $w' = -z$, so $w$ now depends on depth. During perspective division, we can enforce shrinking with distance (far objects get large $w$.)
	![](Pasted%20image%2020260214223345.png)
	


### Normalised Device Co-ordinate Space
- We take 4D embedded co-ordinates and perform perspective division based on the $w$ value derived from the perspective matrix applied to all vertices in the clip space. 
- We yield 3D co-ordinates from this, where $w = 1$ again, implicitly
- We divide each vector's component by its $w$ value, which is just the inverse of $z$ prior to applying the projection matrix. 
- After this division, co-ordinates are inside a cube. The origin of NDC space is the centre of this cube. We essentially transform the frustrum's contents to an NDC cube which contains what will be rendered.
- Notice that in the NDC space below, the red and green cubes now have perspective, and the far faces of the cubes are smaller than the near faces. 
	![](Pasted%20image%2020260214224114.png)

### Screen space
- Screen space corresponds to pixels on the screen. The vertices in NDC space are transformed to screen space using the **viewport transformation.** 
- Viewport transformation uses the following formula, where $width$ and $height$ are the dimensions in pixels of the display.
	$\mathbf{v}_{\text{SCREEN}} = \begin{pmatrix} (NDC_x + 1.0) \cdot (width/2) \\ (NDC_y + 1.0) \cdot (height/2) \end{pmatrix}$ 
- We don't directly use $z$ in the equation; we only use $x$ and $y$ while $z$ is kept aside for depth testing to detect which fragment is visible. By the time we reach screen space, NDC via perspective projection has converted the 3D clip space into a correct 2D projection with depth attached via perspective division, so the full co-ordinates are not needed.
- The vertex $\mathbf{v}_{1}^{NDC} = (-0.5, -0.5, 0.5)$ is transformed to screen space as follows:
	$\mathbf{v}_{1}^{screen} = \begin{pmatrix} (-0.5 + 1.0) \cdot (300/2) \\ (-0.5 + 1.0) \cdot (200/2) \end{pmatrix} = \begin{pmatrix} 75 \\ 50 \end{pmatrix}$
	![](Pasted%20image%2020260214225253.png)
	
