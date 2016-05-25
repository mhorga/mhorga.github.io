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

To make our calculations simple, __x0__, __y0__ and __z0__ are all __0__ since we put our sphere in the center of the screen. We also assume __z__ is always __0__ because we are currently working with a __texture2d__ so we will just fake the notion of `depth` for now. So let's replace the last line in the kernel, with these lines:

{% highlight swift %}float planet = float(sqrt(radius * radius - uv.x * uv.x - uv.y * uv.y));
planet /= radius;
output.write(distance < 0 ? float4(planet) : float4(0), gid);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_1.png "1")

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_3.png "3")

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_4.png "4")

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!

{% highlight swift %}
{% endhighlight %}