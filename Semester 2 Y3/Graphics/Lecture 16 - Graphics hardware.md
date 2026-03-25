# GPU architecture overview
- GPU architecture diverge from CPU design - CPUs were build around the idea of executing a stream of complex instructions as fast as possible via mechanisms like branch prediction, out of order execution, cache hierarchy, all dedicated to making one thread/core run quickly.
- GPU architecture is focused on parallelism. Every vertex in a 3D mesh needs the same transformation applied to it
	- Every fragment needs the same shader to run on it
	- There is no dependency between the data operated on then, in the GPU, which means that we do not need one fast processor then, but thousands of simpler ones all doing the same thing simultaneously.
- The GPU performs parallel processing of vertices and fragments on shader cores, which do not share memory with other shader cores.
- The application program sets up the vertices and sends the draw commands, which includes(as setup):
	- The actual vertex data e.g., VBO, usually alongside an index buffer which permits vertex reuse to save VRAM. 
	- Textures.
	- Shader programs which are compiled and uploaded to the GPU.
	- Draw calls  trigger the entire pipeline which denote which vertex buffer to use, which shader program to use, which textures are bound, where in the buffer to start.
- Fully programmable vertex shaders are executed in parallel, whose main job it is to provide input for the fragment shader, and transform vertices through to clip space.
- Fixed-function parallel rasterising pipeline + post-vertex shader processing execute thereafter (clips triangles in clip space, performs perspective division, and performs viewport transformation to yield screen-space). Rasterisation behaves as standard.
- Following on from this, the queue of fragments outputted by the rasteriser into a FIFO queue are shaded in the pixel stage, executing the compiled fragment shader over the received fragments.
- These 3 stages run concurrently: Vertices -> FIFO QUEUE -> RASTERISE -> FIFO QUEUE -> PIXELS -> DISPLAY with the FIFO queues between them acting as the buffer.

## GPU Shader Core
- GPUs have thousands of shader cores.
- Shader cores are also sometimes called ALUs, Unified Shaders, Shader units, or simply Shaders
- Shader cores are specialised for processing vertices and fragments. 
- A shader core is a small processor, similar in manner to a CPU but less complex, that executes one instruction per cycle
- The instructions include:
	- **Fused Multiply and Add(FMA)**
		- Single most important instruction
		- Many computations in graphics reduce to $a = b * c + d$ 
		- Transforming a vertex with a matrix is dot products
		- Interpolating texture co-ordinates uses multiply-adds
		- Hardware that can do this in a single clock cycle rather than 2 seperate instructions is extremely valuable.
	- **Move (MV)**
	- **Compare (CMP)**
	- **Load (LDR)**
	- **Store (STR)**
	- **Branch (BR)**
	- **Mathematic operators (cos() etc)**

## Multiprocessor
- On the GPU, ALUs/Shader cores are grouped together with some memory equivalent to L1 cache access speed in a unit called a multiprocessor
- An example multiprocessor is denoted below:
	![](Pasted%20image%2020260324012834.png)
- A **warp** is a set of many shader cores all processing the same shader program, one instruction at a time, all together in sync, but operating on different data e.g., all running compiled vertex shader but for 32 different vertices.
- This is a NVIDIA convention, AMD uses 64, calling their equivalent a wavefront.
- Latency can be introduced when, for example an instruction tries to sample a texture, which can take several GPU cycles to be resolved.
	- If this happens, all cores in the warp will be stalled simultaneously.
	- This warp is therefore swapped for a different warp which is executing some other SIMD; when the original stall is resolved e.g., by texture RGB values becoming available, then the original warp is swapped back
	- We could view warps as programs, and their swapping as multitasking/multiprogramming.
- What happens when warps take divergent paths e.g., if statement different for a particular fragment?
	- This is known as warp divergence
	- When threads within a single warp take alternate execution paths, the GPU serialises the divergent paths by masking (disabling) the threads/shader cores not on the active path, reducing parallel efficiency
	- The reason for this is because at least in prior graphics cards (not the case now) threads in a warp did not maintain their own program counters. 
		- Newer architectures permit threads to have separate program counters which improves efficiency somewhat.
- Warps are executed and swapped in this manner until all are completed, minimising latency in execution. 


## Logical vs Physical pipeline
- It should be noted that GPUs have a logical, rather than physical rendering pipeline. 
- In older GPUs, there were literally separate hardware blocks for vertex processing and pixel processing. The problem with this was resource utilisation - imagine a scene with complex geometry but very simple shading, the vertex hardware would be overloaded while the pixel hardware sat idle.
- In modern GPUs, there is a **unified shader architecture**.
	- The same ALUs/Shader Cores execute both vertex shaders and fragment shaders
	- The hardware scheduler decided, based on what work is queued, whether a multiprocessor should be running vertex warps or fragment warps at any given moment. 
