---
published: true
title: Using MetalKit part 9
layout: post
---
I bet many of you missed the `MetalKit` series, so today we are returning back to it, and we will learn how to draw 3D content in `Metal`. Let's continue working on our playground and pick up where we left off in [part 8](https://github.com/MetalKit/metal) of the series. 

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

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_1.jpg?raw=true "1")

So for the front face (square) we use vertices stored at positions __0__ through __3__ in the `vertex_buffer`. Later on we will add the other __4__ vertices as well. The front face is made of two triangles. We first draw the triangle that uses vertices __0__, __1__ and __2__ and then we draw the triangle that uses vertices __2__, __3__ and __0__. Notice that two of the vertices are re-used, as expected. Also notice that the drawing is done __counterclockwise__. This is the default rendering rule in `Metal` but it can be changed to `clockwise` as well.

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

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_2.png?raw=true "2")

Now that we know how to draw a square, let's see how to draw more squares!

{% highlight swift %}
let vertex_data = [
    Vertex(pos: [-1.0, -1.0,  1.0, 1.0], col: [1, 0, 0, 1]),
    Vertex(pos: [ 1.0, -1.0,  1.0, 1.0], col: [0, 1, 0, 1]),
    Vertex(pos: [ 1.0,  1.0,  1.0, 1.0], col: [0, 0, 1, 1]),
    Vertex(pos: [-1.0,  1.0,  1.0, 1.0], col: [1, 1, 1, 1]),
    Vertex(pos: [-1.0, -1.0, -1.0, 1.0], col: [0, 0, 1, 1]),
    Vertex(pos: [ 1.0, -1.0, -1.0, 1.0], col: [1, 1, 1, 1]),
    Vertex(pos: [ 1.0,  1.0, -1.0, 1.0], col: [1, 0, 0, 1]),
    Vertex(pos: [-1.0,  1.0, -1.0, 1.0], col: [0, 1, 0, 1])
]
let index_data: [UInt16] = [
    0, 1, 2, 2, 3, 0,   // front
    1, 5, 6, 6, 2, 1,   // right
    3, 2, 6, 6, 7, 3,   // top
    4, 5, 1, 1, 0, 4,   // bottom
    4, 0, 3, 3, 7, 4,   // left
    7, 6, 5, 5, 4, 7,   // back
]
{% endhighlight %}

Now that we have the entire cube geometry ready for rendering, let's go to `MathUtils.swift` and in `modelMatrix()` comment out the `rotation` and the `translation` calls, and only leave the scaling on for a factor of __0.5__. You will most likely see an image like this:

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_3.png?raw=true "3")

Hmm, but it's still a square! Yes, it is, because we still don't have the notion of `depth` and the cube looks just flat. It's time to tweak some math logic now. We don't need to use the `Matrix` struct anymore because the __simd__ framework offers us similar data structures and math functions we can readily use. We can easily rewrite our transform functions to work with __matrix_float4x4__ instead of the custom `Matrix` struct we used. 

But how do 3D objects end up on our 2D screens, you might ask. This process takes each pixel through a series of transformations. First the __modelMatrix()__ transforms the pixel from `object space` to `world space`. This matrix is the one we already know, the one responsible for translations, rotations and scaling. With the newly rewritten functions above, the `modelMatrix` could look like this:

{% highlight swift %}
func modelMatrix() -> matrix_float4x4 {
    let scaled = scalingMatrix(0.5)
    let rotatedY = rotationMatrix(Float(M_PI)/4, float3(0, 1, 0))
    let rotatedX = rotationMatrix(Float(M_PI)/4, float3(1, 0, 0))
    return matrix_multiply(matrix_multiply(rotatedX, rotatedY), scaled)
}
{% endhighlight %}

You notice the useful `matrix_multiply` function which we could not use before for the `Matrix` struct. Also, since all these pixels will undergo the same transformation, we want to store the matrix as a __Uniform__ and pass it to the `vertex shader`. For this. let's create a new struct:

{% highlight swift %}
struct Uniforms {
    var modelViewProjectionMatrix: matrix_float4x4
}
{% endhighlight %}

Back in the `createBuffers()` function, let's pass the Uniforms to the shader via the buffer pointer we already used to pass the `modelMatrix`:

{% highlight swift %}
let modelViewProjectionMatrix = modelMatrix()
var uniforms = Uniforms(modelViewProjectionMatrix: modelViewProjectionMatrix)
memcpy(bufferPointer, &uniforms, sizeof(Uniforms))
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_4.png?raw=true "4")

