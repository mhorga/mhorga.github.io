---
published: true
title: Ray tracing in a Swift playground part 6
layout: post
---
Today, I am looking again at the `Ray Tracing` project because I wanted to see how it runs in an `iPad Playground`. There aren't any changes in the core code for now, except I have updated it to run on iOS 10, Xcode 8, Swift 3 and the new iPad Playground app. 

If you run the playground, now you have the option to play with the __number of samples (ns)__ right on the main page of the playground. Fair warning, the higher you set that number, the longer it will take your playground to finish running but the higher the quality of the output image will be. The runtime will also increase if you make the `width` or `height` bigger. For `400 x 200` and `ns = 10`, you will get an image like this:

![alt text](https://github.com/MetalKit/images/raw/master/raytracing_01.png "1")

In order to get the image to show, you need to tap on the little icon that looks like an image, at the end of the line, and choose `Add viewer`. You can amp up the resolution as well to say, `800 x 400`, but this will also increase the running time of your playground. However, the output image is worth the waiting!

![alt text](https://github.com/MetalKit/images/raw/master/raytracing_02.png "2")

We will soon look at ways to make the playground run faster and generate higher quality output images. My good friend and scientific programming guru, [Jeff](https://twitter.com/hyperjeff/), is working on a `Metal`-based version of this raytracer. We will talk about that soon, too. The [source code](https://github.com/MetalKit/raytracing) for the playground in this article is posted on Github as usual.

Until next time!