---
published: true
title: Using MetalKit part 8
layout: post
---
By now, you might have all noticed I am in love with `Swift`. I am also an avid user of `Xcode` playgrounds. This week we will take our `Metal` code and put it into a playground. Huzzah, for `Metal` prototyping in the playgrounds!

Let's start by creating a new `Xcode` playground for `OS X`. Once created, click the `Show the Assistant editor` button and also press `Command + 1` to show the `Project navigator` as well. Your playground should look like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter08_1.png "1")

First thing we do is create __Shaders.metal__ under the __Resources__ folder in the `Project navigator` with the same source code from the previous episode of this series. Next, we create __MathUtils.swift__ and __MetalView.swift__ under the __Sources__ folder. The only change we make in `MathUtils.swift` is we create a handy initializer for the `Vertex` struct:

{% highlight swift %} 
struct Vertex {
    var position: vector_float4
    var color: vector_float4
    init(pos: vector_float4, col: vector_float4) {
        position = pos
        color = col
    }
}
{% endhighlight %}

In `MetalView.swift` we need to make more changes though. First, we need to make the class __public__ as we will call it from outside the `Sources` folder. Consequently, the initializer and the __drawRect(:)__ method need to become `public` as well. Also, we will create a second initializer so we can create a `MetalView` with a given __frame__: 

{% highlight swift %} 
public class MetalView: MTKView {
    ... 
    required public init(coder: NSCoder) {
        super.init(coder: coder)
    }

    override public init(frame frameRect: CGRect, device: MTLDevice?) {
        super.init(frame: frameRect, device: device)
        createBuffers()
        registerShaders()
    }
    ... 
}
{% endhighlight %}

Next, we need to make a rather curious change because as you notice, creating a `Library`: 

{% highlight swift %} 
let library = device.newDefaultLibrary()!
{% endhighlight %}

fails with this error message:

{% highlight text %} 
MTLLibrary.mm:1016: failed assertion `filepath must not be nil.'
{% endhighlight %}

Since the playground does not have a default `filepath` that we could use, we will need to create our own: 

{% highlight swift %} 
func registerShaders() {
    let path = NSBundle.mainBundle().pathForResource("Shaders", ofType: "metal")
    let input: String?
    let library: MTLLibrary
    let vert_func: MTLFunction
    let frag_func: MTLFunction
    do {
        input = try String(contentsOfFile: path!, encoding: NSUTF8StringEncoding)
        library = try device!.newLibraryWithSource(input!, options: nil)
        vert_func = library.newFunctionWithName("vertex_func")!
        frag_func = library.newFunctionWithName("fragment_func")!
        let rpld = MTLRenderPipelineDescriptor()
        rpld.vertexFunction = vert_func
        rpld.fragmentFunction = frag_func
        rpld.colorAttachments[0].pixelFormat = .BGRA8Unorm
        rps = try device!.newRenderPipelineStateWithDescriptor(rpld)
    } catch let e {
        Swift.print("\(e)")
    }
}
{% endhighlight %}

Notice that we tell the playground to find a path where a resource named `Shaders` of type `metal` exists. Next, we convert that file into a large `String` and then we create the library from this source. 

Finally, we go to the playground's main page and create a new `MetalView` with a given frame. Then we tell the playground to present us the live view: 

{% highlight swift %} 
import Cocoa
import XCPlayground

let device = MTLCreateSystemDefaultDevice()!
let frame = NSRect(x: 0, y: 0, width: 300, height: 300)
let view = MetalView(frame: frame, device: device)
XCPlaygroundPage.currentPage.liveView = view
{% endhighlight %}

If you are showing the `Timeline` in the `Assistant editor` you should have a similar view:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter08_2.png "2")

The [source code](https://github.com/Swiftor/Metal/tree/master/ch08) is posted on Github as usual.

Until next time!