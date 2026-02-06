100% CW

## Modern Real-time graphics
- 2022, achieved photorealistic quality in real-time applications
- **Physically based lighting** - accurate simulation of how light interacts with surfaces
- **Dynamic shadows** real time shadow calculations that respond to moving lights and objects
- **Particle systems**
- **Volumetric clouds**
- **PBR** - realistic materials using albedo, metallic, roughness
- **Complex geometry** millions of polygons rendered per frame
- **Post processing effects** - bloom, depth of field
- **Multiple interactive objects** 
- Etc

### Three eras of graphics programming
#### 1990s
- OpenGL became the standard cross-platform graphics API
- Per vertex shading, point in 3d space, shading = figuring out colour based on light, so calculate the colour/brightness per vertex, and fill in the colours between corners by blending
- This was limiting, imagine a square wall lit by a spotilight in the center, the computer calculates light only at the 4 corners, the center of the wall gets the blended corner. The spotlight looks incorrect. 
- Fixed series of steps
	- Position vertices in 3D space
	- Calculate colors at vertices
	- Draw triangles 
	- Apply texture
	- Put pixels on screen
- This rendering pipeline could only be configured, not significantly altered
- Lower polygon counts in this era, and no programmable shaders
#### 2000s
- Per fragment shading introduced:
	- A fragment is essentially a pixel in progress, when drawing a 3D triangle, a fragment is each individual spot that will map to a pixel; lighting calculations at every pixel for much higer quality
- Introduced shaders, a program that informs exactly how to calculate colours and positions, rather than being stuck with preset modes provided by the rendering pipeline.
	- Had vertex shaders, code that ran for every vertex mapped
	- Had fragment/pixel shaders, that ran for every pixel
	- Could now create realistic metal, convincing water with ripples, reflections, glowing effects, and custom visual styles e.g., cartoon shading, cel-shading, any artistic style really. 

#### 2020s 
- Battlefield V etc ray tracing cores leveraged
- This simulates how light functions in real life (calculated in reverse)
	1. **Ray generation** create a ray from the camera through each pixel
	2. **Intersection testing** find what object (if any) the ray hits first
	3. **Shading** calculate the colour at that intersection point
	4. **Recursive rays** spawn additional rays for reflections, refractions, shadows
- Ray tracing follows the actual physics of light, in reverse, rays leave the camera, and we trace them backward through the scene to the light sources (reverse for efficiency, so only trace rays that actually contribute to the image).
- Reflects objects not visible to camera, natural faloff and blending
- Approach is typically rasterise primary visibility, and ray trace only reflective surfaces, limited to 1-2 bounces. 
- RT cores are dedicated hardware components designed to accelerate ray trcing e.g., BVH traversal, accelerate ray-triangle intersection testing.
	- Traditional approach is traverse BVH, test ray-triangle intersections, and return closest hit. 


### What we will study
- **Rendering** - how to turn 3D scenes into 2D images
- **Models** - representing 3D geometry
- **Cameras** - defining viewpoint and projection
- **3D Geometry** - vertices, triangles, meshes
- **Transparency** - alpha blending, order-dependent rendering
- **Rasterisation** - converting triangles to pixels
- **Antialiasing** - removing jagged edges
- **Graphics APIs** - OpenGL and Vulkan
- **Shaders** - Writing vertex and fragment shaders in GLSL
- **Perspectives** - perpsecitve-correct interpolation
- **Textures** - mapping images to geometry
- **Rendering pipeline** - GPU stages
- **GPU hardware** - how it works
- **Light** - lighting models and equations
- **Shadows** - shadow mapping and variants
- **Curves** - bezier curves, splines
- **Bounding volume hierarchy** - acceleration structures
- **Ray tracing** - ray generation, intersection, and shading
- **Path tracing** - monte carlo integration, global illumination
- **The rendering equation** - mathematical foundation of light transport