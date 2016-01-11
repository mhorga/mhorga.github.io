---
published: false
title: Using MetalKit part 1
layout: post
---
In the previous episode of this series we noted that the Metal framework was announced the WWDC 2014. A year later, at the WWDC 2015, the new MetalKit framework is announced which brings a great deal of improvements and new features for Metal. Meet __MTKView__, a subclass of NSView/UIView. In comes with an embedded CAMetalLayer, and it also manages the framebuffer and its render target attachments, as well as takes care of the draw loop.

Let’s create a new Cocoa Application (since iOS simulators do not support Metal). Make sure only the Swift and Use Storyboards fields are selected. Next let’s create a new class named __MetalView__ of type NSView (for now). Once created, go to the storyboard and select the View under the View Controller as set its class to a __MetalView__ type under Identity Inspector as seen in the image below. 

Do the same for the View Controller and in Identity Inspector under Class delete View Controller because we’re not going to use it. Also delete the __ViewController.swift__ class since we don’t need it anymore. Now go back to the __MetalView__ class and import __MetalKit__. There are two ways we can prepare our class for drawing: either conform to the MTKViewDelegate protocol and implement its drawInView(:) method, or subclass MTKView and override its __drawRect(:)__ method. We choose the latter, so go ahead and change the class type from NSView to MTKView, and create a new method named __render()__ that has the following content:

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

Let’s walk over the code line by line. First, we created a device, and you learned how to do that last week. We set this device as our view’s device or otherwise it will stay nil and will crash the app. Optionally, we can modify the view’s drawable properties before drawing. Next, we create a render pass descriptor which allows us to configure a render pass with the current drawable’s texture attached as its primary color. For fun, we create a nice color named bleen which is half blue and half green. Next, we create a command queue for the device, and use the queue to create a command buffer. Finally, we use the command buffer to create the render command encoder to perform the draw calls. For each iteration of the draw loop, a new MTLRenderPassDescriptor object is made available when queried from currentRenderPassDescriptor. This object is created based on the currentDrawable object. Presentation is not handled by MTKView so it is our responsibility to first check that both currentRenderPassDescriptor and currentDrawable are not nil before we call the __presentDrawable(:)__ method on the view’s current drawable.

Let’s refer to the Metal documentation for more details. As you can see in the image below, the Metal framework contains several objects:

	•	device - an abstraction of the GPU that consumes the rendering and compute commands from the command queue
	•	command queue - is a serial sequence of command buffers and establishes the order the stored commands will be executed
	•	command buffer - stores the translated commands from a command encoder. Metal notifies the application when the buffer finished executing the commands.
	•	command encoder - translates the API commands to GPU hardware commands - there are 3 types of encoders: render (for graphics rendering), compute (for data parallel processing) and blit (for resource copy operations). for now we will only look at the render command encoder. 
	•	states (eg blend and depth)
	•	shaders (source code)
	•	resources (textures and data buffers)

We will talk about the last 3 objects in the next episode of this series. All we care for now are the device, queue, buffer and encoder. You can see how they all fit in this image below (System - left side layer of the Metal stack):

The Render Command Encoder (RCE) generates hardware commands for a single rendering pass, which means all of the rendering that is sent to a single framebuffer object (set of targets). If another framebuffer (set of targets) needs to be rendered, a new RCE needs to be created. The RCE specifies states for the vertex and fragment stages of the graphics pipeline (which we will talk about in the next episode) and it also interleaves resources, state changes and draw calls. A huge advantage of using RCE is the fact that there is no draw-time compilation; the app decides when the compilation and state validation occurs, thus offering the programmers great performance leverage.
 
Let’s return to our code now. Inside the __drawRect(:)__ method call the __render()__ method:

override func drawRect(dirtyRect: NSRect) {
    super.drawRect(dirtyRect)
    render()
}

If you run the app, you will see a nice, nothing out of ordinary, `bleen-ish` screen like this:

In the next episode we will finally get to introducing shaders, talk about loading textures and about managing model data. The source code is posted on Github as usual.

Until next time!
