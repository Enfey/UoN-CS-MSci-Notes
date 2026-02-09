# Rendering pipeline
- Rendering includes the following 5 main steps:
	- Specify vertex data
	- Execute a vertex shader to calculate each vertex position
	- Assemble triangles from the vertices
	- Rasterise the triangles to create fragments
	- Execute a fragment shader to calculate the fragment colours. 
		![](Pasted%20image%2020260206000912.png)

## Introduction
- A modern OpenGL program is not a single program, it is:
	- One CPU program (C/C++)
		- Creates windows, handles user input, prepares data, sends instructions to GPU
	- Several GPU programs (shaders); some steps of the rendering pipeline are implemented in hardware on the GPU and OpenGL provides an interface to that functionality
	- A shared state machine (OpenGL context)
- OpenGL is a state based API allowing control of the GPU pipeline, most function calls modify global GPU state.
	- The GPU executes asynchronously, the CPU issues commands into a **command queue**
	- The GPU processes these commands later in order, and the CPU does not block unless explicitly forced. 
- We use:
	- **OpenGL** - the core overarching graphics API
	- **GLSL** - shader language used to write shaders that run on the GPU
	- **GL3W** - loads OpenGL functions so that we can use them.
	- **GLFW** - window creation + user input handling

## Structure of a graphics program
1. Make a window
2. Initialise OpenGL
3. Specify vertices
4. Compile Shaders
5. Create buffers, Send to GPU
6. Render loop
7. Deinitialisation
	![](Pasted%20image%2020260208164639.png)
Specify vertices does not appear here as we have already seen this in lecture 2. 


### Additional OpenGL info
- When GLFW creates a window, and we call glfwMakeContextCurrent, we instantiate a context
	- Technically, a context owns all OpenGL objects, shaders, biuffers etc exist inside that context 
	- Destroying the window destroys the context. 
- A **draw call** is a command issued by the CPU that tells the GPU to execute the **graphics pipeline** using the currently bound state. 
	- Triggers execution of:
		- Vertex fetching
		- Vertex shader
		- Primitive assembly
		- Rasterisation
		- Fragment shader
		- Framebuffer writes. 
	- Essentially process a batch of geometry using the current configuration

## Simplest example shaders
During this module, we study both vertex and fragment shaders. 
### Simplest vertex shader
```C
#version 450 core

layout (location = 0) in vec4 vPos;

void main()
{
	gl_position = vPos;
}
```
Input variable instantiated on line 3, means input to this shader stage, of type vec4, with identifier vPos. This input variable represents the position of a single vertex. vPos will come from the vertex buffer passed to the GPU. This is simply a pass through shader; it does not modify/transform/project the vertices position, but simply sets the global variable gl_position which is a built in output variable of the vertex shader. 
### Simplest fragment shader
```C
#version 450 core

layout (location = 0) out vec4 fColour;

void main()
{
	fColour = vec4(1.f, 0.f, 0.f, 1.0f); // R, G, B, A
}
```
Output variable declared on line 3 with the modifier out, of type vec4 called fColour, which is used to represent the fragment colour. The variable is set to the colour red on line 7, meaning all fragments passed into this shader will have the colour red. 


### Shader Input and Output variables
![](Pasted%20image%2020260208171331.png)
Shader input variables are passed into the shader from the previous stage in the rendering pipeline.
- In the case of the vertex shader, the input variable is passed from the application stage in the form of a vertex (comes from the vertex specification in the C/C++ program that runs on the CPU). 
- In the case of the fragment shader, the input variable is passed from the rasterisation stage in the form of a fragment. 

Shader output variables are passed out of the shader onto the next stage in the rendering pipeline.
- In the case of the vertex shader, the output variable is passed to the vertex post processing stage in the form of a vertex, which is where primitive assembly takes place.
- In the case of the fragment shader, the output variable is passed to the colour buffer in the form of a colour, the colour buffer will then be displayed. 

