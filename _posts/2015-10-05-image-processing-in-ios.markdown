---
published: false
title: Image processing in iOS
layout: post
---
Last time we looked at animations in iOS. This time we will look into `image processing` and how it works in iOS. An image, in the most basic definition, is a 2-D (two dimensional) array of pixels which make the `width` and `height` of the image. Each pixel contains information about its `color` and `opacity`, so in a pixel data structure we would need to reserve memory for each of the 3 component colors (`red`, `green`, `blue`) as well as for the opacity (`alpha`) channel.