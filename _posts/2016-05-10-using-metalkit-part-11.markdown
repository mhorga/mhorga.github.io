---
published: false
title: Using MetalKit part 11
layout: post
---
Let's continue our journey into the wonderful world of shaders using the `Metal Shading Language (MSL)` by picking up where we left off in [Part 10](http://mhorga.org/2016/05/02/using-metalkit-part-10.html). Using the same playground we worked on last time, we will next try to get close to making art using `MSL` math functions such as `sin`, `cos`, `tan`, `abs`, `fmod`, `step`, `clamp`, `mix` and `smoothstep`. 

First, let's look at our "sun eclipse" example from last time. Strangely enough, we start from the end of the list of functions above because `smoothstep` is the function we need to fix an issue we had last time and we did not pay attention to it -- our output image has jaggies (is aliased) as you can see below if we zoom in enough to make it visible:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_1.png "1")

The `smoothstep` function depends on a `left edge` being smaller than a `right edge`. The function takes a real number `x` as input and outputs `0` if `x` is less than or equal to the left edge, `1` if `x` is greater than or equal to the right edge, and smoothly interpolates between `0` and `1` otherwise. The `smoothstep` function implements cubic `Hermite` interpolation after doing a clamp. An improved version, named `smootherstep`, has zero `1st` and `2nd` order derivatives at `x=0` and `x=1`:

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

Our output image now has anti-aliasing and the jaggies are all gone:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter11_2.png "2")

{% highlight swift %}
{% endhighlight %}

The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!