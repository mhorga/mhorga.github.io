---
published: false
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

Let's work on a nicer background next. A smooth gradient would look like a great sunset. We can use __mix__ to blend colors. We tell the function to blend vertically, and take the reverse of __Y__ to switch the colors:

{% highlight swift %}"   float3 color = mix(float3(1.0, 0.6, 0.1), float3(0.5, 0.8, 1.0), sqrt(1 - uv.y));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_3.png "3")

From here we could go to drawing a black hole. We would achieve that by using a distance function (__length__) to draw black in the middle of the screen __(0.5, 0.5)__ and add more and more background color outside of it, until we reach the maximum value in the screen corners. Replace the last line with:

{% highlight swift %}"   float2 q = uv - float2(0.5);" +
"   color *= length(q);" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_4.png "4")

Next we use smootstep to draw a round shape that is black inside, blue outside and a blended color between r and (r + 0.01). Replace the last line with:

{% highlight swift %}"   float r = 0.2;" +
"   color *= smoothstep(r, r + 0.01, length(q));" +
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter18_5.png "5")



The [source code](https://github.com/MetalKit/metal) is posted on `Github` as usual.

Until next time!