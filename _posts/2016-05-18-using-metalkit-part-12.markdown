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

{% highlight swift %float2 cc = 1.1*float2( 0.5*cos(0.1) - 0.25*cos(0.2), 0.5*sin(0.1) - 0.25*sin(0.2) );
{% endhighlight %}

with this line:

{% highlight swift %}}float2 cc = 1.1*float2( 0.5*cos(0.1*timer) - 0.25*cos(0.2*timer), 0.5*sin(0.1*timer) - 0.25*sin(0.2*timer) );
{% endhighlight %}

Next,

{% highlight swift %}
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter12_1.png "1")

The [source code](https://github.com/Swiftor/Metal) is posted on Github as usual.

Until next time!