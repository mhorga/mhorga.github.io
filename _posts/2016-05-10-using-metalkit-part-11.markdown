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

The next functions we look at are `abs` and `fmod`. The `abs` function simply returns the absolute value, or the distance of a number from `0`. In other words, any value loses its sign and always returns a non-negative value. The `fmod` function returns the remainder fractional part of a float (the equivalent of the modulo operator % for integers). Let's apply these two functions to some values and see what we can get:  

{% highlight swift %}float3 color = float3(0.7);
if(fmod(uv.x, 0.1) < 0.005 || fmod(uv.y, 0.1) < 0.005) color = float3(0,0,1);
float2 uv_ext = uv * 2.0 - 1.0;
if(abs(uv_ext.x) < 0.02 || abs(uv_ext.y) < 0.02) color = float3(1, 0, 0);
if(abs(uv_ext.x - uv_ext.y) < 0.02 || abs(uv_ext.x + uv_ext.y) < 0.02) color = float3(0, 1, 0);
output.write(float4(color, 1), gid);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_3.png "3")

First, we drew a grid of blue lines spaced out at __0.1__ between them and with a thickness of __0.005__. Next, we normalized the screen coordinates so we can work with the __[-1, 1]__ interval, and then drew the `X` and `Y` axes in red with a thickness of __0.02__. Finally, we drew the two diagonals in green with the same thickness, keeping in mind that __x - y__ gives us the decreasing slope (diagonal) while __x + y__ gives us the increasing one. 

Finally, let's use __sin()__, __cos()__, __fract()__, __dot()__ and __pow()__ together with other functions we already discussed:

{% highlight swift %}float2 cc = 1.1 * float2(0.5 * cos(0.1) - 0.25 * cos(0.2), 0.5 * sin(0.1) - 0.25 * sin(0.2) );
float4 dmin = float4(1000.0);
float2 z = (-1.0 + 2.0*uv) * float2(1.7, 1.0);
for(int i=0; i<64; i++) {
    z = cc + float2(z.x * z.x - z.y * z.y, 2.0 * z.x * z.y);
    dmin=min(dmin, float4(abs(0.0 + z.y + 0.5 * sin(z.x)), abs(1.0 + z.x + 0.5 * sin(z.y)), dot(z, z), length(fract(z) - 0.5)));
}
float3 color = float3(dmin.w);
color = mix(color, float3(1.00, 0.80, 0.60), min(1.0, pow(dmin.x * 0.25, 0.20)));
color = mix(color, float3(0.72, 0.70, 0.60), min(1.0, pow(dmin.y * 0.50, 0.50)));
color = mix(color, float3(1.00, 1.00, 1.00), 1.0 - min(1.0, pow(dmin.z * 1.00, 0.15)));
color = 1.25 * color * color;
color *= 0.5 + 0.5 * pow(16.0 * uv.x * (1.0 - uv.x) * uv.y * (1.0 - uv.y), 0.15);
output.write(float4(color, 1), gid);
{% endhighlight %}

The `sin()` function is just the `sine` of an angle, the `cos()` function is obviously the `cosine` of an angle, the `fract()` function returns the fractional part of a value, the `dot()` function returns the scalar product of two vectors and finally, the `pow()` function returns the value of a number raised to the power of another number. This code generates a beautiful fractal, a true piece of art courtesy of `Inigo Quilez`. The output image should look like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_4.png "4")

You should take some time to try to understand how the `magic` works here. If you have any questions about this code, feel free to contact me either on this blog or on [Twitter](https://twitter.com/mhorga_). The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!