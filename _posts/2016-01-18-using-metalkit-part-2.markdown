---
published: false
title: Using MetalKit part 2
layout: post
---
In the previous episode of this series we introduced the MetalKit framework.

func render() {

// 1. start here

    // last week we created a new device and set it as the view's device
    //      however, we can also instantiate MTKView's device directly
//        let device = MTLCreateSystemDefaultDevice()!
//        self.device = device
    device = MTLCreateSystemDefaultDevice()

// 3. do this part after the big IF statement below

    // the coordinate system has its axes running through the center of the screen
    //    the vertices below are located: bottom left, bottom right and top center
	//	  and are normalized on a -1 to 1 scale (more about it in the next episode)
    let vertex_data:[Float] = [-1.0, -1.0, 0.0, 1.0,
                                1.0, -1.0, 0.0, 1.0,
                                0.0,  1.0, 0.0, 1.0]
    let data_size = vertex_data.count * sizeofValue(vertex_data[0])
    let vertex_buffer = device!.newBufferWithBytes(vertex_data, length: data_size, options: [])
    let library = device!.newDefaultLibrary()!
    let vertex_func = library.newFunctionWithName("vertex_func")
    let frag_func = library.newFunctionWithName("fragment_func")
    let rpld = MTLRenderPipelineDescriptor()
    rpld.vertexFunction = vertex_func
    rpld.fragmentFunction = frag_func
	// what does this .BGRA8Unorm thing mean anyway?!
	// 		this configures the pixel format so that everything that goes through the render pipeline conforms to the same order (in this case Blue, Green, Red, Alpha) of color components as well as size (in this case an 8-bit color value goes from 0 to 255)
    rpld.colorAttachments[0].pixelFormat = .BGRA8Unorm
    var rps: MTLRenderPipelineState! = nil
    do {
        try rps = device!.newRenderPipelineStateWithDescriptor(rpld)
    } catch let error {
        self.print("\(error)")
    }

// 2. continue here after initializing device

    // last week we said we should make sure that `currentRenderPassDescriptor` and
    //      `currentDrawable` are not nil or the app will crash. let's do that now.
    if let rpd = currentRenderPassDescriptor, drawable = currentDrawable {
//            let rpd = MTLRenderPassDescriptor()
//            rpd.colorAttachments[0].texture = currentDrawable!.texture
//            rpd.colorAttachments[0].loadAction = .Clear
        rpd.colorAttachments[0].clearColor = MTLClearColorMake(0, 0.5, 0.5, 1.0)
    // what does this colorAttachments[0] thing mean anyway?!
	// 	 to specify the rendering pipeline state, the framework provides 3 types of attachments that we can write to:
	// 		colorAttachments 	//		depthAttachmentPixelFormat 	// 		stencilAttachmentPixelFormat 	//	right now we are only interested in storing color data. colorAttachments is an array of textures
	// 	that hold drawings results and display them on the screen. we currently only have one such
	// 	texture - the first element of the array.
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

// 4. do this part last

Shaders.metal

#include <metal_stdlib>
using namespace metal;

struct Vertex {
    float4 position [[position]];
};

vertex Vertex vertex_func(constant Vertex *vertices [[buffer(0)]], uint vid [[vertex_id]]) {
    return vertices[vid];
}

fragment float4 fragment_func(Vertex vert [[stage_in]]) {
    // we hardcode a color here but we could have added a `color` struct member
    return float4(0.7, 1, 1, 1);
}

Until next time!
