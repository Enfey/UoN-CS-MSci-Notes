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

### Depth Map

## Light space transformation matrix