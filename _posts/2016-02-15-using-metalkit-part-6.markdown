---
published: true
title: Using MetalKit part 6
layout: post
---
Few days ago I discussed with my `Metal` mentor and friend, [Warren](https://gum.co/metalbyexample), about the greatness of using the `MTKView` and since I haven't covered alternatives to using it, I feel like I owe it to our readers to explain what are the differences between using the newer `MetalKit` framework as opposed to using the earlier `Metal` framework. They are both still coexisting, however, `MetalKit` introduces a few powerful features such as: 

- Easy `texture` loading (even asynchronous loading with a few lines of code).
- Efficient data transfer between `Model I/O` meshes and `Metal` buffers.
- `MTKView` - a convenient `Metal-aware` view (we'll look at it in more detail).

I will start by reminding you how our first program looked like in chapter 1:

{% highlight swift %} 
import MetalKit

class MetalView: MTKView {

    override func drawRect(dirtyRect: NSRect) {
        super.drawRect(dirtyRect)
        render()
    }
 
    func render() {
        device = MTLCreateSystemDefaultDevice()
        if let rpd = currentRenderPassDescriptor, drawable = currentDrawable {
            rpd.colorAttachments[0].clearColor = MTLClearColorMake(0, 0.5, 0.5, 1.0)
            let command_buffer = device!.newCommandQueue().commandBuffer()
            let command_encoder = command_buffer.renderCommandEncoderWithDescriptor(rpd)
            command_encoder.endEncoding()
            command_buffer.presentDrawable(drawable)
            command_buffer.commit()
        }
    }
}
{% endhighlight %}

That was it! Simple and elegant way to clear your background color to a custom color of your choice. Now let's switch to using the `Metal` framework which does not have the `MTKView` so we will need to subclass `NSView` (or `UIView` in `iOS`) instead.

{% highlight swift %} 
import Cocoa

class MetalView: NSView {
{% endhighlight %}

You will immediately notice a few errors signaling that the following properties are not provided to us by default anymore:

- a device
- a drawable
- a render pass descriptor

Let's fix that. First, since `NSView` is not `Metal-aware` we need to create a `CAMetalLayer` and tell `NSView` to use it as its backing layer. `CAMetalLayer` is a `Core Animation` layer that manages a pool of textures for rendering its content. To use `Metal` for rendering, we need to use this class as the backing layer for our view by returning it from your view’s __layerClass()__ class method.:

{% highlight swift %} 
override class func layerClass() -> AnyClass {
    return CAMetalLayer.self
}

var metalLayer: CAMetalLayer {
    return self.layer as! CAMetalLayer
}
{% endhighlight %}

Next, inside the __render()__ function create a new `device` and tell __metalLayer__ that it owns it, and also set the pixel format the layer will use. Then, create a `drawable`. Notice that we are not using a `currentDrawable` that was provided with the `MTKView`. Rather, `CAMetalLayer` provides a __nextDrawable__ for us to use. Finally, create a render pass descriptor. Again, notice that we are not provided with a `currentRenderPassDescriptor` either:

{% highlight swift %}
let device = MTLCreateSystemDefaultDevice()!
metalLayer.device = device
metalLayer.pixelFormat = .BGRA8Unorm
let drawable = metalLayer.nextDrawable()
let texture = drawable!.texture
let rpd = MTLRenderPassDescriptor() 
{% endhighlight %}

Before ending this episode, let's look at the __MTKView__ class to see once again why this is the preferred way of using `Metal` for rendering content in our apps:

{% highlight swift %} 
@available(OSX 10.11, *)
public class MTKView : NSView, NSCoding {
    public init(frame frameRect: CGRect, device: MTLDevice?)
    public init(coder: NSCoder)
    weak public var delegate: MTKViewDelegate?
    public var device: MTLDevice?
    public var currentDrawable: CAMetalDrawable? { get }
    public var framebufferOnly: Bool
    public var presentsWithTransaction: Bool
    public var colorPixelFormat: MTLPixelFormat
    public var depthStencilPixelFormat: MTLPixelFormat
    public var sampleCount: Int
    public var clearColor: MTLClearColor
    public var clearDepth: Double
    public var clearStencil: UInt32
    public var depthStencilTexture: MTLTexture? { get }
    public var multisampleColorTexture: MTLTexture? { get }
    public func releaseDrawables()
    public var currentRenderPassDescriptor: MTLRenderPassDescriptor? { get }
    public var preferredFramesPerSecond: Int
    public var enableSetNeedsDisplay: Bool
    public var autoResizeDrawable: Bool
    public var drawableSize: CGSize
    public var paused: Bool
    public func draw()
}

@available(OSX 10.11, *)
public protocol MTKViewDelegate : NSObjectProtocol {
    public func mtkView(view: MTKView, drawableSizeWillChange size: CGSize)
    public func drawInMTKView(view: MTKView)
}
{% endhighlight %}

Among the plethora of properties, notice the ones we were interested in particular: __device__, __currentDrawable__ and __currentRenderPassDescriptor__. Also worth mentioning, the class also provides a protocol for its __MTKViewDelegate__ property. To read more about each of these properties and functions, see the [MTKView](https://developer.apple.com/library/prerelease/ios/documentation/MetalKit/Reference/MTKView_ClassReference/index.html) reference documentation.

The [source code](https://github.com/Swiftor/Metal/tree/master/ch06) is posted on Github as usual.

Until next time!