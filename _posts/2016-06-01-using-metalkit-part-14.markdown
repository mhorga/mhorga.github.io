---
published: false
title: Using MetalKit part 14
layout: post
---
Let's pick up where we left off in [Part 13](http://mhorga.org/2016/05/25/using-metalkit-part-13.html). Using the same playground we worked on last time, we will learn about __noise__ today. From _Wikipedia_:

> `Noise` refers to any random fluctuations of data that makes the perception of an expected signal, difficult. `Value noise` is a type of noise commonly used as a procedural texture primitive in computer graphics. This method consists of the creation of a lattice of points which are assigned random values. The noise function then returns the interpolated number based on the values of the surrounding lattice points. Multiple octaves of this noise can be generated and then summed together in order to create a form of fractal noise.

The most obvious characteristic of noise is randomness. Since `MSL` does not provide a random function let's just create one ourselves. What we need is a random number between __[0, 1]__. We can get that by using the `fract` function which returns the fractional component of a number:

{% highlight swift %}float random(float2 p)
{
    return fract(sin(dot(p, float2(15.79, 81.93)) * 45678.9123));
}
{% endhighlight %}

The __noise()__ function will bilinearly interpolate a lattice (grid) and return a smoothed value. Bilinear interpolation allows us to transform our __1D__ `random` function to a value based on a __2D__ grid:

{% highlight swift %}float noise(float2 p)
{
    float2 i = floor(p);
    float2 f = fract(p);
    f = f * f * (3.0 - 2.0 * f);
    float bottom = mix(random(i + float2(0)), random(i + float2(1.0, 0.0)), f.x);
    float top = mix(random(i + float2(0.0, 1.0)), random(i + float2(1)), f.x);
    float t = mix(bottom, top, f.y);
    return t;
}
{% endhighlight %}

We first use __i__ to move along grid points and __f__ as an offset between the grid points. Then we calculate a __Cubic Hermite Spline__ with the formula `3f^2 - 2f^3` and which creates a S-shaped curve that has values between __[0, 1]__. 
    // https://www.desmos.com/calculator/mnrgw3yias
    // Interpolate the along the bottom of our grid.
    // Interpolate the along the top of our grid.
    // We have now interpolated the horizontal top and bottom grid lines.
    // We will now interpolate the vertical line between those 2 horizontal points
    // to get our final value for noise.

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_6.gif "6")

You can read more about [bilinear filtering](http://www.scratchapixel.com/old/lessons/3d-advanced-lessons/interpolation/bilinear-interpolation) and about [value noise](http://www.scratchapixel.com/old/lessons/3d-advanced-lessons/noise-part-1/creating-a-simple-2d-noise) if you're interested. You can also see an example of the [Cubic Hermit Spline](https://www.desmos.com/calculator/mnrgw3yias). The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!

{% highlight swift %}
{% endhighlight %}