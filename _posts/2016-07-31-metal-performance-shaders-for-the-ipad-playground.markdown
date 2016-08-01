---
published: true
title: Metal Performance Shaders for the iPad playground
layout: post
---
As many of you might have seen at `WWDC 2016`, the new [Playground app for the iPad](https://developer.apple.com/videos/play/wwdc2016/408/) was a really big hit! As an already playgrounds lover, for me it was even more than that. Now we are able to easily write `Swift` code on the iPad and run it with just a button tap. Today we are going to have fun with __Metal Performance Shaders (MPS)__ since I have not discussed about them before, and also because it is so easy to use them on mobile devices. This reminds me to warn you, the `MPS` framework only works on `iOS` and `tvOS` devices. We will look into a handy way of writing the playground on a `macOS` device and sharing it with your mobile device through the `iCloud Drive`. Also, the `iPad` playgrounds are working only on `iOS 10` or newer.

To start, let's create a new `iOS` playground in `Xcode`. You can any image you like under the `Resources` folder. I already added one named __nature.jpg__. Next, write a few lines of code in the main playground page, like in this screenshot:

![alt text](https://github.com/MetalKit/images/raw/master/mps_1.png "1")

Let's go over the code. We have been writing most of this code over and over in the previous blog posts. The first new addition you will notice immediately is also generating an error, and that is because `macOS` does not "know" about it. Once we load this playground on an iPad, the error will go away:

{% highlight swift %}import MetalPerformanceShaders
{% endhighlight %}

Next, we use `MTKTextureLoader` to create a new texture from the image we added above. Now comes the really interesting part! Once we created our `MTLCommandBuffer` object, we are not going to also create a `MTLCommandEncoder` from this command buffer as we were used to do. Rather, we create a new `MPSImageGaussianBlur` object as in the code below:

{% highlight swift %}let texOut = view.currentDrawable!.texture
let shader = MPSImageGaussianBlur(device: view.device!, sigma: 5)
shader.encode(commandBuffer: commandBuffer, sourceTexture: texIn, destinationTexture: texOut)
{% endhighlight %}

What's great about the `MPS` objects is that they let you apply a compute shader (kernel function) to an input texture without us even having to configure any states, descriptors, pipelines or even a kernel function! The `MPS` object takes care of everything for us. Of course, by taking this approach we are somewhat limited to only picking a preset shader and possibly changing a parameter such as __sigma__, for this particular shader.

So far so good! We are now ready to send our playground to the `iPad` through the `iCloud Drive`. Open a `Finder` window, click on `iCloud Drive` and copy your playground into this folder:

![alt text](https://github.com/MetalKit/images/raw/master/mps_8.PNG "2")

We are finally getting to use the `iPad`! Open the new `Playgrounds` app that you downloaded from the `App Store` and go to `My Playgrounds`. In the top left corner of the screen, tap on __+__. As you can see you can also create a new playground on the `iPad` and then easily export it to your `macOS` devices or share it using various other tools. For now, however, we are going to tap on `iCloud Drive` as seen below:

![alt text](https://github.com/MetalKit/images/raw/master/mps_2.PNG "2")

When the `iCloud Drive` window pops up, notice that we have access to the `iPad`'s private folder for playgrounds as well, however, we now want to import the one we were working on, __MPS.playground__, so tap on it:

![alt text](https://github.com/MetalKit/images/raw/master/mps_3.PNG "3")

As soon as the `iPad` loaded the playground, we're in business! You can see the main playground page now on your `iPad`:

![alt text](https://github.com/MetalKit/images/raw/master/mps_4.PNG "4")

All you have to do now is tap on `Run My Code` and see our image with a nice blur filter applied to it:

![alt text](https://github.com/MetalKit/images/raw/master/mps_5.PNG "5")

"Ok," you might say, "but how do we see the entire image?" The answer is in the animated GIF below. It's as easy as long pressing in the middle of the screen until a screen separating line shows. With the finger still on the screen, drag the line to either the left side or the right side, depending on what do you need to see, your code or your output image:

![alt text](https://github.com/MetalKit/images/raw/master/mps_6.gif "6")

Now you can see that blurred image entirely! And when you're done admiring the image, tap that handy button on left side (and at the center) of the screen to go back to split screen. Before we wrap it up, one more piece of awesomeness. Replace the line below:

{% highlight swift %}let shader = MPSImageGaussianBlur(device: view.device!, sigma: 5)
{% endhighlight %}

with this line:

{% highlight swift %}let shader = MPSImageSobel(device: device)
{% endhighlight %}

Check out the new output image:

![alt text](https://github.com/MetalKit/images/raw/master/mps_7.PNG "7")

Imagine the possibilities you have here, by only changing one line of code! There are a couple dozen different shaders that you can try. Check out the [Metal Performance Shaders API](https://developer.apple.com/reference/metalperformanceshaders#symbols) for more details. If you are really into image processing, you might want to also check Simon Gladman's [Core Image for Swift](https://itunes.apple.com/us/book/core-image-for-swift/id1073029980?mt=13) book. If you want to learn more about the `Metal` backend of `MPS`, also check out Warren Moore's [Metal by Example](https://gum.co/metalbyexample) book. The [source code](https://github.com/MetalKit/mps) for the playground in this article is posted on Github as usual.

Until next time!