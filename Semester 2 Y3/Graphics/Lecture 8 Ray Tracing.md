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
	
	Note that $0.5$ is added so we shoot through the centre of the pixel. We map to $-1$ and $1$ to yield a symmetric co-ordinate system around the camera's forward direction, so it behaves much like a plane. Further note that the angle of the camera to the top of the image plane, and the forward distance to said plane form a right angle triangle. Thus, via basic trig, the tangent of that angle is:
		$$\tan(θ/2) = \frac{\text{{Half Image Plane Height}}}{\text{Distance to plane}}$$
		In all derivations, we choose the distance to the image plane as $1$, so simplifies to just half the image plane height. 
	So we choose one direction to define the camera's FOV, and the other direction is then determined automatically by the aspect ratio $W/H$, such that the image plane matches the actual screen proportions.
- We normalise $d$ to make it a direction vector, and use the camera position in world space for the origin of the ray $o$. 
	- $d = \frac{d}{\sqrt{d^2_{x} + d^2_{y} + d^2_{z} } }$ - Yields a unit direction vector. 
	- $o$ = $camera$
		Self note: We yield a direction vector for each pixel that is shot from the camera, such that an infinite ray is shot into the world space for each pixel, and determine colour based on $min(t, 0)$ and secondary, recursively generated rays. 

### Ray Triangle Intersection
- To test the intersection of a ray with a triangle, we 
	1. Compute if the ray intersects with a plane on which the triangle lies, and if so, then
	2. Test if the point of intersection is inside the triangle. 

#### Ray intersects with a plane
- **Ray** = $q(t) = o + dt$ 
- We use the equation for a plane to determine ray-plane intersection.
- For a known point on the plane $p$, and a known normal of the plane $N$, all points $p'$ on the plane satisfy: $$\forall p'(p-p') \cdot N = 0$$
	That is, the vector from a known plane point to any other plane point is perpendicular to the normal. Two vectors have a dot product of $0$ if they are perpendicular, yielding this definition.
- The point $q$ at which the ray intersects the plane is $q = o + dt$ where $$t = \frac{(p-o) \cdot N}{d \cdot N}$$
	That is, the numerator has two parts, $p$ and $o$, that is a vector parallel to the plane, and a component perpendicular to the plane, we compute their distance, and project them onto the normal, which equals the perpendicular distance from $o$ to the plane. 
	
	The denominator is the direction unit vector projected onto the normal, extracting the component of the direction that points toward the plane. It tells us how aligned the ray is with the perpendicular-plane direction, and tells us the 'rate of approach'
	
	We can visualise this using the below figure, where the length of hypotenuse edge is the value $t$ we want to compute, and the length of the adjacent edge $a = (p-o)\cdot N$ can be seen. 
		![](Pasted%20image%2020260218023104.png)
#### Point inside a triangle
- Once we know the point $q$ where the ray intersects the plane, then we calculate if that point is inside a triangle whose 3 vertices all lie on the plane. 
	![](Pasted%20image%2020260218023417.png)
- We did the plane test first as it is cheap
- We determine whether the point $q$ is inside a triangle **if it lies on the same side of all three edges**
	- We can achieve this via a cross product and a dot product... CANT BE MESSED


### Light Contribution
- Light will be described in more detail in a further chapter. Here we describe a simple model of light
- **Ambient** - in real life we have light sources that bounce many times before it dies out, a small ambient amount is an approximation of this indirect light that is acquired from reflecting/bouncing off of other surfaces, before reaching the surface we are concerned with. It is often approximated using a constant $k$. 
	$i_{amb} =k$
	$ambient = i_{amb}\cdot surface\_colour$
	Here $i_{amb}$ is the scalar used to approximate the indirect light, and $surface\_colour$ is a vector storing the RGB colour of an illuminated surface.
	We need this, as say in whitted ray tracing, if a surface/triangle that comes into contact with a primary ray generates a shadow ray that intersects with something else in world space other than the light source, there will be zero diffuse contribution, whereas realistically, it should still receive bounced light. 
- The contribution from a light source, called *diffuse* is computed as a term based on **Lambert's Law** which states that the reflected light at point $p$ on a surface is equal to the cosine of the angle between the surface normal $n$ and the vector $l$ from $p$ to the light source, where both $n$ and $l$ are normalised unit vectors. 
	$i_{diff} = n \cdot l = cos \theta$
	$diffuse = i_{diff} \cdot surface\_colour$
	The brightness of a surface depends on how directly it faces the light. 