Hmm... the cube almost looks right, but something is still missing. The next transformation the pixels need to go through is from `world space` to `camera space`. Everything we see on the screen is `viewed` by a virtual camera through a __frustum__ (pyramidal shape) that has a __near__ and __far__ planes to limit the `view (camera) space:

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_5.png?raw=true "5")

Back in `MathUtils.swift` let's create the __viewMatrix()__ as well: 

{% highlight swift %}
func viewMatrix() -> matrix_float4x4 {
    let cameraPosition = vector_float3(0, 0, -3)
    return translationMatrix(cameraPosition)
}
{% endhighlight %}

The next transformation the pixels need to go through is from `camera space` to `clip space`. Here, all the vertices that are not inside the `clip space` will determine whether the triangle will be `culled` (all vertices outside the clip space) or `clipped to bounds` (some vertices are outside but not all). The __projectionMatrix()__ will help us compute the bounds and determine where the vertices are:

{% highlight swift %}
func projectionMatrix(near: Float, far: Float, aspect: Float, fovy: Float) -> matrix_float4x4 {
    let scaleY = 1 / tan(fovy * 0.5)
    let scaleX = scaleY / aspect
    let scaleZ = -(far + near) / (far - near)
    let scaleW = -2 * far * near / (far - near)
    let X = vector_float4(scaleX, 0, 0, 0)
    let Y = vector_float4(0, scaleY, 0, 0)
    let Z = vector_float4(0, 0, scaleZ, -1)
    let W = vector_float4(0, 0, scaleW, 0)
    return matrix_float4x4(columns:(X, Y, Z, W))
}
{% endhighlight %}

The last two transformations are from `clip space` to `normalized device coordinates (NDC)` and from `NDC` to `screen space`. These two transformations are handled by the Metal framework for us. 

Next, back in the `createBuffers()` function, let's modify the `modelViewProjectionMatrix` we set before to just the `modelMatrix`:

{% highlight swift %}
let aspect = Float(drawableSize.width / drawableSize.height)
let projMatrix = projectionMatrix(1, far: 100, aspect: aspect, fovy: 1.1)
let modelViewProjectionMatrix = matrix_multiply(projMatrix, matrix_multiply(viewMatrix(), modelMatrix()))
{% endhighlight %}

In `drawRect(:)` we need to set rules for the culling mode and for front facing, in order to avoid weird artifacts such as cube transparency:

{% highlight swift %}
command_encoder.setFrontFacingWinding(.CounterClockwise)
command_encoder.setCullMode(.Back)
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_6.png?raw=true "6")

This is finally the 3D cube we were all waiting to see! There is one more thing we can do to make it even more realistic and lively looking: give it a spin. First, let's create a global variable named __rotation__ which we want to update as time goes by:

{% highlight swift %}
var rotation: Float = 0
{% endhighlight %}

Next, grab all the matrices from inside the `createBuffers()` function and let's create a new one named __update()__. Here is where we update `rotation` every frame to create a smooth rotation effect:

{% highlight swift %}
func update() {
    let scaled = scalingMatrix(0.5)
    rotation += 1 / 100 * Float(M_PI) / 4
    let rotatedY = rotationMatrix(rotation, float3(0, 1, 0))
    let rotatedX = rotationMatrix(Float(M_PI) / 4, float3(1, 0, 0))
    let modelMatrix = matrix_multiply(matrix_multiply(rotatedX, rotatedY), scaled)
    let cameraPosition = vector_float3(0, 0, -3)
    let viewMatrix = translationMatrix(cameraPosition)
    let aspect = Float(drawableSize.width / drawableSize.height)
    let projMatrix = projectionMatrix(0, far: 10, aspect: aspect, fovy: 1)
    let modelViewProjectionMatrix = matrix_multiply(projMatrix, matrix_multiply(viewMatrix, modelMatrix))
    let bufferPointer = uniform_buffer.contents()
    var uniforms = Uniforms(modelViewProjectionMatrix: modelViewProjectionMatrix)
    memcpy(bufferPointer, &uniforms, sizeof(Uniforms))
}
{% endhighlight %}    

In `drawRect(:)` call the `update` function:

{% highlight swift %}
update()
{% endhighlight %}

In the main playground page, you should see a similar image:

![alt text](https://github.com/MetalKit/images/blob/master/chapter09_7.gif?raw=true "7")

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!