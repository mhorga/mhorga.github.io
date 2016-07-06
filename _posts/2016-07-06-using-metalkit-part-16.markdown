---
published: false
title: Using MetalKit part 16
layout: post
---
A couple of weeks ago, at the `WWDC 2016`, the `Apple` engineers released a new document, the __Metal Best Practices Guide__ which includes useful information about organizing your code for better performance in your `Metal` apps. Because the documentation is quite extensive, we are just outlining the main concepts in this article. An efficient `Metal` app requires:

- Low `CPU` overhead.
- Optimal `GPU` performance.
- Continuous processor parallelism. 
- Effective resource management. 

>

## __1 Resource Management__

### _1.1 Persistent Objects_

__Best Practice__: Create persistent objects early and reuse them often.

- Initialize Your Device and Command Queue First
- Compile Your Functions and Build Your Library at Build Time
- Build Your Pipelines Once and Reuse Them Often
- Allocate Resource Storage Up Front

For more information, consult the [Persistent Objects](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/PersistentObjects.html) section of the documentation. 

### _1.2 Resource Options_

__Best Practice__: Set appropriate resource storage modes and texture usage options for your resources.

- Familiarize Yourself with Device Memory Models
- Choose an Appropriate Resource Storage Mode (iOS and tvOS)
- Choose an Appropriate Resource Storage Mode (OS X)
- Set Appropriate Texture Usage Flags

For more information, consult the [Resource Options](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/ResourceOptions.html) section of the documentation. 

### _1.3 Triple Buffering_

__Best Practice__: Implement a triple buffering model to update dynamic buffer data.

- Prevent Access Conflicts and Reduce Processor Idle Time
- Reduce Memory Overhead and Frame Latency
- Allow Time for Command Buffer Transactions
- Implement a Triple Buffering Model

For more information, consult the [Triple Buffering](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html) section of the documentation. 

### _1.4 Buffer Bindings_

__Best Practice__: Use an appropriate method to bind your buffer data to a graphics or compute function.

Metal provides several API options for binding buffer data to a graphics or compute function. The __setVertexBytes:length:atIndex:__ method is the best option for binding an amount of dynamic buffer data (a transient buffer) that is less than __4 KB__ to a vertex function. If the data size is larger than 4 KB, you should create a __MTLBuffer__ once and update its contents as needed. 

For more information, consult the [Buffer Bindings](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/BufferBindings.html) section of the documentation. 

>

## __2 Display Management__

### _2.1 Drawables_

__Best Practice__: Hold a drawable as briefly as possible.

- Use a MetalKit View to Acquire a Drawable

For more information, consult the [Drawables](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/Drawables.html) section of the documentation. 

### _2.2 Native Screen Scale (iOS and tvOS)_

__Best Practice__: Render at the exact pixel size of your target display.

- Use a MetalKit View to Support Native Screen Scale

For more information, consult the [Native Screen Scale](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/NativeScreenScale.html) section of the documentation. 

### _2.3 Frame Rate (iOS and tvOS)_

__Best Practice__: For apps that can't maintain a __60 FPS__ frame rate, present your drawables at a steady frame rate.

- Use the Display Link
- Adjust the Frame Interval
- Adjust the Drawable Presentation Time

For more information, consult the [Frame Rate](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/FrameRate.html) section of the documentation. 

>

## __3 Command Generation__

### _3.1 _



Until next time!