## Colour perception
- What humans perceive as surface colour depends on the light reflected from the surface which reaches the eye
- Light is comprised of photons, and 3 different nerve cells at the back of the eye are excited by photons with different wave lengths, corresponding to excitation by red light, green light, and blue light, respectively. 
- We perveive many different colours by mixturres of these primary colours. 
- This is the motivation for using RGB in graphics. 
- Light sources emit light of a given colour according to the wavelengths of the photons. 
- **White light** is made up of equal amounts of all wavelengths. 
- Surface materials - surfaces do not usually emit their own visible light, they absorb some components of incoming light and reflects others according to the colour of the light and the colour of the material.
	- A red surface under a white light appears red because it reflects the red component strongly and suppresses others. 
	- The reflected light/colour is defined as vector component multiplication of light and surface colour denoted $\otimes$:
		![598](Pasted%20image%2020260307165724.png)
		*"What colour is available after the material filters the light"*
- The colour of a fragment is calculated as the combined surface and light colours, which is the light not absorbed by the surface material, scaled by the total illumination $i$ of the fragment.$$fragment\_colour = i \cdot (lightcolour \otimes surface\_colour)$$
	Deliberately distinguish hue/content from brightness/illumination. 


## Phong Lighting
- A *local lighting model* considers the direct illumination from light sources only, rather than considering light bouncing off other surfaces which is global illumination (e.g., path-tracing, ray tracing).
- Tells us *how much light* reaches some geometry.
- The model specifies that the total illumination of a fragment is the sum of ambient, diffuse, and specular components: $$i_{phong} = i_{amb} + i_{diff} + i_{spec}$$
- Phong's model can then be used to colour the fragment:$$fragment\_colour = i \cdot (lightcolour \otimes surface\_colour)$$
- The ambient component accounts for some light bouncing off other surfaces before reaching the one being calculated, and is often approximated using a constant $k$
	- $i_{amb} = k$ 
- The diffuse component $i_{diff}$ is based on Lambert's law, is measures the lighting we get from a matte surface by determining how directly the surface takes the light (which explains why we use $n$ as we need the surface orientation. )
	- $i_{diff} = n \cdot l = \cos \theta$
	- If the light is behind the surface, in practice we take $max(n \cdot l, 0)$ as this would not be meaningful. 
- The specular component $i_{spec}$ models the **highlight** on a shiny surface. A bright highlight appears when the viewer is looking in almost the same direction as the light would reflect, so specular lighting depends on the **viewer position**.
	- $i_{spec} = r\cdot v = \cos p$ 
	- As the angle gets bigger, the highlight weakens as $\cos p$ gets smaller and vice-versa. We essentially measure alignment-with-reflection.
	- The specular component is modified by a shininess parameter $shi$ $$i_{spec} = (r \cdot v)^{shi} = (\cos p)^{shi}$$
		Where $1 \leq shi \leq 256$.
	 - $shi$ controls the sharpness of the highlight - note that the original equation yields a value between $0$ and $1$, exponentiating this makes the resultant value smaller, unless it is exactly 1. 
	 - So perfect alignment stays at full strength regardless of $shi$, points with slightly imperfect alignment get suppressed more strongly with large $shi$, and points with moderate misalignment are crushed toward $0$ for large $shi$, and vice versa.
	 - So small $shi$ reduces values much less, so fairly wide regions around the perfect reflection direction still have noticeable intensity.
![](Pasted%20image%2020260307172312.png)


### Directional Light
- A **directional light** is treated as being infinitely far away.
- The sun is the intuitive example
- We call it directional lighting as for scene-lighting purposes, what matters is its direction, not its position. 
	- All fragments are lit the same way regardless of where they are, provided they have the same orientation relative to the light (discounting the specular component).
		![](Pasted%20image%2020260307174948.png)
- A directional light is defined by:
	1. **An RGB colour vector**
	2. **A direction vector** $l = (l_0, l_1, l_2)$ 
	![](Pasted%20image%2020260307174204.png)
