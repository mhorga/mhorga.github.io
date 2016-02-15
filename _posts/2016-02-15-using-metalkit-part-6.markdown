---
published: false
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

Let's fix that. First, since `NSView` is not `Metal-aware` we need to create a `CAMetalLayer` and tell `NSView` to use it as its backing store:

{% highlight swift %} 
var metalLayer: CAMetalLayer {
    return self.layer as! CAMetalLayer
}
{% endhighlight %}

Next, inside the __render()__ function create a new `device` and tell __metalLayer__ that it owns it.

{% highlight swift %}
let device = MTLCreateSystemDefaultDevice()
metalLayer.device = device 
{% endhighlight %}

Next

{% highlight swift %} 
{% endhighlight %}

Until next time!