### A more detailed look at the code
#### 1. Make a window
```C
glfwInit();
GLFWwindow window = glfwCreateWindow(640, 480, "A triangle", NULL, NULL);
glfwMakeContextCurrent(window);
```
- The method `int glfwInit (void)` initialises GLFW and needs to be called before any other GLFW method is used. 
- The method `GLFWwindow * glfwCreateWindow(int width, int height, const char * title, GLFWmonitor, GLFWwindow * share)` creates and returns a window of `width` and `height` pixels, with `title` as the title. Setting `monitor` to `NULL` will make a sized window rather than being fullscreen. `Share should also be null`.
- `void glfwMakeContextCurrent (GLFWwindow * window)` makes the context of `window` the current one, so that drawing happens to that window. 

#### 2. Initialise OpenGL
- The code for initialising openGL looks like this `gl3wInit();`, OpenGL methods need to be loaded before we can use them. This is accomplished using an OpenGL extension wrangler, we use GL3W which loads the core profile of OpenGL 3 and 44. 
- Provides access to the core OpenGL API. 

#### 3. Specify Vertices
- Recall that a triangle is determined by 3 vertices. 
- Each vertex has one of more vertex attributes, which may include a 3-dimensional position in the form of a vector (mandatory for rendering), coloud have a colour, and maybe more attributes. 
- Interpolation by the rasteriser means that once a fragment is determined to lie inside the triangle, that fragment's attribute values can be computed by weighting the triangle's vertex attributes using barycentric co-ordinates (this is done automatically on the hardware).
- Vertex specification occurs as a C/C++ array
- The screen space is defined as -1 to to 1 left to right, and -1 to 1 bottom to top, with 0,0 as the center, we skip the model/view/projection stages thus far by specifying vertices within these constraints. 
	- ![](Pasted%20image%2020260208173936.png)
- Each vertex is defined row by row, each composed of 5 floats, 2 floats for the position attribute, and 3 floats for the colour attribute.

#### 4. Compile Shaders
- The overall code to compile the two shaders that we use is as follows:
	 ![](Pasted%20image%2020260208174131.png)
- The process of *compiling a shader is as follows*
	1. Use OpenGL to create a shader.
	2. Specify the source code. Each shader program is specified as a raw string.
	3. Use OpenGL to compile the shader. 
	This shader is essentially sent to the GPU driver, which compiles it into GPU machine code, which is why shader compilation errors appear at **runtime**.
- The method `GLuint glCreateShader (GLenum shaderType)`
	- Allocates a shader object on the GPU, and returrns a handle (integer ID) for referencing it. In this module we use `shaderType` `GL_VERTEX_SHADER` and `GL_FRAGMENT_SHADER`.
- The `read_file()` function
	- Will be provided for the module, the source code which defines a shader is read from a file using the function. See the lab for the implementation. 
- The function `glShaderSource(GLuint shader, GLsizei count, const GLchar **string, const GLInt *length)`
	- Sets the shader source as specified in `string` to the `shader`.
- The method `void glCompileShader(GLuint shader)`
	- The GPU driver compiles the shader, syntax, types, interfaces are checked, failure here means that the shader won't run. 
		- Shaders cannot run alone, a shader program is a linked combination of shader stages that form a complete pipeline. 
- The method `GLuint glCreateProgram()`
	- Creates a shader program, creating a container for shaders, and returns a handle for referencing. Stores the shaders for the rendering process.
- The method `glAttachShader(GLuint program, GLuint shader)`
	- Attaches `shader` to `program`, both shaders need to be attached to the shader program. 
- The method `glLinkProgram(program)`
	- Invokes OpenGL to link `program`, slightly different to standard linking e.g., if vertex shader outputs `out vec3 vCol`, then the fragment shader must have `in vec3 vCol`, but yes generally the same checking interface matching. Produces executable GPU code. 
#### 5. Create buffers, Send to GPU, Interface Matching
- OpenGL stores all the data that is required during execution in **buffer objects**, which is a way to interface with GPU memory
- The programmer needs to setup two OpenGL objects to specify how the vertex attributes array, is parsed and passed as input to the vertex shader
	1. **VBO** - The *Vertex Buffer Object* resides in **GPU VRAM** and stores raw vertex data; just a copy of the vertex attribute array.
	2. **VAO** - The *Vertex Array Object* contains all of the necessary details for OpenGL to render, including a copy of the VBO just created, and metadata regarding how vertex buffer elements are to be interpreted by the vertex attributes. 
		E.g., stores strides + offsets, attribute layouts, attribute indices, all metadata. 
