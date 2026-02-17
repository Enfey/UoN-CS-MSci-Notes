# Ray Tracing
- Ray tracing is an alternative rendering paradigm to rasterisation, which is closely related to the physics of light. 
	- Rasterisation = Vertex processing, primitive assembly, rasterisation, fragment processing, depth testing.
- Ray tracing uses the mathematical definition of a ray and allows rays to originate anywhere(usually from camera into the scene), and have any direction. 
- Lighting emerges from ray interactions. 
- It should be noted that ray tracing operates in **world space**, 

# Whitted Ray Tracing
-  Recursive **lighting model** that explictly handles direct lighting via shadow rays, perfect mirror reflections, perfect refractions(transparent materials), and hard shadows.
- It does not simulate full global illumination. 
- A brief overview:
	- Calculating the colour of a pixel is achieved by casting a primary ray, shown as a dashed black arrow, originating at the camera position, passing through a pixel
	- The ray tracing algorithm calculates the closest intersecting triangle.
	- A new ray is generated at the intersection point to determine if the intersected point on the triangle is in shadow, which is shown using a dashed orange arrow. 
	- An additional ray is generated at the same intersection, with a direction reflected over the triangle normal, $N$.
	- The reflection ray is used to calculated reflected colours, shown using a dashed green arrow.
	- An additional ray may also be refracted from the same point, which isn't shown. 
	- These secondary rays for shadows, reflections, and refractions, can be recursively generated at each closest intersection with geometry up to a maximum depth, at which point the accumulated ray colour along the ray is used for the pixel
## Pseudocode
```
rayTraceImage()
{
	for each pixel_y in PIXEL_H
	{
		for each pixel_x in PIXEL_W 
		{
			ray = GetRayDirection for pixel_x, pixel_y
			colour of pixel = trace(ray)
		}
	}
}
```

```
trace(ray)
{
	for each tri in triangles
	{
		t = RayTriangleIntersection(tri)
		save closest_triangle
	}
	return shade(closest_triangle)
}
```

```
shade(triangle)
{
	colour = small ambient amount 
	for each L in lights
	{
		trace (shadow ray to L)
		if no intersection
			colour += diffuse light contribution
	}
	colour += trace(reflection ray)
	colour += trace(refraction ray)
	return colour
}
```

## Whitted Ray Tracing
- A ray is defined as all points $q$ satisfying $$q(t) = o + dt$$
	Where $o$ is the ray origin, $d$ is the ray direction(normalised), and $t$ is a scalar measuring the distance along the ray. We solve for $t$ when testing intersection. 
- The below shows the calculation of a ray direction through a pixel
	![](Pasted%20image%2020260217200308.png)
- We generate a vector $d$ pointing to each pixel defined by $x$ and $y$ pixel co-ordinates via this formula $$d = af\left(\frac{2(x+0.5)}{W}-1\right)R 
  + f\left(\frac{2(y+0.5)}{H}-1\right)U + F$$
	Where $a$ is the aspect ratio $a = W/H$, $f = tan(𝜃/2)$ is a function of the vertical field of view $𝜃$ in radians, and $W$ and H$ are the display dimensions. The vectors $R, U$ and $F$ are the Right, Up, and Forward direction vectors of the camera. As can be seen, Right and Up are scaled and added to the camera forward direction in a linear combination to make the direction of the pixel ray.
	
	Note that 0.5 is added so we shoot through the centre of the pixel. We map to -1 and 1 to yield a symmetric co-ordinate system around the camera's forward direction, so it behaves much like a plane. Further note that the angle of the camera to the top of the image plane, and the forward distance to said plane form a right angle triangle. Thus, via basic trig, the tangent of that angle is:
		$$\tan(θ/2) = \frac{\text{{Half Image Plane Height}}}{\text{Distance to plane}}$$
		In all derivations, we choose the distance to the image plane as $1$, so simplifies to just half the image plane height. 
	So we choose one direction to define the camera's FOV, and the other direction is then determined automatically by the aspect ratio $W/H$, such that the image plane matches the actual screen proportions.
- We normalise $d$ to make it a direction vector, and use the camera position in world space for the origin of the ray $o$. 
 



- To calculate the colour of a single pixel.
	1. Cast a primary ray from the origin (camera) to a pixel
		- A ray is defined as all points $q$ satisfying $q(t) = o + dt$ where $o$ is the ray origin, $d$ is the ray direction, and $t$ is a scalar measuring the distance along the ray, assuming that $d$ is normalised. We solve for $t$ when testing intersection. 
		- We test this ray against all triangles (rays are cast per pixel), and find $t_{closest} = min(t > 0)$ (pick smallest possible $t$ given parametric equation)
			- This yields an intersection point $p$, a surface normal $n$, and material properties. 
			- If nothing is hit, simply return the background colour. 













Calculating the colour of a pixel is achieved by testing a primary ray, shown below as a dashed black arrow.
	![](Pasted%20image%2020260217192634.png)
- The ray tracing algorithm calculates the closest intersecting triangle
	- This is called a primary ray.
- A new ray, shown in dashed yellow, is generated at the intersection point and points toward the light source to calculate if the point on the triangle is in shadow/can reach the light source without collision.
	- This is called a shadow ray. 
- An additional ray is generated at the same intersection point with a direction reflected over the triangle normal, $N$
- The reflection ray is tested to calculate reflected colours, shown using a dashed green arrow in Figure 1. An additional ray can also be refracted from the same point, which isn’t shown. These secondary rays for shadows, reflections and refractions can be recursively generated at each closest intersection with geometry, up to a maximum depth, at which point the accumulated ray colour along the ray is used for the pixel. This is known as a Whitted Ray Tracer [2
- 