## OpenGL as a driver-controlled command system
- OpenGL is a rasterisation rendered, which means that geometric primitives are rasterised to produce fragments. 
	- This is opposed to Vulkan, a ray-tracing rendered, which means a ray is traced through each pixel to test if it intersects with any triangles.
- OpenGL is an API for writing software that controls the GPU. The actual implementation of OpenGL is part of the GPU driver installed onto your system, which translates the API functions into commands the GPU can understand. 
	- The implementation details of many `gl***()` functions we use are highly specific to the particular GPU we use. 

## OpenGL Objects
- OpenGL objects are specialised constructs stored in the GPU's main memory (VRAM).
- These are essentially because they must be accessed by multiple ALUs and raster engines, amongst other GPU/GPC components.
- Note that how memory is organised on the GPU is not known to a graphics programmer.
- We focus on the following OpenGL objects:
	1. Vertex Buffer Object
	2. Vertex Array Objects
	3. Program Pipeline Objects
	4. Texture Objects
	5. Framebuffer Objects
	OpenGL objects have corresponding `glGen***(), glBind***(), glDelete***()` functions. 
	- `glGen***()` generates a number of objects, that is, reserving an integer ID for that object.
	- `glBind***()` binds an object so that OpenGL operations take effect on the bound object. Teels which object is active.
	- `glDelete***()` deletes a number of objects, invalidating their handle/ID. 
	Ids are used to communicate with the GPU resources/objects.
		When using `glBind***()` it sends the $id$ created at object generation to the driver, and ensures it points to a specific contiguous block of memory in VRAM. 
- In modern OpenGL objects are categorised by how they interact with VRAM. While all objects technically reside on the GPU, they do not all "allocate" memory in the same way.
	- Data store objects which hold bulk data, require explicit storage allocation because the driver needs to know the exact runtime size and hardware optimisation hints to manage VRAM fragmentation (simply contiguous).
		- **VBOs** and **Texture Objects** fall under this category
		- Their allocation is handled differently:
			- `glNamedBufferStorage()` - permits allocation of continuous data store in VRAM and often copies data from a `malloc()`'ed block in RAM simultaneously.
			- `glTexImage2d` - defines the width, height, and internal format e.g., RGB. Have a sampling state via `glTexParameteri()` which tells the GPU how to handle magnification etc.
	- State and container objects act as managers, and do not need a large manual allocation call because their size is small and fixed, or they "borrow" memory from other objects
		- **VAOs**
			- Links a VBO with formatting instructions. 
		- **Shader and Program Objects**
			- The final executable linked from multiple shaders is the program object, and the memory for this final machine code is allocated internally by the GPU driver during `glLinkProgram`.
			- Values that are constant for every vertex in a draw call are stored directly inside the Program Object.
		- **Framebuffer object**
			- Colour buffer's and depth buffer
			- Has zero bytes of storage initially
			- Use `glFrameBufferTexture2d()` to 'plug in' an existing Texture Object, thus borrowing its memory.

### Vertex Buffer Objects
- A *Vertex Buffer Object* stores vertex data
- The memory for the vertex data is allocated on the GPU and the vertex data is simultaneously copied to the GPU via `glNamedBufferStorage()`
- Assuming there are 2 triangles, made of 3 vertices each, with only a position attribute formed of a `vec3` in C, then we can dynamically allocate memory as follows:
	`float* verts = (float*) malloc(sizeof(float) * (3 * 3 * 2));`
- Yields 8 floats and stores the pointer. 
- The vertex data needs to be copied to the GPU memory, achieved as follows:
	`glNamedBufferStorage(bfrs[0], sizeof(float) * (3 * 3 * 2), verts, 0);`
	Last argument is flag (here says immutable and static)
	Access via id `bfrs[0]`


### Vertex Array Objects
- A *Vertex Array Object* consists of a VBO storing vertex array data, along with the formatting of the vertex data for formal interpretation. 
- Suppose there is a vertex array set up for two triangles, each of which has vertex attributes for position and colour as follows:
	`float* verts = (float*) malloc(sizeof(float) * (6 * 3 * 2));`
	`glNamedBufferStorage(bfrs[0], sizeof(float) * (6 * 3 * 2), verts, 0);`
