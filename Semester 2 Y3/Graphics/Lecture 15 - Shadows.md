## Intro
- Shadows are an important element in computer graphics due to the improvement they make in the correct perception of a given scene.
	![](Pasted%20image%2020260308205722.png)

## Surfaces in Shadow
- A shadow is cast when a physical object blocks the light rays emitted from a light source.
- Blocked geometry will not be directly illuminated, and will instead be in shadow. 
	![](Pasted%20image%2020260308205836.png)
	A directional light illuminating geometry with shadows cast by the geometry blocks the light rays emitted by that light. Recall that a directional light is defined solely by a colour and a direction toward which the light is facing. 
- Mathematically, the surface point which is closest to a light source along a ray of light emitted by a light source is illuminated by that light.
	- All other surface points along that ray of light are in shadow, as can be seen by the black surface points. 
		![](Pasted%20image%2020260308210109.png)
- This can be calculated by computing intersection points of all visible geometry with a light ray, and only illuminating the fragment which is closest to the light source. 
- This is effectively how ray-traced shadows work:
	- Camera shoots primary rays and finds closest intersection point $P$
	- A secondary shadow ray is generated at $P$ and its cast from $P$ in the reverse direction of the light to determine whether $P$ is directly illuminated. 
	- The shadow ray travels from the surface point to the light source, and if it intersects with geometry, then P is in shadow, and there is no direct illumination from this light (though there may be lighting contribution from other secondary rays e.g., reflection and refraction rays and ambient).

## Shadow Mapping
- Two-pass rasterisation technique that approximates ray-traced shadows without casting any rays.
- The idea is that if we render the scene from the light's point of view, any surface that was visible from the light is illuminated, and any surface that was hidden from the light is in shadow.
	- This corresponds directly to the post-merging **z-buffer** used to resolve visibility and shade fragments correctly. 
	- After merging the z-buffer holds the z-values of fragments which are visible i.e., closest to the camera. 
- We maintain two entirely separate co-ordinate systems: camera space and light space, both achieved by applying corresponding matrices to vertice's world positions.
- We render the scene from the light's point of view, and keep only the resulting z-buffer (distinct from the camera z-buffer).
- We call this buffer the **depth map**, which stores the depth of fragments which are closest to the light source and therefore are illuminated. This can be seen below, where we can imagine rendering the geometry from the position of the light source. 
	![](Pasted%20image%2020260309004339.png)
- Closer to light = smaller depth, value near 0.
- Further from light = larger depth, value near 1.

## Light space transformation matrix
- To render geometry from the point of view of the light, we need a transformation matrix.
- During rendering, we normally construct a model matrix to transform a point to world space, a view matrix to transform a world--space point to view space, and finally a project matrix. These 3 matrices are often combined to project a point in the rendering pipeline in the vertex shader. 
- If we replace the view matrix with a light matrix, to transform a point to light space, then we have a combined light space transformation matrix. 
- In the below figure, point $p$ is the fragment being shaded. We can calculate if $p$ is illuminated or not by first transforming $p$ to light space using the light space transformation matrix. 
- The $z$ value of the transformed fragment is the distance to the light in light space. 
	![](Pasted%20image%2020260309005047.png)
- The $x$ and $y$ co-ordinates of the transformed fragment are the co-ordinates used to reference the corresponding depth value in the depth map.
	- Simply compare incoming $z$ values, anything that is smaller replaces, and is illuminated. 
