## Texture mapping
- We have seen prior, how to render a triangle with the colours of fragments interpolated from the vertex colour attributes as seen in the below figure.
	![](Pasted%20image%2020260225222743.png)
	Each fragment is coloured by interpolating vertex colours across the triangle, via use of barycentric co-ordinates. 
- Here, we learn about adding more detail to the triangle by mapping a **texture** to the triangle
	> *A texture is an image which can be rendered inside a triangle*.
	
- Instead of interpolating vertex colours, we want to interpolate *texture co-ordinates* so that each fragment can "look up" a colour from an image. 
	![](Pasted%20image%2020260225223129.png)
- A **texel** is a texture element, is essentially a colour. Below we can see individual texels from the texture when a specific region of the texture, drawn in red, is magnified.
	![](Pasted%20image%2020260225223141.png)
- To map a texture onto the triangle, we need a method to specify which texels should be mapped to the fragments. 
- To perform texture mapping, we assign each triangle vertex a UV co-ordinate:
	- $u$ = horizontal co-ordinate on the texture
	- $v$ = vertical co-ordinate on the texture. 
	- Each vertex gets its own $(u, v)$ co-ordinate.
		These values are typically normalised between 0 and 1. 
	![](Pasted%20image%2020260225224345.png)

### Texture Sampling
- *Texture mapping* involves sampling a texture for each fragment on a triangle to retrieve the texel colour from the texture and use that to colour the fragment.
- Let's say we have specified a triangle as a list of vertex attributes consisting of positions, and we want to draw a texture on the triangle. We need to specify UV values/texture-co-ordinates for each vertex in the array. 
	![](Pasted%20image%2020260225225132.png)
- Texture sampling involves 4 steps:
	1. **Project**
	2. **Correspond**
	3. **Sample**
	4. **Transform**

#### Project
- The fragment position is *projected to a fragment UV*.
- When the triangle is rasterised, the GPU generates fragments. For each fragment, it computes barycentric co-ordinates. The same barycentric weights are used to interpolate UVs. 
- $(u_f, v_f) = w_1(u_1, v_1) + w_2(u_2, v_2) + w_3(u_3, v_3)$ 
	![](Pasted%20image%2020260225225547.png)

#### Correspond
-  A UV is in normalised co-ordinates, but the texture image is indexed in pixels, yielding a texel.
- A corresponder function maps the fragment UV to a texel co-ordinate. $$TEXEL = \frac{UVF_{x}*width}{UVF_{y}*height}$$
	![](Pasted%20image%2020260225230852.png)

#### Sample
- Samples the actual texel by ignoring the potentially present fractional parts returned by the corresponder function. 
	- Essentially samples the nearest texel. 
- Reads the texel colour values (RGB) directly from the image data stored in memory. 

#### Transform
- Finally, a transform function converts the retrieved RGB values from the texture to ones that can actually be dsplayed
- For COMP3011, we only store texture in formats that we are able to display directly, so the RGB values are unchanged. 
- Sometimes textures are stored in formats that require conversion e.g., compression formats, but like said prior, we only store displayable RGB.

## Magnification
- Magnification occurs during texture mapping when the number of pixels is greater than the number of texels in a given area, $pixels > texels$.
- Magnification happens when the rendered object covers a large area on screen, but the texture being used has fewer texels for that area, so multiple screen pixels will map to the same texel(s).
	![](Pasted%20image%2020260225235254.png)
- When a texture is sampled, the corresponder function maps the fragment UV to a texel co-ordinate including fractional parts
- The texel is then sampled by dropping the fractional parts and reading the texel values that remain, effectively sampling the nearest texel, which is called *Nearest sampling*.
- Suppose that we map the texture shown in the bellow figure to a quad made from two triangles. The resulting pixellated magnification will occur as shown to the right.
	![](Pasted%20image%2020260225235709.png)