- This data is then organised in VRAM and RAM as follows:
	- ![](Pasted%20image%2020260325205152.png)
- An attribute at a given index represents a vector of between of between 1 and 4 components, and specifies how that vector should be interpreted.
- If a verttex attribute defined in the vertex shader to be passed to the fragment shader has a different number of components than specified by `glVertexAttribPointer()` then the following rules are applied:
	- If there are additional values in the vertex array, then they are not used and are discarded.
	- If there are less values in the vertex array, the default values of $(0, 0, 0, 1)$ are used for each consecutive expected value. 
- To set up the VAO for the vertex data we depicted prior, we would use the following calls:
	![](Pasted%20image%2020260325213629.png)
	
### Shader Programs
- Shader Objects represent GLSL code.
- A shader object consists of GLSL source code, a compilation success flag, errors generated by a failed compilation, and compiled object code. 
	- Shaders are compiled by the GPU driver.
	- They are generally not stored in VRAM, as this would be fruitless; the GLSL, compile flags, and error flags would be entirely wasteful.
	- Only store the compiled shader code.
- A program object is what we actually run on the GPU, formed from compiled and linked shaders.  It further stores uniforms. 

```C
GLuint vs = glCreateShader(GL_VERTEX_SHADER);
glCompileShader(vs);

GLuint fs = glCreateShader(GL_FRAGMENT_SHADER);
glCompileShader(fs);

GLuint program = glCreateProgram();
glAttachShader(program, vs);
glAttachShader(program, fs);
glLinkProgram(program);
```
- Note that the handles created here are local, and the shader object code is in the driver's memory
- We only copy the program object over via `glLinkProgram()` which allocated uniform locations, resolves inputs and outputs, and produces a single executable. 
- There may be *Pipeline Objects* where each stage can be a separate program e.g., a vertex program and a fragment program, say you had multiple frag shaders, we want to avoid recompiling and relinking duplicate vertex shaders against these independent fragment shaders
- Instead, we compile and link each stage/shader as a standalone program and execute them independently, along with some additional modification to ensure that the stages interface uniformly.
- In the pipeline object, we then combine these independently combined programs. We typically do not use them in this module. 

### Textures
- A **texture** is an OpenGL object that contains image data
	![](Pasted%20image%2020260325220050.png)
- The GPU must fetch texels when rendering during execution of the fragment shader so textures must be stored in memory accessible to the GPU.
- The texture object's Texture Storage for the image data is allocated in VRAM in a call to the function `glTexImage2D()` which also copies the image data from RAM to VRAM. 
	`loadImageData(filename, pxls);`
- Textures store parameters; both sampling and texture parameters are set via `glTexParameteri()`.

### Framebuffer objects
- OpenGL framebuffers are called FrameBuffer Objects, which consist of Colour Buffers and a Depth buffer
	![](Pasted%20image%2020260325220519.png)
- The default framebuffer is created by the windowing system and is used during rendering, with the contents of the front colour buffer being displayed.
- A framebuffer object is a user-created framebuffer, what allows us to create our own rendering targets. This has no storage initially and must be given attachments to be valid. 
- ROPs write the colour of a single pixel and therefore need access to the framebuffer.
- The rendering pipeline by default has the Default Framebuffer bound
- `glBindFrameBuffer` can be used to bind and unbind a framebuffer object to the rendering pipeline to redirect the output of ROPs to a user-created framebuffer. 
- For example, the code below binds a user-created framebuffer, draws geometry, and binds the Default Framebuffer back again. Since the default FrameBuffer was not bound this will draw nothing to the monitor. 
	![](Pasted%20image%2020260325220940.png)
- `glClear(GL_DEPTH_BUFFER_BIT)` is used to clear the depth buffer of the currently bound framebuffer to specified values. 
- `glClearBufferfv()` can be used with `GL_COLOR` and a float array representing a colour to clear the colour buffer of the currently bound framebuffer to specified values. 
- `glFramebufferTexture2D()` is used to attach an existing Texture Object memory storage to a colour buffer or depth buffer of a Framebuffer Object. This can be useful when keeping a copy of a colour buffer or depth buffer after the rendering pipeline has been used for rendering.


![](Pasted%20image%2020260325221217.png)