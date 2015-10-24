---
published: false
title: Image processing in iOS part 3
layout: post
---
In the last part of this series, we will look at how to manipulate pixel information to get great image effects. Let's take up where we left off in the previous part by creating a `RGBA` image from our `UIImage`:

{% highlight swift %} 
let image = UIImage(named: "image")!
let rgba = RGBA(image: image)!
{% endhighlight %}

The origin point `0 x 0` in the UIKit framework is at the top left corner of the screen while in the Core Graphics and Core Image frameworks is at the bottom left corner. This comes handy when we need to debug issues such as when an `upside-down` image resulted unexpectedly. Now we have access to the 2D-array of pixels which we can easily access by creating an index:

{% highlight swift %} 
let index = y * rgba.width + x
{% endhighlight %}

So for example if we need to access the pixel located at `15 x 20` we would need to vertically move the index `20` rows down on the `Y` axis, and horizontally `15` columns to the right, on the X axis. In other words, if we had an image sized `100 x 100` we would need to traverse 20 full rows and 15 more pixels on the next row, and the index would have a value of `2015` (interesting coincidence :-D).

{% highlight swift %} 
let newImage = contrast(rgba).toUIImage()
{% endhighlight %}

I want to thank [@JackTripleU](https://twitter.com/JackTripleU) who inspired me to write this series.

Until next time!