- With nearest sampling, multiple fragments will resolve to the same texel co-ordinate where the texture resolution is not high enough to provide individual texels for each fragment. 
- **Bilinear filtering** is a method for smoothing the pixellation during magnification. The nearest 4 texels are sampled, on either side of the texel co-ordinate including fraction parts, as shown below
	![](Pasted%20image%2020260226000204.png)
	- Do every combination of alteration on the $UVF$ to yield 4 independent texels. 
	- Interpolate in the $x$ direction using the fractional $x$ amount
		![](Pasted%20image%2020260226000440.png)
	- Interpolate in the $y$ direction using the fractional $y$ amount
		![](Pasted%20image%2020260226000448.png)
- The final result is that the image will no longer be pixellated, as we take the weighted average of the 4 closest texels, smoothing the magnification.
	- The smoothing is because bilinear filter essentially turns a step function into a continuous one via linear interpolation - adjacent screen pixels will get slightly different blended colours rather than mapping to the same texel which creates jumps and spikes.

## Minification and Mipmaps
- Minification occurs when the number of texels is greater than the number of pixels in a given area.
- In other words $texels > pixels$ 
- For example if the texture resolution is 1024 x 1024, but the object is far away, in screen space that surface/object may only be 50x50 pixels. Each screen pixel would correspond to (1024/50) ≈ 20 texels per pixel.
	![](Pasted%20image%2020260226001245.png)
- Like just mentioned, if the geometry which is textured moves away from the camera, then the pixels/texels ration would get even smaller. 
- If the geometry which is textured was rotated on the x-axis, rotating the top away from the camera and the bottom toward the camera, then the top fragments would have a very small pixels/texels to ratio, whereas the bottom fragments would have a larger pixels/texels ratio.
- During texture sampling, if only the nearest texel is sampled for each fragment, then a lot of colour information stored in unsampled texels will be lost. This is called **texture aliasing**. 
	- This is solved by sampling all the texels covering a given pixel, and averaging the colour for the fragment.
	- This raises the new problem of how many texels should be sampled given the corresponded UVF onto the texture. 
		- We solve this problem by calculating the following differentials per fragment UVF (mapped to texel space).
			1. The change in $x$ texels for one change in the $u$ texture co-ordinate $∂x/∂u$ 
			2. The change in $y$ texels for one change in the $v$ texture co-ordinate $∂y/∂v$ 
			3. The change in $x$ texels for one change in the $v$ texture co-ordinate $∂x/∂v$ 
			4. The change in $y$ texels for one change in the $u$ texture co-ordinate $∂y/∂u$ 
		![](Pasted%20image%2020260226002709.png)
			Note this is done on the GPU, computed automatically using neighbouring fragments, comparing UVFs and the corresponding change in texels. We do not actually need the original texture, $x = u \cdot \text{texture width}$, $y = v \cdot \text{texture height}$ 
		- We take the maximum differential to represent the square size of the maximum number of texels covered by a fragment, $d = 7.2$ 
			- We take the maximum to ensure we don't underestimate and undersample, causing aliasing. 
- **Mipmaps** is a technique which stores multiple copies of the same image, each half the size of the previous image in the sequence. The sequence continues until the image is 1x1 pixel in size
	![](Pasted%20image%2020260226003342.png)
- We can arrange the mipmaps in a pyramid with the largest at the base, called level 0, and the smallest at the top, called level $n-1$ 
	![](Pasted%20image%2020260226003414.png)
- Mipmaps store downsampled versions of the original texture. 
- When a pixel needs to sample the colour of a texel(s), the $d$ value is compared against the height in the mipmap pyramid using the following formula, and the nearest levels are used in the sampling: $$level_{a} < \log_{2}d < level_{b}$$
- In this example $d = 7.2$ so $\log_{2}7.2 = 2.85$ and the texels are sampled from mipmap levels 2 and 3. 
- Bilinear sampling as described above, samples a colour $ca$ from level $a$, and a colour $cb$ from level $b$. These are then interpolated to acquire the final fragment colour, this is called trilinear sampling.
	- We determine where on the mipmap to sample exactly thesame as the UVf sampling process, just apply to the different-resolution texture. 


### Anisotropic filtering
- ...