---
published: false
title: Using MetalKit part 12
layout: post
---
Today we continue working on that beautiful fractal, so let's pick up where we left off in [Part 11](http://mhorga.org/2016/05/10/using-metalkit-part-11.html). Using the same playground we worked on last time, we will next see how to bring it to life, that is, animate it. For that, we will use `uniforms` again. We introduced them in [Part 5](http://mhorga.org/2016/02/08/using-metalkit-part-5.html) in case you want to read again why they are useful in cases such as this one. 

First, at the top of the `MetalView.swift` file, let's create a global variable named __timer__ and a data buffer to hold it:

{% highlight swift %}var timer: Float = 0
var timerBuffer: MTLBuffer!
{% endhighlight %}

Next, inside `registerShaders()` we want to initialize the buffer:

{% highlight swift %}timerBuffer = device!.newBufferWithLength(sizeof(Float), options: [])
{% endhighlight %}

Then, we need to create an __update()__ function which will increase the timer and send its new value to the buffer:

{% highlight swift %}func update() {
    timer += 0.01
    var bufferPointer = timerBuffer.contents()
    memcpy(bufferPointer, &timer, sizeof(Float))
}
{% endhighlight %}

Next, in `drawRect()` right below the line where we set the texture to the current drawable, we need to also set the timer buffer at index __1__. Then we need to also call the `update` function since `drawRect()` runs every frame so the timer will have a bigger value as frames go by: 

{% highlight swift %}commandEncoder.setBuffer(timerBuffer, offset: 0, atIndex: 1)
update()
{% endhighlight %}

Then, in `Shaders.metal` we need to update our kernel signature to also include the timer buffer:

{% highlight swift %}kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    constant float &timer [[buffer(1)]],
                    uint2 gid [[thread_position_in_grid]])
{% endhighlight %}

Next, here comes the fun part! Replace the line below:

{% highlight swift %}float2 cc = 1.1*float2( 0.5*cos(0.1) - 0.25*cos(0.2), 0.5*sin(0.1) - 0.25*sin(0.2) );
{% endhighlight %}

with this line:

{% highlight swift %}float2 cc = 1.1*float2( 0.5*cos(0.1*timer) - 0.25*cos(0.2*timer), 0.5*sin(0.1*timer) - 0.25*sin(0.2*timer) );
{% endhighlight %}

If you run the playground right now, you should see something similar:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter12_1.gif "1")

There is another important and useful feature we could have, and that is mouse interaction. Obviously, we can use `uniforms` again. Let's conform our `MetalView` class to the `NSWindowDelegate` protocol so we can use its `mouse` methods.

{% highlight swift %}public class MetalView: MTKView, NSWindowDelegate {
{% endhighlight %}

Next, let's create again, like we did for the timer, a global variable named __pos__ for the mouse position (coordinates) and a data buffer to hold it. We can now override the __mouseDown()__ method and work with the coordinates:

{% highlight swift %}var mouseBuffer: MTLBuffer!
var pos: NSPoint!

override public func mouseDown(event: NSEvent) {
    pos = convertPointToLayer(convertPoint(event.locationInWindow, fromView: nil))
    let scale = layer!.contentsScale
    pos.x *= scale
    pos.y *= scale
}
{% endhighlight %}

As you notice, we are scaling down the coordinates, from the entire screen to only our `MetalView` size, and then we update the coordinates with this scale factor we got from the view's layer. Next, inside the `registerShaders()` function let's initialize the mouse buffer:

{% highlight swift %}mouseBuffer = device!.newBufferWithLength(sizeof(NSPoint), options: [])
{% endhighlight %}

Now go back to the `update()` function and add these lines at the end of it so we can send the current mouse coordinates to the buffer:

{% highlight swift %}bufferPointer = mouseBuffer.contents()
memcpy(bufferPointer, &pos, sizeof(NSPoint))
{% endhighlight %}

Next, in `drawRect()` we set the mouse buffer at index __2__: 

{% highlight swift %}commandEncoder.setBuffer(mouseBuffer, offset: 0, atIndex: 2)
{% endhighlight %}

Then, in `Shaders.metal` we again update the kernel signature to also include the mouse buffer:

{% highlight swift %}kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    constant float &timer [[buffer(1)]],
                    constant float2 &mouse [[buffer(2)]],
                    uint2 gid [[thread_position_in_grid]])
{% endhighlight %}

Finally, we update the line below:

{% highlight swift %}float3 color = float3( dmin.w );
{% endhighlight %}

with this line:

{% highlight swift %}float3 color = float3(mouse.x - mouse.y);
{% endhighlight %}

What we are trying to do here is change the way color is calculated, by passing the mouse coordinates to the `color` variable. Run the playground and click in various view areas, then notice the effect. The output image should look like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter12_2.gif "2")

You can play with the kernel code to achieve prettier effects by using the mouse coordinates in other parts of the code. There is one other matter I need to take a closer look at, the fact that `NSPoint` might not correctly map to the kernel `float2` type we used. The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!