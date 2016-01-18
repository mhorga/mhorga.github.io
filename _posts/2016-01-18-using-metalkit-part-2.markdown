---
published: true
title: Using MetalKit part 2
layout: post
---
In the first part of this series we introduced the MetalKit framework. Let's reuse the project from part one and pick up where we left off. If you look again at the __render()__ function, it was looking like this:

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

Let's improve the code above a bit. First, since our class already subclasses __MTKView__, it already has its own `device` so there is no need to declare another one. This lets us reduce the first two lines to just one:

{% highlight swift %} 
device = MTLCreateSystemDefaultDevice()
{% endhighlight %}

Second, last week we said that we should make sure __currentDrawable__ and __currentRenderPassDescriptor__ are not nil or the app will crash. For the sake of simplicity, we have not done that in part one, so let's do that now. This will also help us get rid of a couple more lines of code. The final version of the function will look like this:

{% highlight swift %} 
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
{% endhighlight %}

You might wonder what __colorAttachments[0]__ means. To set the `rendering pipeline state`, the `Metal` framework provides __3__ types of attachments that we can write to:

- colorAttachments	
- depthAttachmentPixelFormat
- stencilAttachmentPixelFormat

We are only interested in storing color data for now and `colorAttachments` is an array of textures that hold drawings results and display them on the screen. We currently only have one such texture - the first element (at index `0`) of the array. Ok, now is a good time to run the app and make sure you are still seeing the same colored background we saw last time. So much better! With only __9__ lines of code we can get safe `Metal` code running on our GPU. Not too shabby.

So far so good! Next, let's dive into a new `Metal` concept - drawing geometry on the screen. All graphics tutorials such as those about `OpenGL` start with a `Hello, Triangle` type of program because a triangle is the simplest form of geometry that can be drawn on screen. It is a __2D graphics__ basic element and all the other objects in the world of graphics are composed of triangles, so this makes it a great place to start. Imagine the screen coordinate system having its axes running through the center of the screen which would have the coordinates __(0, 0)__. The edges of the screen would have values of __-1__ and __1__ respectively. Let's create an array of floats and a buffer to hold the vertex values for our triangle. Insert these lines right after initializing the `device`:

{% highlight swift %} 
let vertex_data:[Float] = [-1.0, -1.0, 0.0, 1.0,
                            1.0, -1.0, 0.0, 1.0,
                            0.0,  1.0, 0.0, 1.0]
let data_size = vertex_data.count * sizeof(Float)
let vertex_buffer = device!.newBufferWithBytes(vertex_data, length: data_size, options: [])
{% endhighlight %}

The vertices above are located in order: bottom left, bottom right and top center. We notice that each vertex uses __4__ floats for its coordinate. The first two are the __x__ and __y__ axes. The ones we do not use this time are: the third one is the `depth (Z-axis)` and the fourth one is the `W coordinate` making our coordinates `homogeneous`. We will talk about them in our next episode. Then we compute the size of this array to simply be the size of __12__ floats and finally we create the buffer based on the array and its size. Now that we have our vertexes stored, we need a way to send them to the `GPU` so it can display them on the screen. Let's look at the entire process (`pipeline`) that facilitates drawing graphics on the screen:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter03_1.png "1")

We have completed the first stage so far, storing the vertexes. You notice that the next stages require that we have a new construct named __shader__. A `shader` is the where programmers are allowed to interfere in the graphics pipeline with their custom functions. `Metal` provides a few types of shaders, however, today we only look at two of them: the __vertex shader__ which is responsible for the __location__ of our point, and the __fragment shader__ which is responsible for the __color__ of our point.

The `Metal` framework provides a function that we can call on the `device` to create a __Library__ of functions (`shaders`), so let's create it:

{% highlight swift %} 
let library = device!.newDefaultLibrary()!
let vertex_func = library.newFunctionWithName("vertex_func")
let frag_func = library.newFunctionWithName("fragment_func")
{% endhighlight %}

We create two new __Functions__ and point them to their corresponding shaders (which we will create later). The next step is to create a __Render Pipeline Descriptor__ which needs to know about our shaders:

{% highlight swift %} 
let rpld = MTLRenderPipelineDescriptor()
rpld.vertexFunction = vertex_func
rpld.fragmentFunction = frag_func
rpld.colorAttachments[0].pixelFormat = .BGRA8Unorm
{% endhighlight %}

You might wonder what __.BGRA8Unorm__ means. This setting configures the pixel format so that everything that goes through the render pipeline conforms to the same order (in this case `Blue`, `Green`, `Red`, `Alpha`) of color components as well as size (in this case an `8-bit` color value goes from `0` to `255`). The last stage is to create a __Render Pipeline State__ based on the above `descriptor`:

{% highlight swift %} 
var rps: MTLRenderPipelineState! = nil
do {
    try rps = device!.newRenderPipelineStateWithDescriptor(rpld)
} catch let error {
    self.print("\(error)")
}
{% endhighlight %}

Now let's get back to the two `shaders` we promised to create when we created the `Library`. For this, we need to create a new file in `Xcode`. Choose the __Metal File__ type, name it __Shaders.metal__ or something similar and click `Create`. You will immediately notice that the code does not resemble `Swift` much, and that is because the `Metal shading language` is based on `C++`. Let's add the code below:

{% highlight swift %} 
#include <metal_stdlib>
using namespace metal;

struct Vertex {
    float4 position [[position]];
};

vertex Vertex vertex_func(constant Vertex *vertices [[buffer(0)]], uint vid [[vertex_id]]) {
    return vertices[vid];
}

fragment float4 fragment_func(Vertex vert [[stage_in]]) {
    return float4(0.7, 1, 1, 1);
}
{% endhighlight %}

The code is pretty straightforward. We first create a `struct` named __Vertex__ that has only one member - an array of position arrays. We notice that the array is __float4__ which in the shading language is the same as the vertexes we created earlier with __4__ floats each. Then the two shaders follow: the __vertex_func__ shader which returns the __location__ of the current vertex, and the __fragment_func__ shader which returns the __color__ of the current vertex. We hardcoded a particular color value, but we could have added a `color` struct member to `Vertex` and set the color separately for each vertex.
    
If you run the app, you should see a triangle like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter03_2.png "2")

In the next part we will learn more about the `Metal shading language` as well as how `3D graphics` is rendered on the `GPUs`. The [source code](https://github.com/Swiftor/Metal/tree/master/ch03) is posted on Github as usual.

Until next time!