- Phong's model can be used to calculate the total illumination of a fragment by a directional light using the below scheme.
	- Phong's model is calculated as $i_{phong} = i_{amb} + i_{diff} + i_{spec}$
		- The ambient component is set to a constant $k$
			$i_{amb} = k$
		- The diffuse component is the dot product of the normal of the fragment with the inverted light direction from the definition of the light. 
			$i_{diff} = max(n\cdot -l, 0)$
		- The specular component is the dot product of the reflected light and the viewer vector raised to the power of $shi$ 
			- $i_{spec} = max(r \cdot v, 0)^{shi}$

### Positional Light
- A **positional light** has an actual location in the geometric space. It is therefore a measurable distance away from each of the fragments in the scene. 
	- The intuitive example is the light attached to the ceiling in the room you currently reside in - this can be thought of as a positional light. 
		![](Pasted%20image%2020260307175517.png)
- A positional light is defined by:
	1. A colour
	2. A position $light\_pos$ 
	3. Constant attenuation, $att_c$ 
		Doesn't fall off with distance, practically stops the light from becoming absurdly strong at very small distances.
	4. Linear attenuation, $att_l$ 
		Grows proportionally to distance, if distance double, this contribution doubles. 
	5. Quadratic attenuation, $att_q$ 
		Grows as a square of the distance, if the distance doubles, the attentuation grows 4x larger. 
	![](Pasted%20image%2020260307175658.png)
- Phong's model can be used to calculate the total illumination of a fragment by a positional light in a manner similar to calculating the illumination by a directional light. 
	- Phong's model is now calculated as: $$i_{phong} = (i_{amb} + i_{diff} + i_{spec}) *attenuation$$
	- Where attenuation scales the illumination. The illumination of a fragment based on its distance to a light source is called attenuation, and is defined like so: $$attenuation = \frac{1}{att_{c} + (att_{l} * d) + (att_{q}* d^2)}$$
	- Where $d$ is the distance between the fragment and the light source. Since the light source has a position, fragments are lit with varying amounts of illumination according to distance. Attenuation values can be adjusted to achieve different effects in the scene. They act like baseline scaling, gentle distance penalty, and strong long-range penalty, respectively. 
		![](Pasted%20image%2020260307180457.png)
	- Since a positional light has a position instead of a direction, the light direction needs to be calculated using the fragment position $p$ and $light\_pos$ as $l = normalise(light\_pos - p)$ so we can do $i_{diff} = max(n \cdot l, 0)$ 

### Spot Light
- A *spot light* is a positional light which emits light in a cone shape pointing in a direction $spot\_dir$ 
	- The intuitive example is using a torch in a dark forest at night. 
- Any fragments that are inside the cone are directly illuminated by the spot light, and any fragments that are outside of the cone are not. 
- A spot light is defined by:
	1. A colour
	2. A position, $light\_pos$
	3. A direction, $spot\_dir$ 
	4. A cutoff cone parameter, $\phi$ 
	5. Constant attentuation, $att_c$ 
	6. Linear attentuation, $att_l$ 
	7. Quadratic attenuation, $att_q$ 
	![](Pasted%20image%2020260307181335.png)
- To calculate if the fragment at position $p$ is lit by the spot light, we need to calculate the dot product of the vector pointing toward the light with the vector of the direction of the light, which is derived from the middle of the cone. 
- The angle $\phi$ is a half-angle to the midpoint, and is equal for both arms of the cone as they are symmetric. 
- We then check if the computed angle is smaller than the cut-off angle cos $\phi$, which of course is equal for both sides. We take $-l$ as the direction vector to perform cross-product with, as can be seen below. 
	![](Pasted%20image%2020260307213418.png)
- Phong's model can be used to calculate the total illumination of a fragment by a spot light in a manner similar to calculating the total illumination of a fragment by a positional light. The only difference is that if the fragment is not hit by the spotlight according to the cutoff angle calculation, then the calculation is just $i_{phong} = i_{amb} * attenuation$ 