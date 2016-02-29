---
published: true
title: Using MetalKit part 7
layout: post
---
One of our readers contacted me about an apparently weird behavior he was seeing: when running the code from our tutorial the `MTLLibrary` returned nil after a few hundred draw calls. That made me realize I was not taking into consideration the fact that some of the `Metal` objects are transient and some are not, according to the [Metal documentation](http://apple.co/1KPOIsX). Thanks __Mike__ for bringing this to my attention!

To deal with this matter, we need to re-organize the code, again! And that is a good thing to always do. We need to get the non-transient `Metal` objects (devices, queues, data buffers, textures, states and pipelines) out of the __drawRect(_:)__ method and put them in a method that only runs once when the view loads. The command buffers and encoders are the only two transient objects designed for a single use, so we can thus create them with each draw call.

We will pick up where we left off in [part 5](https://github.com/Swiftor/Metal/tree/master/ch05) of the series. To start, let's first create a new method -- an initializer -- that only runs once when the view is loaded:

{% highlight swift %} 
required init(coder: NSCoder) {
    super.init(coder: coder)
    device = MTLCreateSystemDefaultDevice()
    createBuffers()
    registerShaders()
}
{% endhighlight %}

Next, delete the __render()__ method as well as its call inside `drawRect(_:)` as we don't need it anymore. Then move all the code from __sendToGPU()__ to `drawRect(_:)` and delete `sendToGPU()` as we don't need this one either. 

{% highlight swift %} 
override func drawRect(dirtyRect: NSRect) {
    super.drawRect(dirtyRect)
    if let rpd = currentRenderPassDescriptor, drawable = currentDrawable {
        rpd.colorAttachments[0].clearColor = MTLClearColorMake(0.5, 0.5, 0.5, 1.0)
        let command_buffer = device!.newCommandQueue().commandBuffer()
        let command_encoder = command_buffer.renderCommandEncoderWithDescriptor(rpd)
        command_encoder.setRenderPipelineState(rps)
        command_encoder.setVertexBuffer(vertex_buffer, offset: 0, atIndex: 0)
        command_encoder.setVertexBuffer(uniform_buffer, offset: 0, atIndex: 1)
        command_encoder.drawPrimitives(.Triangle, vertexStart: 0, vertexCount: 3, instanceCount: 1)
        command_encoder.endEncoding()
        command_buffer.presentDrawable(drawable)
        command_buffer.commit()
    }
}
{% endhighlight %}

Finally, let's create a new class named __MathUtils__ and move both `structs` to it so we have a cleaner view class.

{% highlight swift %} 
import simd

struct Vertex {
    var position: vector_float4
    var color: vector_float4
}

struct Matrix {
    var m: [Float]
    
    init() {
        m = [1, 0, 0, 0,
            0, 1, 0, 0,
            0, 0, 1, 0,
            0, 0, 0, 1
        ]
    }
    
    func translationMatrix(var matrix: Matrix, _ position: float3) -> Matrix {
        matrix.m[12] = position.x
        matrix.m[13] = position.y
        matrix.m[14] = position.z
        return matrix
    }
    
    func scalingMatrix(var matrix: Matrix, _ scale: Float) -> Matrix {
        matrix.m[0] = scale
        matrix.m[5] = scale
        matrix.m[10] = scale
        matrix.m[15] = 1.0
        return matrix
    }
    
    func rotationMatrix(var matrix: Matrix, _ rot: float3) -> Matrix {
        matrix.m[0] = cos(rot.y) * cos(rot.z)
        matrix.m[4] = cos(rot.z) * sin(rot.x) * sin(rot.y) - cos(rot.x) * sin(rot.z)
        matrix.m[8] = cos(rot.x) * cos(rot.z) * sin(rot.y) + sin(rot.x) * sin(rot.z)
        matrix.m[1] = cos(rot.y) * sin(rot.z)
        matrix.m[5] = cos(rot.x) * cos(rot.z) + sin(rot.x) * sin(rot.y) * sin(rot.z)
        matrix.m[9] = -cos(rot.z) * sin(rot.x) + cos(rot.x) * sin(rot.y) * sin(rot.z)
        matrix.m[2] = -sin(rot.y)
        matrix.m[6] = cos(rot.y) * sin(rot.x)
        matrix.m[10] = cos(rot.x) * cos(rot.y)
        matrix.m[15] = 1.0
        return matrix
    }
    
    func modelMatrix(var matrix: Matrix) -> Matrix {
        matrix = rotationMatrix(matrix, float3(0.0, 0.0, 0.1))
        matrix = scalingMatrix(matrix, 0.25)
        matrix = translationMatrix(matrix, float3(0.0, 0.5, 0.0))
        return matrix
    }
}
{% endhighlight %}

Run the program to make sure you are still seeing the glorious triangle as we have seen it in the previous part. We are now ready to move on to the next stage -- rendering 3D objects -- in the next part of this series. The [source code](https://github.com/Swiftor/Metal/tree/master/ch07) is posted on Github as usual.

Until next time!