## Memory
- GPUs access **video memory** rather than CPU motherboard memory
- CPU memory is optimised for latency - it may, at any moment, be executing a complex branching program, jumping between different memory locations unpredictably
	- It needs to be able to fetch a piece of data quickly, process it, and fetch another piece of data that may be unrelated or related.
	- DRAM is designed around low latency to increase CPU throughput
- GPU memory however doesn't jump around unpredictably, it processes millions of fragments all needing texture data, or thousands of vertices all needing position data. The access pattern is predictable and parallelisable.
	- VRAM is therefore designed around high bandwidth, moving enormous amounts of data per second, rather than minimising latency for any individual request. 
- Data transfers from the CPU memory to GPU memory via the host interface.
	- On modern systems, this is **PCIe**
	- The PCIe bandwidth vs VRAM bandwidth is a crucial bottleneck
	- The connection between the CPU and GPU, for the 3070, is roughly 14x slower than the GPU's internal bandwidth
		![](Pasted%20image%2020260324015214.png)
	- This is why assets are uploaded to VRAM once at load time and the graphics pipeline is designed around this as repeatedly transferring data across PCIe would completely destroy performance. 
- Contents of VRAM often include:
	- Vertex buffers
	- Index buffers
	- Textures
	- Framebuffer
	- Depth buffer
	- Shader code
	- Etc
- The colour buffer is a 2D array in VRAM with one entry per pixel on the display that stores the RGBA value interpolated across multiple different stages, (though usually formulated mostly at depth testing, transparency, texture mapping, and fragment shading stages) for the current frame. 
	- Typically maintain two colour buffers, 1 for the currently displayed buffer, and 1 that is currently being interpolated into by the GPU.
	- Swaps when a frame has finished rendering.
		

## Video Display Controller
- The video display controller is a dedicated hardware block on the GPU responsible for reading the colour buffer in VRAM and sending it to the display. 
- The display controller reads the front buffer at a fixed-rate dictated by the display's refresh rate usually either 60, 144, or 240 times per second
	- This happens on a completely fixed schedule without reference to the actual contents of the back or front buffer, it simply wants a fresh image regardless of what that image is. 
- Refresh rate formally is the amount of times per second the display controller reads the colour buffer. It is a property of the monitor hardware.
- The frame rate is how fast the GPU produces a finished frame, which is variable.
	- If the GPU produces frames - that is, a complete colour buffer (and corresponding depth buffer), faster than the refresh rate, it is considered wasteful as some frames never appear on the screen, they will be overwritten before the display controller can read them. 
- If the GPU writes to the front buffer while the display controller is mid-way through reading it, that is the GPU replaces front buffer with back buffer contents you get screen-tearing.
	- The tears are horizontal due to how the colour buffer is read.
	- This happens in systems where:
		- GPU writes often to the frame buffer, faster than the display can read it
		- VSync is disabled, so the GPU swaps buffers immediately after finishing the frame, instead of matching the rate at which frames are completed to the rate at which they are read by the display controller.
			- But usually adds latency by having to wait for the frame to be rendered where the FPS is much higher than the refresh rate.
				![](Pasted%20image%2020260324230617.png)
- GSync is a monitor technology that dynamically alters the monitor refresh rate to match the FPS in real-time, eliminating both tearing and stuttering. 
- HDMI 2.1 supports data transfer rates including 8K at 60hz and 4K at 120hz.
	- For each pixel on screen, at each refresh cycle, the display controller must transmit colour data.
		![](Pasted%20image%2020260324230937.png)
	- HDMI 2.1 has a bandwidth of 48 Gbps, 6 GB/s

## Following a triangle through the GPU
- CPU runs the application code, which uses a graphics API to load vertex data to GPU memory via PCIe and issues draw calls via `glDrawArrays()` or equivalent.
- The PCIe bus carries the draw call and other data e.g., textures to the GPU
- The vertex shader programs run on GPU shader cores
- A multiprocessor can process both vertex warps and pixel warps in a concurrent manner, executing portions of a shader program, one instruction a time, across many shader cores, all responsible for different data. SIMD.
- After vertex shader programs are executed, a specialised piece of hardware close to the multiprocessor assembles and clips the triangles and performs viewport transformation. 
- Each triangle is rasterised on another specialised piece of hardware, which isn shared between several multiprocessors. There are multiple of these rasterisation hardware components on a GPU and each is responsible for a region of the display area. The specific one which processes a triangle depends on where the screen the triangle is located and a single triangle may be processed by many units.
- After rasterisation, fragments are generated which are processed by the fragment shader on the ALUs which calculate the fragment colour. 
- Depth testing and merging of fragment colours with the frame buffer is done on a final specialised hardware component, of which there are many on the GPU, with each one handling a subset of pixels in the buffer.

