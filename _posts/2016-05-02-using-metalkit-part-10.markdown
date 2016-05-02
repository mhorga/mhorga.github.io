---
published: false
title: Using MetalKit part 10
layout: post
---
Today we will look at the only other type of `shader function` we have not used so far, the __kernel function__ or __compute shader__. You will often hear a variation of intermixed words from both of them. A few important facts to keep in mind about them: there is no rendering going on, the function always returns `void` and its name always starts with the __kernel__ keyword, just like the other functions we used before we preceded by the `vertex` and `fragment` keyword.

Let's start by stripping down the playground we used in [Part 8](http://mhorga.org/2016/03/07/using-metalkit-part-8.html). First, delete the `MathUtils.swift` file as we will not need it anymore. Then, in `MetalView.swift` delete the `createBuffers()` function, as well as its call inside the initializer, and also the two global variables for the buffers. Replace the `MTLRenderPipelineState` declaration with a __MTLComputePipelineState__ declaration. Next on to the `registerShaders()` function. Here are the differences between the old and the new version of it:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_1.png "1")

As you notice, we're not using a `MTLRenderPipelineDescriptor` anymore and instead create our `MTLComputePipelineState` with the kernel function itself. Next, let's see the differences for the `drawRect()` function:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_2.png "2")

You notice immediately that the `currentRenderPassDescriptor` is not used anymore. As such, the command encoder is created by using the `computeCommandEncoder()` function instead. Obviously, we do not need to set the vertex buffers anymore or draw primitives either. Instead, with a kernel function we set a texture to work with, create thread groups and then dispatch them to do work. 

Finally, we go the `Shaders.metal` file and replace everything with the code below:

{% highlight swift %} 
#include <metal_stdlib>

using namespace metal;

kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    uint2 gid [[thread_position_in_grid]])
{
    output.write(float4(0, 0.5, 0.5, 1), gid);
}
{% endhighlight %}

We basically just set the same color to every pixel/position in the texture. Now if you go to the home page of the playground and if you are showing the `Timeline` in the `Assistant editor` you should have a similar view:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_3.png "3")

{% highlight swift %} 
{% endhighlight %}

Special thanks to [Chris Wood](https://twitter.com/_psonice) for his valuable advising. The [source code](https://github.com/Swiftor/Metal/tree/master/ch10) is posted on Github as usual.

Until next time!