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

## Video Display Controller
- 

## Following a triangle through the GPU

## RTX 3070