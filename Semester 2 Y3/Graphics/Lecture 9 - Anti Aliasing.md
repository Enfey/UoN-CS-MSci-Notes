# Aliasing
- Rendering essentially boils down to sampling. 
- A display is a finite array of width by height pixels, and each pixel needs to sample infinite resolution geometry to decide the colour that should be displayed. 
	- Geometry is infinite resolution; it exists initially in continuous 3D space; even in rendering pipelines where projection to screen space occurs, those co-ordinates are still continuous, not discrete like pixel values. 
	- The pixel grid samples such continuous geometry. 
- **Aliasing** is an undesirable effect of the frequency chosen to take samples. Aliasing happens when samples are taken at a frequency which is too low. 
- Pixels whose sample point lies inside the triangle get triangle colour, otherwise they get the background, as seen below, this produces a coarse approximation to the true edges of the triangle.
	![](Pasted%20image%2020260218033845.png)
	![](Pasted%20image%2020260218033850.png)
- If the samples are only taken at the pixel centre points, then the resulting colours will be shown as in Figure 2. Clearly, information is lost in the sampling proceedure. There are parts of the triangle which are clearly not sampled. 
- This aliasing is often called *jaggies*. We now cover methods for reducing the effects of aliasing, which are called antialiasing techniques. 


## Supersampling
- Supersampling, also called Supersampling Antialiasing SSAA, is the most obvious idea for how to reduce aliasing.
- If aliasing happens because there are too few samples, then it is a natural idea to perform more samples.
- Supersampling can be parameterised by the number of samples taken in each dimension for one pixel. 
	- 2x SSAA generates twice as many fragments in both the x and y screen dimensions, 4x supersampling generates four times as many fragments in both the x and y screen dimensions, and so on and so forth. 
		![](Pasted%20image%2020260218034827.png)
		SSAA 2x
- Supersampling works by sampling the fragment colour using the fragment shader, and then storing each colour in a colour buffer which is larger than the display size. When the pixels are displayed, the colour of each pixel is the average colour of the samples that make up the colour buffer. 
	- The fragment shader is ran per sample now, so say for SSAA 4x, the shading work is 16x greater for a given pixel(although, there could be more than 16 fragment evaluations per pixel, and the z buffer would be increased to account for this visibility testing, there may even be multiple fragment shader calls per subpixel if overdraw exists).
- ![](Pasted%20image%2020260218035227.png)
	The above figure, shows 4 pixels with the centre points in red and a triangle that is being rendered, each pixel has 4 samples/subpixels taken using the fragment shader as shown by the points in yellow. The middle  image shows the samples that are inside the triangle and are shaded in blue. Each pixel colour is then calculated as the average of the 4 samples for that pixel. The right image shows that the top-right pixel is 25% blue, and the bottom-right pixel is 75% blue. 
- Supersampling has sometimes been provided as a hardware solution, however, supersampling is extremely expensive to compute because all samples are coloured by the return value of the fragment shader for that given sample/subpixel. 

### Pseudocode
```
for each sample y in Y BUFFER_SIZE
{
	for each sample x in X BUFFER_SIZE
	{
		if sample x, y is inside triangle t
		{
			calculate the fragment colour via the shader
			copy fragment_collour to colour_buffer
		}
	}
}
AveragePixelColours()
```

```
AveragePixelColours()
{
	d = BUFFER_SIZE/DISPLAY_SIZE // BUFFER_SIZE depends on SSAA parameterisation, for 2X, d would be 2. 
	for each pixel y in Y DISPLAY_SIZE
	{
		for each pixel x in X DISPLAY_SIZE
		{
			for each sample dy < d 
			{
				for each sample dx < d
				{
					pixel x, y += colour_buffer[(x * d) +
					dx] [(y * d) + dy]
				}
			}
		}
		pixel x, y /= (d * d) // Smoothing, get the average.
	}
}
```



## Multisampling
- Multisampling, known as Multisampling Antialiasing MSAA, attempts to utilise the increased number of samples whilst reducing the fragment shader calls
- In Supersampling, the fragment shader is called one time for each sample that is inside the triangle
- In multisampling, the fragment shader is only called one time for each pixel that is inside the triangle, and the colour is copied to all sample points that are inside the triangle.
	![](Pasted%20image%2020260218042053.png)
	

### Pseudocode
$d = \frac{BUFFER\_WIDTH}{DISPLAY\_WIDTH}$ 
```
d = BUFFER_SIZE / DISPLAY SIZE
subpixel_size = 1/d
for each pixel y in Y DISPLAY_SIZE
{
	for each pixel x in X DISPLAY_SIZE 
	{
		first_sample = true
		for each of d subpixels sy
		{
			for each of d subpixels sx
			{
				if sample point x+sx, y+sy is inside
				triangle t
				{
					if first_sample
					{
						calculate the fragment_colour
						set first_sample to false
					}
					copy fragment_colour to colour_buffer
				}
				sx += subpixel_size
			}
			sy += subpixel_size
		}
	}
}
AveragePixelColours()
```
### Temporal antialiasing
- Temporal Antialiasing *TAA* is an antialiasing algorithm which has been increasingly used. 
- It works much the same way as super sampling. 
- It is essentially supersampling performed across time instead of space; instead of taking may subpixel samples in one frame, we take one slightly offset sample per frame and accumulate results over multiple frames. 
- Over several frames, each pixel effectively receives multiple subpixel samples, approximating supersampling. 
	![](Pasted%20image%2020260218204322.png)
- With TAA, all the samples for each pixel are not generated by the fragment shader every frame, instead, take one sample per frame, and therefore requires multiple frames to take the required number of samples per pixel.
	- Iterative/progressive refinement across frames (one full pass of the rendering pipeline producing a complete image).
	- We could term this as amortising the fragment shader cost across frames. 
- We take one jittered sample per pixel, and blend it with the accumulated history, and display the blended result per frame, converging toward the supersampled result after enough frames.
	![](Pasted%20image%2020260218210011.png)
- This means that the number of samples is 1 pixel per frame, which is the same as if no anti-aliasing is used. This means there is the antialiasing effect of Supersampling but without the extra time requirement as it is stretched across frames. 
- The downside of TAA is that there are 'ghost artifacts' if the geometry within the co-ordinate space(this matters for screen space/how close to camera) moves fast, or the camera pans quickly.
	- This is because TAA blends the current frame with the samples taken at previous frame, if something moves fast/significantly, the pixel in frame $t$ doe not represent the same surface point as in frame $t-1$, but TAA blends regardless. 
		![](Pasted%20image%2020260218205915.png)
- For a static scene, each frame contributes a new subpixel sample, the history buffer accumulates these, and after enough frames, the image converges toward the same result as 4x SSAA. 


## Comparison of Antialiasing algorithms

|     | NO AA       | SSAA        | MSAA                         | TAA              |
| --- | ----------- | ----------- | ---------------------------- | ---------------- |
| Pro | No overhead | Best result | Cheaper than SSAA            | Minimal overhead |
| Con | Jaggies     | High cost   | Still some overhead(copying) | Ghosting         |
