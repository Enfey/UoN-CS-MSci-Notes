# GPU architecture overview
- GPU architecture diverge from CPU design - CPUs were build around the idea of executing a stream of complex instructions as fast as possible via mechanisms like branch prediction, out of order execution, cache hierarchy, all dedicated to making one thread/core run quickly.
- GPU architecture is focused on parallelism. Every vertex in a 3D mesh needs the same transformation applied to it
	- Every fragment needs the same shader to run on it
	- There is no dependency between the data operated on then, in the GPU, which means that we do not need one fast processor then, but thousands of simpler ones all doing the same thing simultaneously.
- The GPU performs parallel processing of vertices and fragments on shader cores, which do not share memory with other shader cores.
- The application program sets up the vertices and sends the draw commands, which includes:
	- The actual vertex data e.g., VBO, usually alongside an index buffer which permits vertex reuse to save VRAM. 
	- Textures.
	- Shader programs which are compiled and uploaded to the GPU.
- Fully programmable vertex shaders are executed in parallel, whose main job it is to provide input for the fragment shader, and transform vertices through to clip space.
- Fixed-function parallel rasterising pipeline + post-vertex shader processing execute thereafter (clips triangles in clip space, performs perspective division, and performs viewport transformation to yield screen-space). Rasterisation behaves as standard.
- Following on from this, the queue of fragments outputted by the rasteriser into a FIFO queue are shaded in the pixel stage, executing the compiled fragment shader over the received fragments.
- These 3 stages run concurrently: Vertices -> FIFO QUEUE -> RASTERISE -> FIFO QUEUE -> PIXELS -> DISPLAY with the FIFO queues between them acting as the buffer.

## GPU Shader Core

## Multiprocessor

## Memory

## Video Display Controller

## Following a triangle through the GPU

## RTX 3070