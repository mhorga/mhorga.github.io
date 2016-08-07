---
published: false
title: Ray tracing in a Swift playground part 6
layout: post
---
Today, I am looking again at the `Ray Tracing` project because I wanted to see how it runs in an `iPad Playground`. There aren't any changes in the core code for now, except I have updated it to run on iOS 10, Xcode 8, Swift 3 and the new iPad Playground app. 

If you run the playground, now you have the option to play with the __number of samples (ns)__ right on the main page of the playground. Fair warning, the higher you set that number, the longer it will take your playground to finish running but the higher the quality of the output image will be. The runtime will also increase if you make the `width` or `height` bigger. For `400 x 200` and `ns = 10`, you will get an image like this:

