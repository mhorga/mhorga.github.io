---
published: false
title: Using MetalKit part 3
layout: post
---
In the previous part I promised we will learn more about the `Metal shading language`. Before we do that, first let's do some code cleaning and structuring since we are already getting into the habit of doing this from previous episodes. Start by downloading the [source code](https://github.com/Swiftor/Metal/tree/master/ch03) from the previous episode. We want to refactor the huge __render()__ function, to start with. So let's take the _vertex buffer_ and the _render pipeline state_ outside of the function, and also create __3__ new smaller functions, so that our old function reduces to this:

{% highlight swift %} 
var vertex_buffer: MTLBuffer!
var rps: MTLRenderPipelineState! = nil

func render() {
    device = MTLCreateSystemDefaultDevice()
    createBuffer()
    registerShaders()
    sendToGPU()
}
{% endhighlight %}

For the first new function, __createBuffer()__, we need to first make some changes. Remember from last episode that _vertex data_ was an array of type `Float` like this:

{% highlight swift %} 
let vertex_data:[Float] = [-1.0, -1.0, 0.0, 1.0,
                                    1.0, -1.0, 0.0, 1.0,
                                    0.0,  1.0, 0.0, 1.0]
{% endhighlight %}

Let's transform it into a better suited format, a __struct__ with two members of `vector_float4` type, one for __position__ and the other for __color__:

{% highlight swift %} 
struct Vertex {
    var position: vector_float4
    var color: vector_float4
};
{% endhighlight %}

You are probably wondering what kind of a data type __vector_float4__ is. From `Apple`'s documentation we find that the vector types are clang-based types that are better suited than traditional `SIMD` types for arithmetic operations of type vector-vector and vector-scalar. It is possible to access vector components both via array-style subscripting, and by using the __.__ operator with component names (`x`, `y`, `z`, `w`, or combinations thereof). Besides the __.xyzw__ component names, the following sub-vectors can be easily accessed: __.lo__ / __.hi__ (first half and second half of a vector), as well as the __.even__ / __.odd__ sub-vectors:

{% highlight swift %} 
vector_float4 x = 1.0f;         // x = { 1, 1, 1, 1 }.
vector_float3 y = { 1, 2, 3 };  // y = { 1, 2, 3 }.
x.xyz = y.zyx;                  // x = { 1/3, 1/2, 1, 1 }.
x.w = 0;                        // x = { 1/4, 1/3, 1/2, 0 }.
{% endhighlight %}

Let's get back to `createBuffer()` so we can change our __vertex_data__ using the new `struct`:

{% highlight swift %} 
func createBuffer() {
    let vertex_data = [Vertex(position: [-1.0, -1.0, 0.0, 1.0], color: [1, 0, 0, 1]),
                       Vertex(position: [ 1.0, -1.0, 0.0, 1.0], color: [0, 1, 0, 1]),
                       Vertex(position: [ 0.0,  1.0, 0.0, 1.0], color: [0, 0, 1, 1])]
    vertex_buffer = device!.newBufferWithBytes(vertex_data, length: sizeof(Vertex) * 3, options:[])
}
{% endhighlight %}

You notice how handy is to have it as an array of structs where we can easily create vertexes in place. You also notice we kept the vertex positions as they were last time, and we added separate colors for each vertex (red, green and blue). Next up, is the __registerShaders()__ function. We don't change anything to the old code other than having it moved to this new place:

{% highlight swift %} 
func registerShaders() {
    let library = device!.newDefaultLibrary()!
    let vertex_func = library.newFunctionWithName("vertex_func")
    let frag_func = library.newFunctionWithName("fragment_func")
    let rpld = MTLRenderPipelineDescriptor()
    rpld.vertexFunction = vertex_func
    rpld.fragmentFunction = frag_func
    rpld.colorAttachments[0].pixelFormat = .BGRA8Unorm
    do {
        try rps = device!.newRenderPipelineStateWithDescriptor(rpld)
    } catch let error {
        self.print("\(error)")
    }
}
{% endhighlight %}

And lastly, we do the same with the __sendToGPU()__ function, nothing being changed to the old code other than having it moved to this new place:

{% highlight swift %} 
func sendToGPU() {
    if let rpd = currentRenderPassDescriptor, drawable = currentDrawable {
        rpd.colorAttachments[0].clearColor = MTLClearColorMake(0.5, 0.5, 0.5, 1.0)
        let command_buffer = device!.newCommandQueue().commandBuffer()
        let command_encoder = command_buffer.renderCommandEncoderWithDescriptor(rpd)
        command_encoder.setRenderPipelineState(rps)
        command_encoder.setVertexBuffer(vertex_buffer, offset: 0, atIndex: 0)
        command_encoder.drawPrimitives(.Triangle, vertexStart: 0, vertexCount: 3, instanceCount: 1)
        command_encoder.endEncoding()
        command_buffer.presentDrawable(drawable)
        command_buffer.commit()
    }
}
{% endhighlight %}

Let's move on to the __Shaders.metal__ file next. We do two modifications here. First, we add a __color__ member to our `Vertex` struct so we can pass it back and forth between the `CPU` and the `GPU`:

{% highlight swift %} 
struct Vertex {
    float4 position [[position]];
    float4 color; 
};
{% endhighlight %}

and second, we replace the hardcoded color we used last time in the __fragment__ shader:

{% highlight swift %} 
fragment float4 fragment_func(Vertex vert [[stage_in]]) {
    return float4(0.7, 1, 1, 1);
}
{% endhighlight %}

with the actual color each vertex carries (sent to the `GPU` from the __vertex_buffer__):

{% highlight swift %} 
fragment float4 fragment_func(Vertex vert [[stage_in]]) {
    return vert.color;
}
{% endhighlight %}
    
If you run the app, you should now see a more nicely colored triangle:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter04.png "1")

You might be wondering why are the colors becoming gradients as we move away from the three vertexes we passed to the shaders? To understand this, it's important to first understand the difference between the two shaders and their role in the graphics pipeline.

The [source code](https://github.com/Swiftor/Metal/tree/master/ch04) is posted on Github as usual.

Until next time!