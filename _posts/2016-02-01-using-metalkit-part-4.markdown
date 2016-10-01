---
published: true
title: Using MetalKit part 4
layout: post
---
Last time we looked at the `Metal Shading Language` basics. Before we look into more advanced topics, I just thought now is a good time to revisit what we have learned so far, especially about the _graphics pipeline_ which I admit, I might have gone too fast on that topic (thank you, anonymous reader, for your suggestion and valuable feedback!)

Lets look at the __graphics pipeline__ in more detail, and I'll start with a piece of history here. It all started about a decade ago, when __shaders__ were introduced as a way for programmers to be able to influence the __fixed pipeline__ that existed until then. At the same time, __floating point__ support was also introduced for `GPUs` facilitating the birth of __GPGPU__ (general-purpose computing on graphics processing units). As a consequence, the new __programmable pipeline__ was significantly changed:

![alt text](https://github.com/MetalKit/images/blob/master/part4_1.png?raw=true "1")

As you can see, the new `pipeline` now has two `shader` stages and that is where we can write our own customized code that the `GPU` will then run. The first part of a graphics program always runs on the `CPU` and is often called __host code__. Here is where most of the resource allocation happens, as well as staging the data transfer to and from the `GPU`. The most important part of the program, however, runs on the `GPU`. The two `shaders` go into a separate file with a __.metal__ extension (in other `GPGPU` frameworks such as `OpenCL` it is named __kernel code__). 

The pipeline starts with the `CPU` stage where the input is sent to the GPU in the form of _vertices_. They undergo transformation and per-vertex lighting. At this point the `vertex shader` can be used to manipulate the vertices prior to `rasterization`. After that, vertices undergo `clipping` and `rasterization` resulting in _fragments_. The `fragment shader` can then be run on each fragment before the pixel values are output to the `framebuffer` for display.

Now let's look at `Metal's` own pipeline. We will refer back to the [part 2 source code](https://github.com/MetalKit/metal) and we'll mention line numbers to exemplify the concepts we touch. Building a `Metal` application is done in two stages. The first one is the __Initialization__ stage:

![alt text](https://github.com/MetalKit/images/blob/master/part4_2.png?raw=true "2")

The very first step is getting the __device__ (line 19 in `MetalView.swift`). A device is the direct connection to the `GPU` driver and hardware; it's the source we need to create all the other objects in `Metal`. The second initialization step is creating a __command queue__ (line 40) which is our channel to submit work to the `GPU`. The third initialization step is creating the buffers, textures and other resources (lines 20-27). The `newBufferWithBytes` function will allocate a new block of shared memory, copy the provided pointer into it, and return a handle to that buffer. The fourth initialization step is creating the __render pipeline__ (lines 28-37) which is a chain of steps that start with taking vertex data from one end and producing a rasterized image on the other end. The pipeline consists of two elements: the __descriptor__ which holds the `shader` information and the pixel format, and the __state__ which is built from the `descriptor` and contains the compiled `shaders`. The fifth (last) initialization step is creating a view. For us it was easier to inherit from __MTKView__ (line 11) rather than creating a new `CAMetalLayer` and adding it as a subview.

Next, let's look at the second stage of building a `Metal` application, the __Drawing__ stage:

![alt text](https://github.com/MetalKit/images/blob/master/part4_3.png?raw=true "3")

The very first step is getting the __command buffer__ (line 40). All of the work that goes to the `GPU` will be enqueued into this buffer. We need the `command queue` from the previous stage to have a `command buffer` created. The second step is setting up a __render pass__ (lines 38-39). A render pass descriptor tells `Metal` what actions to take while an image is being rendered. Configuring it requires that we specify what color textures are we rendering to (the `currentDrawable` texture). We need to also choose which color the screen will be cleared to before we draw any geometry. The third step is the actually __drawing__ (lines 43-44). We specify the buffer where the vertices are stored and then the primitives we need drawn. The fourth and final step is to __commit the command buffer__ (lines 46-47) to the `GPU`. When calling `commit`, the `command buffer` gets encoded, sent to the end of the command queue, and executed on the `GPU` when its time comes.

I hope this part of the tutorial brought more clarity to understanding general topics such as the `graphics pipeline` and the `Metal pipeline`. I look forward to returning back to more coding in the next part of this tutorial.

Until next time!