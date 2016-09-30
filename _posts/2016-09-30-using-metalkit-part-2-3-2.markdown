---
published: true
title: Using MetalKit part 2*3^2
layout: post
---
Yes, as the title suggests, we're going to have another of those posts with math (and fun) in it. I was thinking the other day, what can we do while commuting for an hour or so, without internet and possibly without a laptop as well, just carrying an `iPad` with us. Luckily, the `iPad` now has the awesome `Swift Playgrounds` app.

Let's start with a new playground that runs a basic compute kernel. Since the current version of the `Swift Playgrounds` app does not yet let us edit the `Auxiliary Source Files`, where all our `Swift` and `Metal` files usually reside, we will have to write our code in the main playground page, but it is not that complicated. All we have to do is modify our `MetalView` initializer and let it take in an extra argument -- our shader/kernel code. Then we start building our code by adding more lines to this long string.

Let's start with a light blue sky background color:

{% highlight swift %}let shader =
"#include <metal_stdlib>\n" +
"using namespace metal;" +
"kernel void k(texture2d<float,access::write> o[[texture(0)]]," +
"              uint2 gid[[thread_position_in_grid]]) {" +
"   float3 color = float3(0.5, 0.8, 1.0);" +
"   o.write(float4(color, 1.0), gid);" +
"}"
{% endhighlight %}

If you run the playground now, the output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_1.png "1")

Next, let's draw a gradient. we divide the current pixel coordinates to the screen dimensions and we get __UV__ -- a pair of floats between __(0-1)__. we then multiply the fixed color with __Y__ -- the vertical component of `UV` which gives us the gradient:

{% highlight swift %}"   int width = o.get_width();" +
"   int height = o.get_height();" +
"   float2 uv = float2(gid) / float2(width, height);" +
"   color *= uv.y;" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_2.png "2")

Let's work on a nicer background next. A smooth gradient would look like a great sunset. We can use __mix__ to blend colors. We tell the function to blend vertically, and take the complement of __Y__ to switch the colors:

{% highlight swift %}"   float3 color = mix(float3(1.0, 0.6, 0.1), float3(0.5, 0.8, 1.0), sqrt(1 - uv.y));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_3.png "3")

From here we could go to drawing a black hole. We would achieve that by using a distance function (__length__) to draw black in the middle of the screen __(0.5, 0.5)__ and add more and more background color outside of it, until we reach the maximum value in the screen corners. Replace the last line with:

{% highlight swift %}"   float2 q = uv - float2(0.5);" +
"   color *= length(q);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_4.png.png "4")

Next we use __smootstep__ to draw a round shape that is black inside, blue outside and a blended color between __r__ and __(r + 0.01)__. Replace the last line with:

{% highlight swift %}"   float r = 0.2;" +
"   color *= smoothstep(r, r + 0.01, length(q));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_5.png "5")

If we're not satisfied with a circular perimeter, we could make it _bumpy_ by using math functions such as __cos__ and __atan2__. We generate here __9__ spikes (__frequency__) with a spike length (__amplitude__) of __0.1__:

{% highlight swift %}"   float r = 0.2 + 0.1 * cos(atan2(q.x, q.y) * 9.0);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_6.png "6")

Adding the __X__ coordinate to the _cosine_ phase introduces a spike _bend-like_ effect:

{% highlight swift %}"   float r = 0.2 + 0.1 * cos(atan2(q.x, q.y) * 9.0 + 20.0 * q.x);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_7.png "7")

You can rotate them by adding a small number such as __1.0__ to the _cosine_ value:

{% highlight swift %}"   float r = 0.2 + 0.1 * cos(atan2(q.x, q.y) * 9.0 + 20.0 * q.x + 1.0);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_8.png "8")

If you think this is starting to look like a palm tree canopy, I see it too! We can draw its trunk using __abs__ which gives us horizontal/vertical distances instead of _euclidian_ distances (to a given point) like _length_ did, so let's take the __X__ distance and add these lines (we are reusing both __r__ and __color__) after the existing ones: 

{% highlight swift %}"   r = 0.015;" +
"   color *= smoothstep(r, r + 0.002, abs(q.x));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_9.png "9")

We can remove the unneeded part of the trunk by using another __smoothstep__ on the __Y__ coordinate:

{% highlight swift %}"   color *= 1.0 - (1.0 - smoothstep(r, r + 0.002, abs(q.x))) * smoothstep(0.0, 0.1, q.y);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_10.png "10")

Since both the canopy and trunk are using __q__, modifying its value will move both:

{% highlight swift %}"   float2 q = uv - float2(0.67, 0.29);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_11.png "11")

By introducing a __sin__ function we can bend the trunk. A too small _frequency_ does not bend it enough while a too high _frequency_ bends it too much so __2.0__ seems right. An _amplitude_ of __0.25__ also moves the base of the trunk towards the edge of the screen so it looks right (as an aside, changing the sign from __+__ to __--__ will shift the base to the other side):

{% highlight swift %}"   color *= 1.0 - (1.0 - smoothstep(r, r + 0.002, abs(q.x - 0.25 * sin(2.0 * q.y)))) * smoothstep(0.0, 0.1, q.y);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_12.png "12")

The trunk, however, is too smooth. To add irregularities to its surface we use __cos__ again. A high _frequency_ and low _amplitude_ seem to be what we need to make it look right:

{% highlight swift %}"   r = 0.015 + 0.002 * cos (120.0 * q.y);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_13.png "13")

Also, trunks are usually shaping the ground a little around their base, so an __exp__ function is what we need here because it grows slowly in the beginning and then it soars to the skies. We use an _attenuation_ factor of __-50.0__:

{% highlight swift %}"   r = 0.015 + 0.002 * cos (120.0 * q.y) + exp(-50.0 * (1.0 - uv.y));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_14.png "14")

We can increase the presence of the second color by using __sqrt__ which gives us bigger numbers (when used on sub-unitary numbers) to work with. The sunset is about to end soon:

{% highlight swift %}"   float3 color = mix(float3(1.0, 0.6, 0.1), float3(0.5, 0.8, 1.0), sqrt(1 - uv.y));" +
{% endhighlight %}

The final image on the iPad should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_15.png "15")

In conclusion, we saw how to use __sqrt__ to shape transitions, then __cos__ to create variations of ups and downs in shapes, then __exp__ that allows us to create curves, then __smoothstep__ for thresholding, then __abs__ for symmetry and __mix__ for blending. Still commuting? Why don't you take a look at how this nice clover was created:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_16.png "16")

I want to say thanks to [Inigo Quilez](https://twitter.com/iquilezles) again, for keep inspiring me to write more and more about drawing with math. All the math in this tutorial belongs to him. The [source code](https://github.com/MetalKit/metal) is posted on `Github` as usual.

Until next time!