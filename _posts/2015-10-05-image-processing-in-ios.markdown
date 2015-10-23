---
published: true
title: Image processing in iOS
layout: post
---
Last time we looked at animations in iOS. This time we will look into `image processing` and how it works in iOS. An image, in the most basic definition, is a 2-D (two dimensional) array of pixels which gives the `width` and `height` of an image. Each pixel contains information about its `color` and `opacity`, so in a data structure we would need to reserve memory for each of the 3 primary colors (`red`, `green`, `blue`) as well as for opacity (`alpha` channel). Since we need to encode color values between 0 and 255 we need 8 bits of memory since that would fit a value up to 256 (= 2^8). For all the colors and the alpha we would thus need 32 bits to store everything about a pixel.

Let's create a `struct` named __Pixel__ that has a 32-bit integer named __value__:

{% highlight swift %}
struct Pixel {
    var value: UInt32
}
{% endhighlight %}

Next we need to store the color and opacity information in this variable. The order bits are stored in the memory is right to left (`little endian`) by the `Intel` processor technology, so the `red` color would go at the end of the 32 bits memory location, followed to the left by `green`, then `blue` and finally `alpha` so the known `RGBA` abbreviation would in fact be `ABGR`. In the example below, the `red` color will occupy locations 1 and 2.

{% highlight swift %}
0x78563412
{% endhighlight %}

In order to get the value for the `red` color we need to bitwise `AND` the `value` variable with a `0xFF` mask. That means - discard all the `0s` while AND-ing the value with eight `1s`, and this will save (mask) the last two locations that we need:

{% highlight swift %}
var red = UInt8(value & 0xFF)
{% endhighlight %}

It is now easier to understand how to get the other values, by shifting to the right 8 bits inside `value` for each of them:

{% highlight swift %}
var green = UInt8((value >> 8) & 0xFF)
var blue = UInt8((value >> 16) & 0xFF)
var alpha = UInt8((value >> 24) & 0xFF)
{% endhighlight %}

By shifting to the right, the `green` value moves to the right side 8 bits and will now occupy locations 1 and 2, and so on. Now let's see the reverse process - how can we change the `red` value once we have something stored in the `value` variable:

{% highlight swift %}
value = UInt32(red) | (value & 0xFFFFFF00)
{% endhighlight %}

So we first pad the value of `red` with `0s` so it fits a 32-bit memory location, then we bitwise AND the old `value` with a mask to clear the old value for `red` (while keeping the other colors and opacity intact), and we finally bitwise OR it with the new value for `red`. As you expected, the other values can be set in a similar manner, except this time we shift to the left because all colors and alpha only occupy 8 bits so they need to be arranged into their places:

{% highlight swift %}
value = (UInt32(green) << 8) | (value & 0xFFFF00FF)
value = (UInt32(blue) << 16) | (value & 0xFF00FFFF)
value = (UInt32(alpha) << 24) | (value & 0x00FFFFFF)
{% endhighlight %}

In the next part of this series we will look at how we can actually manipulate the pixel information to get desired effects.

Until next time!