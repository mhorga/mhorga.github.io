---
published: true
title: Using MetalKit part 16
layout: post
---
A couple of weeks ago, at the `WWDC 2016`, the `Apple` engineers released a new document, the __Metal Best Practices Guide__ which includes useful information about organizing your code for better performance in your `Metal` apps. Because the documentation is quite extensive, we are just going to outline the main concepts in this article. An efficient `Metal` app requires:

- Low `CPU` overhead.
- Optimal `GPU` performance.
- Continuous processor parallelism. 
- Effective resource management. 


## __1 Resource Management__

### _1.1 Persistent Objects_

__Best Practice__: Create persistent objects early and reuse them often.

The `Metal` framework provides several protocols to manage persistent objects throughout the lifetime of your app. These objects are expensive to create but are usually initialized once and reused often. Do not create these objects at the beginning of every render or compute loop.

- Initialize Your Device and Command Queue First
- Compile Your Functions and Build Your Library at Build Time
- Build Your Pipelines Once and Reuse Them Often
- Allocate Resource Storage Up Front

For more information, consult the [Persistent Objects](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/PersistentObjects.html) section of the documentation. 

### _1.2 Resource Options_

__Best Practice__: Set appropriate resource storage modes and texture usage options for your resources.

`Metal` resources must be configured appropriately to take advantage of fast memory access and driver performance optimizations. Resource storage modes allow you to define the storage location and access permissions for your `MTLBuffer` and `MTLTexture` objects. Texture usage options allow you to explicitly declare how you intend to use your `MTLTexture` objects.

- Familiarize Yourself with Device Memory Models
- Choose an Appropriate Resource Storage Mode (`iOS` and `tvOS`)
- Choose an Appropriate Resource Storage Mode (`OS X`)
- Set Appropriate Texture Usage Flags

For more information, consult the [Resource Options](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/ResourceOptions.html) section of the documentation. 

### _1.3 Triple Buffering_

__Best Practice__: Implement a triple buffering model to update dynamic buffer data.

Dynamic buffer data refers to frequently-updated data stored in a buffer. To avoid creating new buffers per frame and to minimize processor idle time between frames, implementing a triple buffering model is strongly recommended.

- Prevent Access Conflicts and Reduce Processor Idle Time
- Reduce Memory Overhead and Frame Latency
- Allow Time for Command Buffer Transactions
- Implement a Triple Buffering Model

For more information, consult the [Triple Buffering](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html) section of the documentation. 

### _1.4 Buffer Bindings_

__Best Practice__: Use an appropriate method to bind your buffer data to a graphics or compute function.

`Metal` provides several `API` options for binding buffer data to a graphics or compute function. The __setVertexBytes:length:atIndex:__ method is the best option for binding an amount of dynamic buffer data (a transient buffer) that is less than __4 KB__ to a vertex function. If the data size is larger than 4 KB, you should create a __MTLBuffer__ once and update its contents as needed. 

For more information, consult the [Buffer Bindings](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/BufferBindings.html) section of the documentation. 


## __2 Display Management__

### _2.1 Drawables_

__Best Practice__: Hold a drawable as briefly as possible.

The command buffer is used to schedule a drawable's presentation with the __presentDrawable:__ method before the command buffer itself is scheduled for execution, however, the drawable itself is actually presented after the command buffer has completed execution.

- Use a MetalKit View to Acquire a Drawable

For more information, consult the [Drawables](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/Drawables.html) section of the documentation. 

### _2.2 Native Screen Scale (iOS and tvOS)_

__Best Practice__: Render at the exact pixel size of your target display.

The pixel size of your drawables should always match the exact pixel size of their target display. This is critical to avoid rendering to off-screen pixels or incurring an additional sampling stage.

- Use a MetalKit View to Support Native Screen Scale

For more information, consult the [Native Screen Scale](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/NativeScreenScale.html) section of the documentation. 

