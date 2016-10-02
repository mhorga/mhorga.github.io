---
published: true
title: Using MetalKit part 10
layout: post
---
Today we will look at the only other type of `shader function` we have not used so far, the __kernel function__ or __compute shader__. You will often hear a variation of intermixed words from both of them. The kernel is used for `compute` tasks, that is, massively parallel computations done on the `GPU`. Some examples include: image processing, scientific simulations, and so on. A few important facts to keep in mind about kernels: there is no rendering pipeline, the function always returns `void` and its name always starts with the __kernel__ keyword, like the other functions we used before were preceded by the `vertex` and `fragment` keyword.

Let's start by stripping down the playground we used in [Part 8](http://mhorga.org/2016/03/07/using-metalkit-part-8.html). First, delete `MathUtils.swift` as we will not need it anymore. Then, in `MetalView.swift` delete the `createBuffers()` function, as well as its call inside the initializer, and the two variables for the buffers. Replace the `MTLRenderPipelineState` declaration with a __MTLComputePipelineState__ declaration. Next, on to the `registerShaders()` function. Here are the differences between the old and the new version of it:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_1.png?raw=true "1")

As you notice, we're not using a `MTLRenderPipelineDescriptor` anymore and instead create our `MTLComputePipelineState` with the kernel function itself. Next, let's see the differences for the `drawRect()` function:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_2.png?raw=true "2")

Notice that the `currentRenderPassDescriptor` is not used anymore. The command encoder is created by using the `computeCommandEncoder()` function instead. Obviously, we do not need to set the vertex buffers anymore or draw primitives either. Instead, with a kernel function we set a texture to work with, create thread groups and then dispatch them to do work. We use `MTLSize` to set the dimensions of each thread group and the number of thread groups that will be executed in each compute call.

Finally, we go the `Shaders.metal` file and replace everything inside, with the code below:

{% highlight swift %}#include <metal_stdlib>
using namespace metal;

kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    uint2 gid [[thread_position_in_grid]])
{
    output.write(float4(0, 0.5, 0.5, 1), gid);
}
{% endhighlight %}

We basically just set the same color to every pixel/position in the texture. Now if you go to the home page of the playground and if you are showing the `Timeline` in the `Assistant editor` you should have a similar view:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_3.png?raw=true "3")

If you see the output above, you are now ready to proceed. From this point forward, we will not look at the host code (`MetalView.swift`) anymore because all our work will be inside the kernel shader. 

Alright, let's start with something simple. Replace the content of the kernel function with the code below:

{% highlight swift %}int width = output.get_width();
int height = output.get_height();
float red = float(gid.x) / float(width);
float green = float(gid.y) / float(height);
output.write(float4(red, green, 0, 1), gid);
{% endhighlight %}

As you probably guessed already, we get the `width` and `height` of the texture, then compute the value of `red` and `green` based on the pixel position in the texture, and then we write back the new color to the texture. You should see something like this:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_4.png?raw=true "4")

Next, let's draw a black circle in the middle of the screen. Replace last line with these lines:

{% highlight swift %}float2 uv = float2(gid) / float2(width, height);
uv = uv * 2.0 - 1.0;
bool inside = length(uv) < 0.5;
output.write(inside ? float4(0) : float4(red, green, 0, 1), gid); 
{% endhighlight %}

You should see something like this:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_5.png?raw=true "5")

How exactly did we just do that? Well, this is a common technique used in shading, and is named __distance function__. We use the `length` function to determine whether the pixel is within __0.5__ distance from the center of the screen which we are just using as the center of the circle as well. Notice that we normalized the __uv__ vector to match the range of the window coordinates __[-1, 1]__. Finally, we look whether the pixel is inside and color it `black`, or otherwise color it with a gradient, as we did before.

Let's abstract the inside/outside circle calculation into a useful distance function:

{% highlight swift %}float dist(float2 point, float2 center, float radius)
{
    return length(point - center) - radius;
} 
{% endhighlight %}

Then replace the line where we define `inside` with these lines:

{% highlight swift %}float distToCircle = dist(uv, float2(0), 0.5);
bool inside = distToCircle < 0;
{% endhighlight %}

Visually, nothing's changed, but we now have a function we can easily reuse later on. Next, let's see how we can modify the background color based on the distance from the circle rather than just the absolute pixel position. We change the value of the alpha channel by calculating the distance from the pixel to the circle. Simply replace the last one with this one:

{% highlight swift %}output.write(inside ? float4(0) : float4(1, 0.7, 0, 1) * (1 - distToCircle), gid);
{% endhighlight %}

You should see something like this:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_6.png?raw=true "6")

Beautiful, right? Now that we just got this idea of a total eclipse of the Sun, let's make it look more realistic. We need one more circle (the Sun) and we want to move the initial circle a bit to the left and a little lower so they are both visible. Replace the line where we define `inside` with these ones:

{% highlight swift %}float distToCircle2 = dist(uv, float2(-0.1, 0.1), 0.5);
bool inside = distToCircle2 < 0;
{% endhighlight %}

You should see something like this:

![alt text](https://github.com/MetalKit/images/blob/master/chapter10_7.png?raw=true "7")

We are just beginning to scratch the surface of shading techniques. In the next episode we will look into more complex and dynamic compute tasks. Special thanks to [Chris Wood](https://twitter.com/_psonice) for his valuable advising. The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!