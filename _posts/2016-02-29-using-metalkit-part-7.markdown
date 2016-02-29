---
published: false
title: Using MetalKit part 7
layout: post
---
One of our readers contacted me about an apparently weird behavior he was seeing: when running his program the Library returned nil after a few hundred draw calls. That made me realize I was not taking into consideration the fact that some of the Metal objects are transient and some are not, according to the [Metal documentation](http://apple.co/1KPOIsX). Thanks Mike for bringing this to my attention!

To deal with this matter, we need to re-organize the code, again! And that is a good thing to always do. We need to get the non-transient Metal objects (devices, queues, data buffers, textures, states and pipelines) out of the drawRect method and create properties for each of them. The command buffers and encoders are the only two transient objects designed for a single use, so we can thus create them with each draw call.