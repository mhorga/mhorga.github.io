---
published: true
title: Using MetalKit part 3
layout: post
---
In the previous part I promised we will learn more about the `Metal shading language`. Before that, first let's do some code cleaning and structuring since we are already getting into the habit of doing this from previous episodes. Start by downloading the [source code](https://github.com/Swiftor/Metal/tree/master/ch03) from the previous episode. We want to refactor the huge __render()__ function, to start with. So let's take the _vertex buffer_ and the _render pipeline state_ outside of the function, and also create __3__ new smaller functions, so that our old function reduces to this:

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

For the __createBuffer()__ function we need to first make some changes. Recall from last episode that _vertex data_ was an array of type `Float` like this:

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

You might be wondering why are the colors becoming gradients as we move away from the three vertexes we passed to the shaders? To understand this, it's important to first understand the difference between the two shaders and their role in the graphics pipeline. Let's look at the syntax for writing any shader (we choose the vertex shader as example):

{% highlight swift %} 
vertex Vertex vertex_func(constant Vertex *vertices [[buffer(0)]], uint vid [[vertex_id]])
{% endhighlight %}

The first keyword is the function __qualifier__ and can only have the value __vertex__, __fragment__ or __kernel__. The next keyword is the __return type__. Next is the function __name__ followed by the function __arguments__ inside the parentheses. The `Metal` shading language restricts the use of pointers unless the arguments are declared with the __device__, __threadgroup__, or __constant__ address space qualifier which specifies the region of memory where a function variable or argument is allocated. The __[[ ... ]]__ syntax is used to declare attributes such as resource locations, shader inputs, and built-in variables that are passed back and forth between shaders and CPU.

`Metal` uses the __[[ buffer(index) ]]__ attribute to identify the location for the `device` and `constant buffer` argument types. Built-in input and output variables are used to communicate values between the graphics (vertex and fragment) functions and the fixed-function graphics pipeline stages. In our case __[[ vertex_id ]]__ is the per-vertex identifier that facilitates the communication. Per-vertex inputs can also be passed as an argument to a vertex function by declaring them with the __[[ stage_in ]]__ attribute qualifier. 

The `vertex shader` takes a pointer to the list of vertices as the 1st parameter. We will be able to index into __vertices__ using the 2nd parameter __vid__ which is attributed with __vertex_id__ that tells `Metal` to insert the vertex index currently being processed as this parameter. We then simply pass along each vertex (with its position and color) for the `fragment shader` to consume. All the `fragment shader` does is to take the vertex passed from the `vertex shader` and pass through the color for each and every pixel without changing anything to the input data. The vertex shader runs infrequently (only 3 times in this case - for each vertex), while the `fragment shader` runs thousands of times - for each pixel it needs to draw. 

So you might still asking: "Ok, but what about the color gradients"? Well, now that you understand what each shader does and how often they run, you can think about the color at any given pixel as the __average__ color value of its neighbors. For example, the color halfway between the the `red` and the `green` pixel will be `yellow` simply because the `fragment shader` interpolates the two colors by average: _0.5 * red + 0.5 * green__. The same happens with the color halfway between `red` and `blue` resulting `magenta`, as well as halfway between `blue` and `green` resulting `cyan`. From here on, the rest of the pixels are interpolated with unequal parts of the primary colors resulting the gradient range you see.

The [source code](https://github.com/Swiftor/Metal/tree/master/ch04) is posted on Github as usual.

Until next time!