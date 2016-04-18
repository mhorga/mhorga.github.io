---
published: false
title: Using MetalKit part 9
layout: post
---
I bet many of you missed the `MetalKit` series, so today we are returning back to it, and we will learn how to draw 3D content in `Metal`. Let's continue working on our playground and pick up where we left off in [part 8](https://github.com/Swiftor/Metal/tree/master/ch08/chapter08.playground) of the series. 

We will render a 3D cube by the end of this episode but first let's draw a 2D square and then we can re-use the square logic for all the other faces of the cube. Let's modify the `vertex_data` array so that it holds __4__ vertices instead of __3__ we needed for a triangle:

{% highlight swift %}
let vertex_data = [
    Vertex(pos: [-1.0, -1.0, 0.0,  1.0], col: [1, 0, 0, 1]),
    Vertex(pos: [ 1.0, -1.0, 0.0,  1.0], col: [0, 1, 0, 1]),
    Vertex(pos: [ 1.0,  1.0, 0.0,  1.0], col: [0, 0, 1, 1]),
    Vertex(pos: [-1.0,  1.0, 0.0,  1.0], col: [1, 1, 1, 1])
]
{% endhighlight %}

Here comes the interesting part. Since squares and any other complex geometry is made from triangles, and since most vertices belong to __2__ or more triangles, there is no need to create copies of these vertices because we have a way of reusing them via an `index buffer` that keeps track of the order in which the vertices will be used by storing each vertex index from the `vertex buffer`. So let's create such a list of indexes:

{% highlight swift %}
let index_data: [UInt16] = [
    0, 1, 2, 2, 3, 0
]
{% endhighlight %}

To understand how these indexes are stored, let's look at this image below:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter09_1.png "1")

As you can see, for the front face (square) we use vertices stored at positions __0__ through __3__ in the `vertex_buffer`. Later on we will add the other __4__ vertices as well. The front face is made of two triangles. We first draw the triangle that uses vertices __0__, __1__ and __2__ and then we draw the triangle that uses vertices __2__, __3__ and __0__. Notice that two of the vertices are re-used, as expected. Also notice that the drawing is done __counterclockwise__. This is the default rendering rule in `Metal` but it can be changed to `clockwise` as well.

Then, we need to create the __index_buffer__:

{% highlight swift %}
var index_buffer: MTLBuffer!
{% endhighlight %}

Next, we need to assign the `index_data` to the `index buffer` inside the `createBuffers()` function:

{% highlight swift %}
index_buffer = device!.newBufferWithBytes(index_data, length: sizeof(UInt16) * index_data.count , options: [])
{% endhighlight %}

Last, inside the `drawRect(:)` function we need to replace the `drawPrimitives` call:

{% highlight swift %}
command_encoder.drawPrimitives(.Triangle, vertexStart: 0, vertexCount: 3, instanceCount: 1)
{% endhighlight %}

with a __drawIndexedPrimitives__ call:

{% highlight swift %}
command_encoder.drawIndexedPrimitives(.Triangle, indexCount: index_buffer.length / sizeof(UInt16), indexType: MTLIndexType.UInt16, indexBuffer: index_buffer, indexBufferOffset: 0)
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter09_2.png "2")

Now that we know how to draw a square, let's see how to draw more squares!

The [source code](https://github.com/Swiftor/Raytracing5) is posted on Github as usual.

{% highlight swift %}
{% endhighlight %}

Until next time!