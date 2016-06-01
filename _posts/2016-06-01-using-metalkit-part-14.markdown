---
published: false
title: Using MetalKit part 14
layout: post
---
Let's pick up where we left off in [Part 13](http://mhorga.org/2016/05/25/using-metalkit-part-13.html). Using the same playground we worked on last time, we will learn about __noise__ today. From _Wikipedia_:

> `Noise` refers to any random fluctuations of data that makes the perception of an expected signal, difficult. `Value noise` is a type of noise commonly used as a procedural texture primitive in computer graphics. This method consists of the creation of a lattice of points which are assigned random values. The noise function then returns the interpolated number based on the values of the surrounding lattice points. Multiple octaves of this noise can be generated and then summed together in order to create a form of fractal noise.

The most obvious characteristic of noise is randomness. Since `MSL` does not provide a random function let's just create one ourselves. What we need is a random number between __0__ and __1__. We can get that by using the `fract` function which returns the fractional component of a number:

{% highlight swift %}float random(float2 p)
{
    return fract(sin(dot(p, float2(15.79, 81.93)) * 45678.9123));
}
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_6.gif "6")

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!

{% highlight swift %}
{% endhighlight %}
