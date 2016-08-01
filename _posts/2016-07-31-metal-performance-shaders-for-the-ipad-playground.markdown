---
published: false
title: Metal Performance Shaders for the iPad playground
layout: post
---
As many of you might have seen at `WWDC 2016`, the new [Playground app for the iPad](https://developer.apple.com/videos/play/wwdc2016/408/) was a really big hit! As an already playgrounds lover, for me it was even more than that. Now we are able to easily write `Swift` code on the iPad and run it with just a button tap. Today we are going to have fun with the __Metal Performance Shaders (MPS)__ since I have not discussed about them before, and also because it is so easy to use them on mobile devices. This reminds me to warn you, the `MPS` framework only works on `iOS` and `tvOS` devices but we will look into a very handy way of writing the playground on a `macOS` device and sharing it with your mobile device through the `iCloud Drive`.

To start, let's create a new `iOS` playground in `Xcode`. You can any image you like under the `Resources` folder. I already added one named __nature.jpg__. Next, write a few lines of code in the main playground page, like in this screenshot:

![alt text](https://github.com/MetalKit/images/raw/master/mps_1.png "1")

Let's go over this code

{% highlight swift %}
{% endhighlight %}

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!