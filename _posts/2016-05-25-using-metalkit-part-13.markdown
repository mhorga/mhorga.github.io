---
published: false
title: Using MetalKit part 13
layout: post
---
Let's pick up where we left off in [Part 12](http://mhorga.org/2016/05/18/using-metalkit-part-12.html). Using the same playground we worked on last time, we will learn about lighting and noise today. Remember the sun eclipse we worked on a couple of weeks ago? It's back! Well, we are going to remove the sun and just focus on the planet this time.

First, let's clean our shader to only include this code:

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

In our particular case __x0__, __y0__ and __z0__ are all __0__ because our sphere is in the center of the screen. Solving for __z__ gives us the equation below, so let's replace the last line in the kernel, with these lines:

{% highlight swift %}float planet = float(sqrt(radius * radius - uv.x * uv.x - uv.y * uv.y));
planet /= radius;
output.write(distance < 0 ? float4(planet) : float4(0), gid);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_1.png "1")

As you expected, the color is now calculated starting with fully white in the center of the circle and ending  with fully black on the circle contour. For that to happen, we had to divide the color by the `radius`, in order to normalize our range to the __[0, 1]__ interval. We basically faked having a light source positioned at __(0, 0, 1)__, which brings up to the next topic, `Lighting`.

In order to have lights in our scene we need to compute the `normal` at each coordinate. Normals are perpendicular vectors on the surface, showing us where the surface "points" to at each coordinate. Replace the last two lines with these lines:

{% highlight swift %}float3 normal = normalize(float3(uv.x, uv.y, planet));
output.write(distance < 0 ? float4(float3(normal), 1) : float4(0), gid);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_5.png "5")

This is probably not what we wanted to see, but at least we now know how normals look like when calculating the color at each coordinate.

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_3.png "3")

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_4.png "4")

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!

{% highlight swift %}
{% endhighlight %}