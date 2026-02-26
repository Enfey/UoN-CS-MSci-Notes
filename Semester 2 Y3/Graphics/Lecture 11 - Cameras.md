## Intro
- Already looked at how to transform vertices relative to a camera in view space using the view matrix in [Lecture 6 - Spaces](Lecture%206%20-%20Spaces.md). 
- A camera is essentially a structured way of generating that view transformation - the camera defines how the world is viewed.
- Without a camera, we cannot meaningfully render a 3D scene as rendering requires transforming world co-ordinates into view space.
- An intuitive and useful camera isn't purely mathematical, but rather an art form. 
- Implemented a 'useful camera' is closely related to the problem of deciding what needs to be viewed whilst considering the viewer.

## Essential Components of aa camera
- For our purposes, a camera consists of:
	1. A 3D position vector in world space
	2. A 3D Forward direction vector
	3. A 3D Right direction vector
	4. A 3D Upward direction vector. 
- We can calculate the camera's direction vectors using Euler angles called **Pitch, Yaw, and Roll**
	- These describe how much the camera is **rotated around its axes**, typically scalars.
	- **Pitch**
		- Rotates the camera on the x-axis. 
	- **Yaw**
		- Rotates the camera on the y-axis
	- **Roll**
		- Rotates the camera on the z-axis. 
	 ![](Pasted%20image%2020260225200643.png)

## Model-Viewer Camera
- The first type of camera we look at is called a *Model-Viewer* camera
	![](Pasted%20image%2020260225200912.png)
- The camera constantly looks at the centre of the model being viewed - do note that this is in world space. 
- It moves on the surface of an imaginary sphere
- The user can modify the camera pitch, which will move the camera up and down along the latitude, around the surface of an imaginary sphere surrounding the model
	- Note, the sphere is not a physical constraint, but rather a result of how the camera position is calculated. 
- The camera can modify the camera yaw, which will move the camera left and right along the longitude, around the surface of an imaginary sphere surrounding the model.
- The user can modify the distance $d$ of the camera from the model forwards and backwards, which changes the radius of the sphere. 
- In this way, the camera can be manipulated to view the model from all angles and distances, whilst focusing at the model's centre point (this is because we do not permit modification of roll).
	![](Pasted%20image%2020260225201608.png)

### Unit Sphere
- The Model-Viewer camera is implemented using a unit sphere. 
- We can extend the algorithm for calculating points on a unit circle so that it calculates points on a unit sphere in 3 dimensions. 
	- Recall that if a point lies on a circle of radius one, then $(x, y)$ = $cos \ \theta$ and $sin \ \theta$, respectively:
		- $sin \ \theta = \frac{Opposite}{Hypotenuse}$ 
		- $\cos \ \theta = \frac{Adjacent}{Hypotenuse}$ 
		- Gives the vertical and horizontal projection of a right-angle triangle, respectively, yielding $(x, y)$.
- We want a point on a sphere of radius 1, with its origin as the model origin.
- We use two angles:
	1. $\theta = Yaw$ - rotation around $y$-axis
	2. $\alpha = Pitch$  - rotation around the $x$-axis
- If we simply imagine the XZ plane, which gives is a circle on the ground (the green circle)
	![](Pasted%20image%2020260225204331.png)
	On that circle $x = cos \ θ$, and $z = sin \ θ$, as it is just the unit circle. 
	This yields an initial unit vector.
- We take our unit vector and tilt it upward by $\alpha$ the pitch (note the red circle describing the YZ plane).
	![](Pasted%20image%2020260225204639.png)
	The vertical height $y = sin \ α$ as per the unit circle definition.
	The horizontal part we had from our prior unit vector is multiplied by $cos \ \alpha$ as this is the horizontal projection of the tilted vector.
- If $p = (p_x, p_y, p_z)$ is a unit vector, the trigonometric functions can be used as follows:
	$p_x  = cos  \ \theta \cdot cos \ \alpha$ 
	$p_y = sin \ \alpha$ 
	$p_z = sin \ \theta \cdot \cos \ \alpha$ 
- As mentioned prior, we use $\alpha = Pitch$ and $\theta = Yaw$, and the camera position is then the calculated point on the unit sphere. 

### Forward, Right, Up Direction vectors
- We have the camera position in world space acquired, but position alone is not enough, it does not tell us which way the camera is facing
- Scale $p = (p_x, p_y, p_z) \cdot d$ 
	![](Pasted%20image%2020260225211225.png)
- **Forward**
	- Because this is a model-viewer camera, we are always looking at the model-space origin. 
		Computed as $Forward = normalise (Origin-Position)$ 
		- For example $(0,0,0) - (0, 5, 0) = (0, -5, 0) = (-1, 0, 0)$, the camera points straight downward and is above the model.
- **Right**
	- The right direction vector of the camera is the vector which is the result of cross product of camera forward with the second standard basis vector $(0, 1, 0)$, perpendicular, must be ordered in this way otherwise would yield perpendicular left. 
- **Up**
	- The up direction vector of the camera is the vector which is the result of cross product of camera Right with camera Forwards.
	![](Pasted%20image%2020260225211847.png)
- These values are then used for the view matrix, so that the world space object(s) are rendered correctly according to camera position and direction. 
## Fly-Through Camera
- A **Fly-Through** camera has: 
	- A **position** in world space
	- A **forward direction**
	![](Pasted%20image%2020260225213454.png)
- May or may not be pointing at the model which is being viewed; it does not have to look at the origin, it can freely move through the scene. 
- It is akin to flying through a 3D world! 
- The user can modify the Yaw and Pitch to alter the forward direction.
- The user can move the camera forwards, backwards, left, and right which are all translations relative to the forward direction. 
- In this way, the camera can be manipulated to fly through the scene in all directions and look in any direction. 
	![](Pasted%20image%2020260225213642.png)
- The Fly-Through camera is also implemented using the unit sphere. However, the camera position is not on the sphere. 
- The unit sphere is used to compute the forward direction, with the camera position as the origin of the unit sphere.
- The forward direction vector is the point on the unit sphere calculated using Pitch and Yaw. 
	![](Pasted%20image%2020260225213902.png)
	XZ, YZ plane etc. See prior notes. 
	Since it is a unit sphere, this point is also a unit direction vector requiring no normalisation.
- The right direction of the camera is the vector which is the result of the cross product of the second standard basis $(0, 1, 0)$ and the forward direction vector.
	![](Pasted%20image%2020260225214039.png)
- The up direction of the camera is the vector which is the result of the cross product of the camera right with the camera forward. 
	![](Pasted%20image%2020260225214045.png)
- Changing the position of the camera to be moved forward or backward in world space is translated by adding camera forward or subtracting camera forward.
	- $Position = Position + Forward$ 
	- $Position = Position - Forward$ 
- The camera can be translated right and left by adding camera Right or subtracting camera Right, respectively.
	- $Position = Position + Right$ 
	- $Position = Position - Right$
- In this way we can calculate the camera position, Forward, Right, and Up components using the camera Yaw and Pitch.
	- Note that the initial camera position is arbitrary and chosen by the developer. 
	- For each frame in a fly through camera
		- Update yaw and pitch from mouse
		- Recompute forward using unit sphere formula
		- Recompute right
		- Recompute up
		- If movement keys are pressed, then update the position. Note that this is movement along whatever the forward and right vectors contain; in something like an FPS, looking up would not lift you (y determined via offset from ground) whereas in something like blender, it would. 
		- Build the view matrix and render. 