### _2.3 Frame Rate (iOS and tvOS)_

__Best Practice__: For apps that can't maintain a __60 FPS__ frame rate, present your drawables at a steady frame rate.

The display refresh rate of `iOS` devices is `60 Hz`. Apps that are consistently unable to complete a frame's work within this time should target a lower frame rate to avoid jitter. The display refresh rate of `tvOS` devices is usually, but not always, `60 Hz`.

- Use the Display Link
- Adjust the Frame Interval
- Adjust the Drawable Presentation Time

For more information, consult the [Frame Rate](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/FrameRate.html) section of the documentation. 


## __3 Command Generation__

### _3.1 Load and Store Actions_

__Best Practice__: Set appropriate load and store actions for your render targets.

Actions performed on your `Metal` render targets must be configured appropriately to avoid costly and unnecessary rendering work at the start (load action) or end (store action) of a rendering pass.

- Choose an Appropriate Load Action
- Choose an Appropriate Store Action
- Evaluate Actions Between Rendering Passes

For more information, consult the [Load and Store Actions](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/LoadandStoreActions.html) section of the documentation. 

### _3.2 Render Command Encoders (iOS and tvOS)_

__Best Practice__: Merge render command encoders when possible.

Eliminating unnecessary render command encoders reduces memory bandwidth and increases performance. 

- Evaluate Rendering Pass Order
- Evaluate Sampling Dependencies
- Evaluate Actions Between Rendering Passes

For more information, consult the [Render Command Encoders](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/RenderCommandEncoders.html) section of the documentation. 

### _3.3 Command Buffers_

__Best Practice__: Submit the fewest command buffers per frame without underutilizing the `GPU`.

Command buffers are the unit of work submission in `Metal`; they are created by the `CPU` and executed by the `GPU`. This relationship allows you to balance `CPU` and `GPU` work by adjusting the number of command buffers submitted per frame.

For more information, consult the [Command Buffers](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/CommandBuffers.html) section of the documentation. 

### _3.4 Indirect Buffers_

__Best Practice__: Use indirect buffers if your draw or dispatch call arguments are dynamically generated by the `GPU`.

Indirect buffers are `MTLBuffer` objects with a specific data layout representing draw or dispatch call arguments. 

- Eliminate Unnecessary Data Transfers and Reduce Processor Idle Time

For more information, consult the [Indirect Buffers](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/IndirectBuffers.html) section of the documentation. 


## __4 Compilation__

### _4.1 Functions and Libraries_

__Best Practice__: Compile your functions and build your library at build time.

Compiling `Metal Shading Language` source code is one of the most expensive stages in a `Metal` app. `Metal` is designed to minimize this cost by allowing you to compile graphics and compute functions at build time, then load them at runtime as a library.

- Build Your Library at Build Time
- Group Your Functions into a Single Library

For more information, consult the [Functions and Libraries](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/FunctionsandLibraries.html) section of the documentation. 

### _4.2 Pipelines_

__Best Practice__: Build your render and compute pipelines asynchronously.

Having multiple render or compute pipelines allows your app to use different state configurations for specific tasks. Building these pipelines asynchronously maximizes performance and parallelism. It is recommended that you build all known pipelines up front and avoid lazy loading. 

For more information, consult the [Pipelines](https://developer.apple.com/library/prerelease/content/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/Pipelines.html) section of the documentation. 


This guide, along with the [__Metal Programming Guide__](https://developer.apple.com/library/prerelease/content/documentation/Miscellaneous/Conceptual/MetalProgrammingGuide/Introduction/Introduction.html) and the [__Metal Shading Language Guide__](https://developer.apple.com/library/prerelease/content/documentation/Metal/Reference/MetalShadingLanguageGuide/Introduction/Introduction.html) both already updated for `iOS 10`, `tvOS 10` and `OS X 10.12`, give you the trilogy of documents containing everything you need to start creating performant `Metal` apps. 

Until next time!