---
published: true
title: Using MetalKit part 13
summary: Learn about 3D drawing and basic lighting using the Metal Shading Language and compute kernels.
layout: post
---
Let's pick up where we left off in [Part 12](http://mhorga.org/2016/05/18/using-metalkit-part-12.html). Using the same playground we worked on last time, we will learn about lighting and `3D` objects today. Remember the sun eclipse we worked on a couple of weeks ago? It's back! Well, we are going to remove the sun and just focus on the planet this time.

First, let's clean our kernel to only include this code:

{% highlight swift %}int width = output.get_width();
int height = output.get_height();
float2 uv = float2(gid) / float2(width, height);
uv = uv * 2.0 - 1.0;
float radius = 0.5;
float distance = length(uv) - radius;
output.write(distance < 0 ? float4(1) : float4(0), gid);
{% endhighlight %}

You surely recognize all this code from few weeks ago. The only thing we did is replace the outside circle color with `black` and the inside circle color with `white`. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_0.png "0")

So far so good. The planet looks pretty flat and the lighting is too uniformly distributed to look real. Let's fix that next. Geometry tells us that in order to find any point on a sphere, we need the sphere equation:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_2.png "2")

In our particular case __x0__, __y0__ and __z0__ are all __0__ because our sphere is in the center of the screen. Solving for __z__ gives us the the value of the `planet` color, so let's replace the last line in the kernel, with these lines:

{% highlight swift %}float planet = float(sqrt(radius * radius - uv.x * uv.x - uv.y * uv.y));
planet /= radius;
output.write(distance < 0 ? float4(planet) : float4(0), gid);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_1.png "1")

As you expected, the color is now calculated starting with fully white in the center of the circle and ending  with fully black on the outer circle. For that to happen, we had to divide the color by the `radius`, in order to normalize our range to the __[0, 1]__ interval for the `z` value, which gives us a full range light effect. We actually faked having a light source positioned at __(0, 0, 1)__. This brings up to the next topic: `lighting`.

__Lighting__ is what gives life to our colors. In order to have lights in our scene we need to compute the `normal` at each coordinate. Normals vectors that are are perpendicular on the surface, showing us where the surface "points" to at each coordinate. Replace the last two lines with these lines:

{% highlight swift %}float3 normal = normalize(float3(uv.x, uv.y, planet));
output.write(distance < 0 ? float4(float3(normal), 1) : float4(0), gid);
{% endhighlight %}

Notice we already had the value of `z` in the `planet` variable. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_5.png "5")

This is probably not what we wanted to see, but at least we now know how normals look like when calculating the color at each normalized coordinate. Next, let's create a source of light located to our left (negative `x`) and a little behind us (positive `z`). Replace the last line with these lines:

{% highlight swift %}float3 source = normalize(float3(-1, 0, 1));
float light = dot(normal, source);
output.write(distance < 0 ? float4(float3(light), 1) : float4(0), gid); 
{% endhighlight %}

We adopted a basic light model called [Lambertian](https://en.wikipedia.org/wiki/Lambertian_reflectance) (diffuse) light, where we need to multiply the normal with the normalized light source. We will talk more about lighting in a future article, however, if you are interested in learning more about lighting models, [here](https://www.evl.uic.edu/aej/488/lecture12.html) is a great resource for your reference. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_3.png "3")

Remember from last time that our kernel also gives us a timer uniform? Let's use it for fun and profit! Replace the `source` line with this one:

{% highlight swift %}float3 source = normalize(float3(cos(timer), sin(timer), 1));
{% endhighlight %}

By using the `cos` and `sin` functions, we gave the light source a circular movement. `x` and `y` are both ranging from __-1__ to __1__ using the parametric equation of a circle. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_6.gif "6")

We how have a good looking, illuminated object in the scene (planet in the sky), however, the object still presents a homogeneous surface. We can make it look more realistic in two ways: either apply a texture to it, or add some noise to the `planet` color. The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!
