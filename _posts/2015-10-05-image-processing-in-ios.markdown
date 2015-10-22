---
published: false
title: Image processing in iOS
layout: post
---
Last time we looked at animations in iOS. This time we will look into `image processing` and how it works in iOS. An image, in the most basic definition, is a 2-D (two dimensional) array of pixels which gives the `width` and `height` of the image. Each pixel contains information about its `color` and `opacity`, so in a pixel data structure we would need to reserve memory for each of the 3 component colors (`red`, `green`, `blue`) as well as for the opacity (`alpha`) channel. Since we need to encode color values between 0 and 255 we need 8 bits of memory because that would fit a value up to 256 (= 2 ^ 8). For all the colors and the alpha we would thus need 32 bits to store everything about a pixel.

Let's create a `struct` named __Pixel__ which has a 32-bit integer variable named __value__:

{% highlight swift %}
struct Pixel {
    var value: UInt32
}
{% endhighlight %}

Next we need to store the color and opacity information in this variable. The order bits are stored in the memory is right to left (`little endian`) in the Intel processor technology, so the `red` color would go at the end of the 32 bits memory location. In the example below, the red color will occupy spots 1 and 2.

{% highlight swift %}
0x78563412
{% endhighlight %}

 