---
published: false
title: Using MetalKit part 10
layout: post
---
Today we will look at the only other type of `shader function` we have not used so far, the __kernel function__ or __compute shader__. You will often hear a variation of intermixed words from both of them. The kernel is used for `compute` tasks, that is, massively parallel computations done on the `GPU`. Some examples include: image processing, scientific simulations, even mathematic operations on really enormous matrices and vectors. A few important facts to keep in mind about kernels: there is no rendering going on, the function always returns `void` and its name always starts with the __kernel__ keyword, just like the other functions we used before we preceded by the `vertex` and `fragment` keyword.

Let's start by stripping down the playground we used in [Part 8](http://mhorga.org/2016/03/07/using-metalkit-part-8.html). First, delete the `MathUtils.swift` file as we will not need it anymore. Then, in `MetalView.swift` delete the `createBuffers()` function, as well as its call inside the initializer, and also the two global variables for the buffers. Replace the `MTLRenderPipelineState` declaration with a __MTLComputePipelineState__ declaration. Next on to the `registerShaders()` function. Here are the differences between the old and the new version of it:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_1.png "1")

As you notice, we're not using a `MTLRenderPipelineDescriptor` anymore and instead create our `MTLComputePipelineState` with the kernel function itself. Next, let's see the differences for the `drawRect()` function:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_2.png "2")

You notice immediately that the `currentRenderPassDescriptor` is not used anymore. As such, the command encoder is created by using the `computeCommandEncoder()` function instead. Obviously, we do not need to set the vertex buffers anymore or draw primitives either. Instead, with a kernel function we set a texture to work with, create thread groups and then dispatch them to do work. We use `MTLSize` to set the dimensions of each thread group and the number of thread groups that will be executed in each compute call.

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

If you see the output above, you are now ready to proceed. From this point forward, we will not look at the host code (`MetalView.swift`) at all anymore because all our work will be inside the shader. Alright, let's start with something simple. Replace the content of the kernel function with the code below:

{% highlight swift %} 
int width = output.get_width();
int height = output.get_height();
float red = float(gid.x) / float(width);
float green = float(gid.y) / float(height);
output.write(float4(red, green, 0, 1), gid);
{% endhighlight %}

As you probably guessed already, we get the `width` and `height` of the texture, then compute the value of `red` and `green` based on the pixel position in the texture, and then we write back the new color to the texture. You should see something like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_4.png "4")

Next, let’s draw a black circle in the middle of the screen. Replace last line with these lines:

{% highlight swift %}
float2 uv = float2(gid) / float2(width, height);
uv = uv * 2.0 - 1.0;
bool inside = length(uv) < 0.5;
output.write(inside ? float4(0) : float4(red, green, 0, 1), gid); 
{% endhighlight %}

You should see something like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_5.png "5")

How did we do that?

In the next episode we will look into more complex and dynamic compute tasks. Special thanks to [Chris Wood](https://twitter.com/_psonice) for his valuable advising. The [source code](https://github.com/Swiftor/Metal/tree/master/ch10) is posted on Github as usual.

{% highlight swift %} 
{% endhighlight %}

Until next time!