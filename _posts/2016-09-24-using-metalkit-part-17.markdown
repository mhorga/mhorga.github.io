---
published: true
title: Using MetalKit part 17
layout: post
---
I am writing this article for three reasons: first, to tell you that I am working on updating all the `Metal` code to `Swift 3` and then moving the tutorials to a new home with a nicer design and a proper domain name; second, I wanted to show you a different way to work with `MetalKit` other than subclassing `MTKView`, that is, using the `MTKViewDelegate`. And third, I wanted to answer one of our readers' question about how to draw wireframes.

Let's start by using the code from Part 4 which is an Xcode project but we will turn it into a playground this time. This is going to be a shockingly short tutorial but all you have to do is add the following line right before encoding the draw command:

{% highlight swift %}renderEncoder.setTriangleFillMode(.lines)
{% endhighlight %}

That's it! Run the playground and enjoy the wireframed triangle. If you don't want it to have an interpolated color, in the fragment shader also replace the return line with a constant color like green, for example:

{% highlight swift %}return half4(0.0, 1.0, 0.0, 1.0);
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter17.png "Wireframe")

The [source code](https://github.com/MetalKit/metal) is posted on `Github` as usual.

Until next time!