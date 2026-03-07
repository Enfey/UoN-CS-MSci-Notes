## Rendering equation
- Considered the most important equation in Computer Graphics:
$$L_{o}(X, \hat{\omega}_{o}) = E_{o}(X, \hat{\omega_{o}}) + \int^\Omega_{\hat{\omega}_{i}} L_{i}(X, \hat{\omega_{i}}) \ f_{X}(\hat{\omega}_{i}, \hat{\omega_{o}}) \ |\hat{\omega_{i}} \cdot  \hat{n}| \ d \hat{\omega_{i}}$$
- Here $X$ is a point on a surface where the camera ray hits according to determined properties, $\hat{\omega}_o$ is a direction vector for light travelling from $X$ toward the camera/viewer, $\hat{n}$ is the surface normal at $X$, telling us the local orientation of the surface, $\Omega$ represents all directions from which light could arrive at $X$ and is a hemisphere, $\hat{\omega}_i$ is a particular direction from which light comes from, $X'$ is a possible place that incoming light may have come from.
	![](Pasted%20image%2020260306230517.png)
- $L(X, \hat{\omega}_o)$ term is the outgoing light from the point $X$ in direction $\hat{\omega}_o$ .
- The $E_o(X, \hat{\omega}_o)$ term is the emitted light from the point $X$ in direction $\hat{\omega}_o$, i.e., it is nonzero if and only if the surface itself is not a light source. 
- The $L_i(X, \hat{\omega}_i)$ term is the incoming light from one particular direction $\hat{\omega}_i$ toward the point $X$, usually coming from some other point $X'$ (this term can be computed as the outgoing light for another point $L_o(X', \hat{\omega '}_o)$)
- The $f_X(\hat{\omega}_i, \hat{\omega}_o)$ term is a material term that defines how much light from direction $\hat{\omega}_i$ contributes to direction $\hat{\omega}_o$ based on the material. This is the surface's scattering law. 
	- For example, a mirror is very selective, if light comes in from an "incorrect" direction, none of it reaches the viewer.
- The $|\hat{\omega_{i}} \cdot  \hat{n}|$ term is the lambert term; the angle factor, incoming light exactly across the normal yields maximum contribution, incoming light below the surface is not counted at all with regard to $\Omega$ etc. 
- The integral simply states that that we perform that portion of the rendering equation for every incoming direction in the hemisphere and sum all contributions. 

### Costs of the rendering equation
- At one point $X$ there are many possible incoming directions in the hemisphere $\Omega$.
- The rendering equation can be solved by calculating recursively, the outgoing light, for all incoming light directions, at each intersection with geometry in a scene. It is extremely expensive to solve since each intersection point has many incoming light directions, and each incoming ray direction may intersect with geometry recursively.


## Path Tracing
- The rendering equation warrants the summation of contribution from all incoming directions according to $\Omega$, but this is far too expensive; the recursive nature and subsequent explosion in scenes with anything more than simple geometry and lighting is much to difficult to compute in a reasonable amount of time. 
- **Path tracing** reduces the huge tree of probabilities that naively following the rendering equation would yield.
	- We create a primary ray passing through a pixel and if it intersects with geometry, pick a random ray from the hemisphere at the intersection point, then recursively follow that ray as it bounces, each time generating one new random ray, until it reaches a light source, at which point the light along that single path is accumulated back to the pixel.
	- This process of following a single path of light repeats a number of times for a given pixel, sampling hemisphere directions at random
	- All the light accumulated over all of the paths can be averaged for that pixel. 

### Path Tracing Pseudocode
```C
pathTraceImage ()
{
	for each pixel_y in PIXEL_Y 
	{
		for each pixel_x in PIXEL_W
		{
			for ray in SAMPLES_PER_PIXEL
			{
				ray = GetPixelRandomRay(pixel_x, pixel_y)
				color += trace_path (ray, 0)
			}
			colour /= SAMPLES_PER_PIXEL
		}
	}
}
```

```C
trace_path(ray, depth)
{
	if depth == path_length_limit
		return (0,0,0)
	for each tri in triangles
	{
		t = RayTriangleIntersection(tri)
		save closest_triangle
	}
	if no closest_triangle
		return sky_colour
	else
	{
		new_ray = GetHemisphereRandomRay(closest_triangle)
		closest_triangle_colour *= trace_path(new_ray,
		depth+1)
		return closest_triangle_colour
	}
}
```

Note that we jitter randomly through the pixel grid into the scene, and do not always shoot through the center, this can be scene in the first function. The `*=` in `trace_path` is for the material interaction.
We return a colour tuple representing radiance/light from `trace_path` much like an RGB vector. 



### Path Tracing Effects
#### Shadow Penumbra
- A **penumbra** is the soft transition region at the edge of a shadow.
- In the below figure, the path-traced result has a shadow that is hard-edged near the door way and softer edged farther away; path-tracing yielding accurate shadow penumbra. 
	 ![](Pasted%20image%2020260307000506.png)
- Path tracing naturally samples many possible light paths, some sampled paths from a surface point which reach visible parts of the light, whilst others will be blocked, and averaging these samples gives a partial-light result, which shows up visually as a soft shadow boundary. 
	- Whitted ray-tracing on the other hand treats shadowing in binary manner, either the light source is visible, or it is blocked, giving hard edged shadows. 

#### Colour bouncing
- By following paths with multiple bounces, those paths wihich are sampled contribute their materials and light recursively, modifying light.
- The lighting contribution returned will be representative of the colkour and material of other surfaces, and if the stochastic hemisphere rays hit many surfaces with the same colour, the point $X$will be tinged/'bleed' that colour. 

#### Ambient Occlusion
- The effect where creases, corners, and cavities look darker because they are less exposed/less able to be exposed to surrounding light.
- The rasterised car on the right uses a constant ambient term, while the path-traced result on the left shows extra detail due to ambient occlusion. 
- Take for example, two points on an object, one on an open, exposed surface and one deep in a crevice, the open point can see a large portion of the surrounding hemisphere, so many directions are available for light to arrive from, whereas the crevice point can see much less of that hemisphere, so the crevice point receives less indirect light and looks darker. 


