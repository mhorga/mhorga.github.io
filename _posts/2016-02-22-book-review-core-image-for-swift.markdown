---
published: true
title: Book review Core Image for Swift
layout: post
---
I am taking a break from the `MetalKit` series this week because there is an important event I couldn’t miss it for the world! My good pal, [@FlexMonkey](https://twitter.com/FlexMonkey), just published his book titled [Core Image for Swift](https://itunes.apple.com/us/book/core-image-for-swift/id1073029980) and I had the privilege of being one of its early reviewers. `Core Image` is a really good "friend" of `Metal` so it makes sense writing about it.

`Core Image for Swift` is a guide to `Apple’s Core Image` framework and it teaches you everything about image processing through `Swift` projects and playgrounds in `Xcode`. A great feature offered by the author is that the `Swift` code can be entirely copy-pasted from the book right into `Xcode` without having to download anything (except a playground in Chapter 2) or to even build any storyboards.

From the beginning, the book goes right on topic and touches briefly on concepts such as context, images, filters and face/shape detection. Another feature I liked a lot is that the author offers reference links for adjacent concepts used in the book (eg. `EAGL` or `GLKit`), so if you feel like reading more about them, you can pause your reading here and follow those links.

As we progress through the book, we are diving deeper and deeper into the `Core Image` concepts, by looking at filter categories and names, querying the system for filters and interrogating the filter for details. Another gem offered by the author is a reference to his wonderful and comprehensive collection of filters — [Filterpedia](https://github.com/FlexMonkey/Filterpedia) — which at the time of writing this article, is nearing a thousand stars on Github. The book is also accompanied by beautiful high-res images to illustrate the topics covered and to provide examples of filters applied to images.

The book then goes in-depth to explain techniques for displaying images in various setups such as working with `UIKit`, `OpenGL` and `Metal`. We are then showed how to create our own custom filters. Next, we are introduced to `kernels` written in the `Core Image shading language` and the benefit of using the `GPU` for image processing. Another great way of leveraging the tremendous power of the `GPU` is by writing your own custom `Metal kernels` or by even using the `Metal Performance Shaders` that are tuned to run optimally on each and every `GPU` family.

The author states oh [his website](https://flexmonkey.blogspot.co.uk) that this is the first of the two books in the `Core Image` series. This one focuses on working with still images. The second book will focus on working with video and with other frameworks (eg. SpriteKit). The book can be [pre-ordered now](https://itunes.apple.com/us/book/core-image-for-swift/id1073029980) and it will be available next week. 

Until next time!