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

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter13_1.gif "1")

So far so good. The planet looks pretty flat and the lighting is too uniformly distributed to look real. Let's fix that next.

The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!

{% highlight swift %}
{% endhighlight %}