- The method `glCreateBuffers(GLsizei n, GLuint * buffers);
	Creates `n` buffer objects, in VRAM on the GPU, and stores handles in the `buffers` array. 
- The method `void glNamedBufferStorage(GLuint buffer, GLsizeiptr size, const void *data GLbitfield flags)`
	Allocates `size` bytes in the `buffer` object's new data store, and copies `data` from the RAM to the VRAM data store, and `flags` for now can be set to `NULL`. 
- The method `glGenVertexArrays(GL sizei n, GLuint *arrays);
	Creates `n` VAOs and stores handles to them in `arrays` array. 
- The method `glBindVertexArray(GLuint array)` 
	Binds the **VAO** `array` so that any vertex attribute setup modifies this VAO.
- The method `glBindBuffer(GLenum target, GLuint buffer)`
	Binds `buffer` to `target`, which should be set to `GL_ARRAY_BUFFER`, so operations that refer to `GL_ARRAY_BUFFER` use this buffer's handle. 
 - Each attribute in the vertex attribute array needs to be described in the VAO. The way each vertex attribute is specified in OpenGL code is with paired calls of `glVertexAttribPointer`, `glEnableVertexAttribArray`.
 - The method `void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalised, GLsize i stride, const void *offset)`
	 Specifies that the `index` attribute has `size` number of components of the specified `type`, where each vertex is made from `stride` bytes, and `offset` is the number of bytes from the beginning of an arbitrary vertex it takes to reach the described attribute. 
	 Example:
```C
glVertexAttribPointer(
  0,        // index
  2,        // size (x, y)
  GL_FLOAT,
  GL_FALSE,
  5 * sizeof(float), // stride
  (void*)0           // offset
);
```
- The method `void glEnableVertexAttribArray (Gluint index)`
	Enables `index` vertex attribute, and follows a call to the prior method. 
- Recall that vertices are specified as a C/C++ array of type float, which will be in RAM on the host, until `glNamedBufferStorage` allocates enough VRAM on the GPU, and copies the values from RAM to VRAM and returns a handle. The reason we use C/C++ is so we can access contiguous memory directly.
	- Thus, vertices are stored on both the CPU and GPU memory storage in respective sections of contiguous memory and we use `glVertexAttribPointer` to specify how vertex attributes are organised in this memory. 
	 ![](Pasted%20image%2020260208182849.png)
	- This way of specifying, describing, and enabling attributes is unified with GLSL code. Vertex shader input variables are specified with the layout qualifier, which look like:
		`layout (location = 0) in vec4 vPos;`
		- This specifies that the first input variable of a vertex shader is `vec4` and represents the position of the vertex. 
		- This maps directly to the vertex position as described in `glVertexAttribPointer()`'s `index` argument.
		- In the worksheet example, we only specify 2 floats for the vertex position when we use `glVertexAttribPointer` and then use a `vec4` for the corresponding `vPos` input variable. OpenGL will fill in the missing 2 floats with a `1.0` value when it passes them into the vertex shader. 
- The output of a vertex shader must match the input of the fragment shader, and this is checked when the shader program is linked. Must have the same name and type. 
#### 6. Render loop
- A simple render loop looks like the following:
```C
while (!glfwWindowShouldClose(window))
{
	static const GLfloat bgd[] = { 1.f, 1.f, 1.f, 1.f };
	glClearBufferfv(GL_COLOUR, 0, bgd);
	
	glUseProgram(program);
	glBindVertexArray(VAOs[0]);
	glDrawArrays(GL_TRIANGLES, 0, 3);
	
	glfwSwapBuffers(windows);
	glfwPollEvents();
}
```
- The program needs to continuously render the current state to the screen so that graphics can be animated, and this is achieved using a render loop as follows:
	![](Pasted%20image%2020260208183753.png)
- The method `int glfwWindowShouldClose (GLFWwindow * window)`
	- Returns the close flag of `window` such that the render loop can terminate when the user has closed the window, meaning the render loop will loop repeatedly while the window is open. 