## RTX 3070

### GigaThread Engine
- Global scheduler for the entire GPU
- Receives draw calls from the CPU via PCIe
- Breaks work down into thread blocks
- Handles context switching at the GPU level.
- Handles load balancing. 
### Graphics Processing Clusters (GPCs)
- The RTX 3070 has 6 GPCs.
- Each GPC is a self-contained unit capable of running the entire rendering pipeline independently.
	- They contain:
		- 1x Raster engine
		- 16x Render Output Units
		- 6x TPCs
			- Each TPC contains:
				- 1x polymorph engine
				- 2x streaming multiprocessors
					- Which contain:
						- 128x shader cores
						- 4x texture units
						- 1x RT core.
- The reason for division into GPCs is to permit scaling for different products e.g., low-end consumer card vs high-end consumer card.
- Each one is independently connected to memory controllers and shared L2 cache, meaning they can largely operate in parallel without interference.
### Raster engines
- Each GPC has one Raster engine. 
- The raster engine performs rasterisation.
- A display area is split into quads which are each assigned to a raster engine:
	![](Pasted%20image%2020260324233154.png)
- A raster engine is responsible for rasterising any triangle which partially covers its screen quad, thus the specific raster engine which processes a triangle depends on where the screen the triangle is located.
	- Note that large triangles are rasterised by multiple raster engines simultaneously.
- Perform the edge testing, hierarchal rasterisation (first test large tiles, then subdivide only tiles that are partially covered, then test individual pixels), much faster than brute force, calculates interpolation gradients and generates fragments. All hardware.

### Render Output Units
- Handle depth testing and colour buffer merging.
- This is the final stage of the rendering pipeline for each fragment.
- When a fragment shader completes on given SMs, and produces colour into a colour buffer, those colours go to ROPs, which perform depth testing with regard to the depth buffer and then blends or overwrites the colour buffer accordingly (or does nothing, if fragments fail the depth test).
- The resulting colour is written back to VRAM at the correct pixel location. 
- Each ROP is responsible for certain pixel locations. 

### Texture Processing Clusters (TPCs)
- Each GPC has 6 TPCs yielding 36 across the entire GPU. 
- Each TPC contains a PolyMorph Engine and 2 Streaming Multiprocessors
	- The PolyMorph Engine runs before the vertex shader can run; it reads the vertex buffer, gathers the attributes for each vertex (position, normal, colour, UV, etc) and organises them ready for shader execution. 
	- It further organises vertices into warps of 32 and dispatches them to the SM's shader cores.
	- After the vertex shader outputs clip space co-ordinates, the PolyMorph engine performs perspective division; this is done on dedicated hardware to avoid wasting shader core cycles as it can be done faster here.
	- It further is responsible for clipping triangles that extend outside the view frustrum, which may result in new vertices, but we don't need to know about this trust.
	- Maps NDC co-ordinates to screen space using `glViewport()` settings.
	- Being inside the TPC, next to SMs is deliberate, as vertex data fetched from VRAM for the ALUs does not have to travel far to be processed by the polymorph engine and vice-versa.

### Streaming Multiprocessors
- 46 SMs on the RTX 3070 (some GPCs have slightly fewer) 
- Has 4 processing blocks, and each block contains:
	- 32x FP32 ALUs
		- Each one can perform FMA per clock cycle, and run the actual shader code.
		- 32 of them in sync form one warp, and so have 4 warps actively executing simultaneously in a single SM.
	- 16x INT 32 units
		- Integer units run in parallel with floating point units e.g., array index calculations
		- This is opposed to competing for the same execution units
	- 1x Tensor Core
		- Matrix multiplication acceleration
	- 4x LD/ST units (load/store - register access)
		- 4 units per block, handle reading from and writing to the register file, which is extremely fast shared storage that shader variables live in.
		- 4 is sufficient to keep up with typical shader workloads; not every instruction needs a memory access.
	- 1x SFU (special function unit)
		- Polynomial approximations of sin, cos, exp, log etc.
		- Interpolating vertex attributes?
	- Warp scheduler
		- Schedules 48 resident warps, selecting ready ones, and context switching stalled warps to hide latency.

### Texture Units

### RT Cores
- Ray-triangle intersection, confirming hit point is inside the triangle.
- Use BVH, see next lecture.

### Cache and Controllers
