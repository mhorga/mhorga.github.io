---
published: false
title: Using MetalKit part 11
layout: post
---
Let's continue our journey into the wonderful world of shaders using the `Metal Shading Language (MSL)` by picking up where we left off in [Part 10](http://mhorga.org/2016/05/02/using-metalkit-part-10.html). Using the same playground we worked on last time, we will next try to get close to making art using `MSL` math functions such as `sin`, `cos`, `pow`, `abs`, `fmod`, `clamp`, `mix`, `step` and `smoothstep`. 

First, let's look at our "sun eclipse" example from last time. Strangely enough, we start from the end of the list of functions above because `smoothstep` is the function we need to fix an issue we had last time and we did not pay attention to it -- our output image has jaggies (is aliased) as you can see below if we zoom in enough to make it visible:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_1.png "1")

The `smoothstep` function depends on a `left edge` being smaller than a `right edge`. The function takes a real number `x` as input and outputs `0` if `x` is less than or equal to the left edge, `1` if `x` is greater than or equal to the right edge, and smoothly interpolates between `0` and `1` otherwise. The difference between the `step` and the `smoothstep` function is that `step` makes a sudden jump from `0` to `1` at the edge. The `smoothstep` function implements cubic `Hermite` interpolation after doing a clamp. An improved version, named `smootherstep`, has zero `1st` and `2nd` order derivatives at `x=0` and `x=1`:

{% highlight text %}
smoothstep(X) = 3X^2 - 2X^3

smootherstep(X) = 6X^5 - 15X^4 + 10X^3
{% endhighlight %}

Let's implement the `smootherstep` function:

{% highlight swift %}
float smootherstep(float e1, float e2, float x)
{
    x = clamp((x - e1) / (e2 - e1), 0.0, 1.0);
    return x * x * x * (x * (x * 6 - 15) + 10);
}
{% endhighlight %}

The `clamp` function moves the point to the nearest available value, given a `min` and `max` value. The input takes the value of `min` if less than it, the value of `max` if greater than it, and keeps its value if in between. Our __compute__ kernel should now look like this:

{% highlight swift %}int width = output.get_width();
int height = output.get_height();
float2 uv = float2(gid) / float2(width, height);
uv = uv * 2.0 - 1.0;
float distance = distToCircle(uv, float2(0), 0.5);
float xMax = width/height;
float4 sun = float4(1, 0.7, 0, 1) * (1 - distance);
float4 planet = float4(0);
float radius = 0.5;
float m = smootherstep(radius - 0.005, radius + 0.005, length(uv - float2(xMax-1, 0)));
float4 pixel = mix(planet, sun, m);
output.write(pixel, gid);
{% endhighlight %}

The last function we look at before moving on, is `mix`. The `mix` function performs a linear interpolation between `x` and `y` using `a` to weight between them. The return value is computed as `x * (1 - w) + y * w`. In this case, the `planet` color and `sun` color are interpolated using `smootherstep` as weight. If you execute the playground, the output image now has anti-aliasing and the jaggies are all gone:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_2.png "2")

The next functions we look at are `abs` and `fmod`.

{% highlight swift %}float3 color = float3(0.7);
if(fmod(uv.x, 0.1) < 0.005 || fmod(uv.y, 0.1) < 0.005) color = float3(0,0,1);
float2 uv_ext = uv * 2.0 - 1.0;
if(abs(uv_ext.x) < 0.02 || abs(uv_ext.y) < 0.02) color = float3(1, 0, 0);
// x+y = `up and right` diagonal and x-y = `down and right` diagonal
if(abs(uv_ext.x - uv_ext.y) < 0.02 || abs(uv_ext.x + uv_ext.y) < 0.02) color = float3(0, 1, 0);
output.write(float4(color, 1), gid);
{% endhighlight %}

Next

{% highlight swift %}
{% endhighlight %}

The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!