- The method `void glCreateBuffer (GLenum buffer, GLint drawbuffer, const GLfloat * value);`
	- Clears `buffer` colour buffer to the colour stored in `value`. The parameter `drawbuffer` can be set to 0 for now. 
- The method `void glBindVertexArray(GLuint array)`
	- Binds the VAO array,
- The method `void glDrawArrays(GLenum mode, GLint first, GLsizei count)`
	- Renders the triangles, this is achieved using `mode` set to `GL_TRIANGLES`, starting from `first` index, and `count` is the number of vertices to be drawn. This causes the GPU to read vertices from the buffer in the bound `VBO` according to the `VAO` and begin rendering. 
	- It is important to note that the GPU is not synchronised with the CPU, even after `glDrawArrays`, there is no guarantee that the `GPU` will have finished drawing to the colour buffer, or even started the rendering process.
	- A draw call means that the GPU reads vertices, runs the vertex shader, rasterises, runs the fragment shader, and writes pixels.
- Double buffering is used in computer graphics to improve the image quality
	- In this process, a first buffer, A, is used for displaying the colour of the pixels, whilst a second buffer, B, is being written to by OpenGL. After the frame has been displayed, the buffers are swapped, so that the new colours in B are displayed, and the next frame is prepared by OpenGL in the other buffer A. 
- The method `void glfwSwapBuffers(GLFWwindow * window)`
	 - Swaps the front and back buffers used in double buffering. 
- The method `void glfwPollEvents(void)` processes input events. 
#### 7. Deinitialise 
```C
glfwDestroyWindow(window);
glfwTerminate(); // deinitialises GLFW. 
```


### Debugging OpenGL
#### Callback
- Can define a callback function which is executed whenever a certain event occurs, in this case, when an error occurs.
	![](Pasted%20image%2020260208190243.png)
- Prototype signature, type can be checked for `GL_DEBUG_TYPE_ERROR`, `severity` is low, medium, or high, `message` is an error message string which can be printed. Don't need source, id, length, and userParam. 
- The code used for debugging in OpenGL looks like 
	![](Pasted%20image%2020260208190418.png)
- The method `glEnable(GL_DEBUG_OUTPUT)` enables debugging so debug messages are produced by OpenGL

#### Debugging shader compilation
- After compiling the vertex and fragment shaders and linking the shader program, we must check for errors. 
	![](Pasted%20image%2020260208190605.png)
	- The method `void glGetShaderiv(Gluint shader, GLenum pname, GLint *params)`
		- Using `pname` set to `GL_COMPILE_STATUS` checks for compilation errors in `shader`, and stores the result in `params`.
	- The method `void glGetShaderInfoLog(GLuint shader, GLsize i, maxLength, GLsizei *length, GLchar *infoLog)`
		- Queries the information log of `shader`, where `maxLength` is the size of the character buffer, `length` can be NULL, and `infoLog` specifies an array of characters that is used to return the information log. 
	- The method `void glGetProgramiv (GLuint program, GLenum pname, GLint *params)`
		- with `pname` set to `GL_LINK_STATUS`, checks for link errors in `program`, and stores the result in `params`.
	- The method `void glGetProgramInfoLog(GLuint program, GLsizei maxLength, GLsizei *length, GLchar *infolog)`
		- Queries the log of `program`, where `maxLength` is the size of the character buffer, `length` can be `NULL` and `infoLog` specifies an array of characters that is used to return the information log. 

#### Handling User Input
https://www.glfw.org/docs/3.0/group__keys.html
Key press function needs to be called in the render loop after polling events. Handle window resize by specifying a resize callback to adjust viewport. Set this callback after creating the window. 
#### Uniforms
- A variable which can have value set by the application program before the GLSL program is executed, and remains immutable
- Uniforms are declared with the `uniform` qualifier in GLSL
- In the application program, first find the location of the uniform, then set the value:
	![](Pasted%20image%2020260208191518.png)
	- The method `GLint glGetUniformLocation(GLuint program, const GLchar *name)` 
		- Queries the location of `name` uniform variable in `program`
	- The method `void glUniformlf(GLint location, GLfloat v0)`
		- Modifies the uniform variable at `location` to `v0` new value
- The uniform needs to be declared in shaders
	- ![](Pasted%20image%2020260208191726.png)
