---
published: false
title: Image processing in iOS part 3
layout: post
---
In the last part of this series, we will look at how to manipulate pixel information to get great image effects. Let's take up where we left off in the previous part by creating a `RGBA` image from our `UIImage`:

{% highlight swift %} 
let image = UIImage(named: "image")!
var rgba = RGBA(image: image)!
{% endhighlight %}

The origin point `0 x 0` in the `UIKit` framework is at the top left corner of the screen while in the `Core Graphics` and `Core Image` frameworks is at the bottom left corner. This comes in handy when we need to debug issues such as when an _upside-down_ image resulted unexpectedly. We have access to the 2D-array of pixels which we can easily access by creating an index:

{% highlight swift %} 
let index = y * rgba.width + x
{% endhighlight %}

So for example if we need to access the pixel located at `15 x 20` we would need to vertically move the index `20` rows down the `Y` axis, and horizontally `15` columns to the right, on the X axis. In other words, if we had an image sized `100 x 150` we would need to traverse `20` full rows and `15` more pixels on the next row, and the index would have a value of `2015 (= 20 x 100 + 15)`. Interesting coincidence! ;-)

Now we are able to access the pixel at the desired index, like this:

{% highlight swift %} 
var pixel = rgba.pixels[index]
{% endhighlight %}

If you remember from last time, the `RGBA` struct has a pointer to a `Pixel` type which acts like an array. The next thing we would like is to see what `color` and `alpha` values the pixel stores, and eventually change some of these values, like for example set the pixel color to full `green`, store the new color and then look at the image to see the outstanding `green` pixel:

{% highlight swift %} 
pixel.red = 0
pixel.green = 255
pixel.blue = 0
rgba.pixels[index] = pixel
rgba = RGBA(image: image)!
{% endhighlight %}



{% highlight swift %} 
let pixel = rgba.pixels[index]
{% endhighlight %}

A special `Thanks` for [@JackTripleU](https://twitter.com/JackTripleU) who inspired me to write this series.

Until next time!