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

Now we have access to the 2D-array of pixels which we can easily access by creating an index:

{% highlight swift %} 
let index = row * rgba.width + column
{% endhighlight %}

The screen origin point `0 x 0` in UIKit framework is at the top left corner while in the Core Graphics and Core Image frameworks is at the bottom left corner. This comes handy when we need to debug issues such as an `upside-down` image resulted.

{% highlight swift %} 
let newImage = contrast(rgba).toUIImage()
{% endhighlight %}

I want to thank [@JackTripleU](https://twitter.com/JackTripleU) who inspired me to write this series.

Until next time!