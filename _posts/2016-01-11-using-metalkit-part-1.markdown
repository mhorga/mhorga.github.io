---
published: false
title: Using MetalKit part 1
layout: post
---
The Metal framework was announced the [WWDC 2014](https://developer.apple.com/videos/wwdc2014/) as we mentioned last week. A year later, at the [WWDC 2015](https://developer.apple.com/videos/wwdc2015/), the new [MetalKit](https://developer.apple.com/library/ios/documentation/MetalKit/Reference/MTKFrameworkReference/index.html) framework was announced which brings a great deal of improvements and new features for `Metal`. Meet __MTKView__, a subclass of `NSView/UIView`. It comes with an embedded `Metal` Layer, and it also manages the framebuffer and its render target attachments, as well as takes care of the draw loop.

Let’s create a new `Cocoa Application` (since `iOS` simulators do not support `Metal`). Make sure only the `Swift` and `Use Storyboards` fields are selected. Next, let’s create a new class named __MetalView.swift__ of type `NSView` (for now). Once created, go to the storyboard and select the `View` under the `View Controller` as set its class to a `MetalView` type under `Identity Inspector` as seen in the image below. 

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter02_1.png "1")

Do the same for the `View Controller` and in `Identity Inspector` under `Class` delete `View Controller` because we’re not going to use it. Also delete `ViewController.swift` since we don’t need it anymore. Now go back to `MetalView.swift` and type __import MetalKit__. There are two ways we can prepare our class for drawing: either conform to the `MTKViewDelegate` protocol and implement its `drawInView(:)` method, or subclass `MTKView` and override its `drawRect(:)` method. We choose the latter, so go ahead and change the class type from `NSView` to `MTKView`, and create a new method named __render()__ that has the following content:

{% highlight swift %} 
func render() {
    let device = MTLCreateSystemDefaultDevice()!
    self.device = device
    let rpd = MTLRenderPassDescriptor()
    let bleen = MTLClearColor(red: 0, green: 0.5, blue: 0.5, alpha: 1)
    rpd.colorAttachments[0].texture = currentDrawable!.texture
    rpd.colorAttachments[0].clearColor = bleen
    rpd.colorAttachments[0].loadAction = .Clear
    let commandQueue = device.newCommandQueue()
    let commandBuffer = commandQueue.commandBuffer()
    let encoder = commandBuffer.renderCommandEncoderWithDescriptor(rpd)
    encoder.endEncoding()
    commandBuffer.presentDrawable(currentDrawable!)
    commandBuffer.commit()
}
{% endhighlight %}

Let’s walk over the code line by line. First, we create a __device__, and you learned how to do that last week. We set this device as our `view’s device` or otherwise it will stay `nil` and will crash the app. Optionally, we can modify the view’s `drawable` properties before drawing. Next, we create a __render pass descriptor__ which allows us to configure a render pass with the `current drawable’s texture` attached as its primary color. For fun, we create a nice color named __bleen__ which is half blue and half green. Next, we create a __command queue__ for the device, and use the queue to create a __command buffer__. Finally, we use the command buffer to create the __render command encoder__ to perform the draw calls. For each iteration of the draw loop, a new `MTLRenderPassDescriptor` object is made available when queried from `currentRenderPassDescriptor`. This object is created based on the `currentDrawable` object. Presentation is not handled by `MTKView` so it is our responsibility to first check that both `currentRenderPassDescriptor` and `currentDrawable` are not `nil` before we call the __presentDrawable(:)__ method on the view’s current drawable.

Let’s refer to the [Metal](https://developer.apple.com/metal/) documentation for more details. The `Metal` framework contains several objects:

- `device` - an abstraction of the `GPU` that consumes the rendering and compute commands from the command queue
- `command queue` - is a serial sequence of command buffers and establishes the order the stored commands will be executed
- `command buffer` - stores the translated commands from a command encoder. Metal notifies the application when the buffer finished executing the commands.
- `command encoder` - translates the `API` commands to `GPU` hardware commands - there are __3__ types of encoders: `render` (for graphics rendering), `compute` (for data parallel processing) and `blit` (for resource copy operations). For now we will only look at the `render command encoder`. 
- `states` (eg blend and depth)
- `shaders` (source code)
- `resources` (textures and data buffers)

We will talk about the last __3__ objects in the next episode of this series. All we care for now are the `device`, `queue`, `buffer` and `encoder`. The __Render Command Encoder (RCE)__ generates hardware commands for a single rendering pass, which means all of the rendering that is sent to a single `framebuffer` object (set of targets). If another framebuffer (set of targets) needs to be rendered, a new `RCE` needs to be created. The `RCE` specifies states for the `vertex` and `fragment` stages of the `graphics pipeline` (which we will talk about in the next episode) and it also interleaves `resources`, `state changes` and `draw calls`. A huge advantage of using `RCE` is the fact that there is no draw-time compilation; the app decides when the compilation and state validation occurs, thus offering the programmers great performance leverage.
 
Let’s return to our code now. Inside the __drawRect(:)__ method call the `render()` method:

{% highlight swift %} 
override func drawRect(dirtyRect: NSRect) {
    super.drawRect(dirtyRect)
    render()
}
{% endhighlight %}

If you run the app, you will see a nice, nothing out of ordinary, `bleen-ish` screen like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter02_2.png "2")

In the next episode we will finally get to introducing `shaders`, talk about loading `textures` and about managing `model data`. The [source code](https://github.com/Swiftor/Metal/tree/master/ch02) is posted on `Github` as usual.

Until next time!