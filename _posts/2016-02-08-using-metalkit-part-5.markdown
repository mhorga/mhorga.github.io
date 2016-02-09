---
published: true
title: Using MetalKit part 5
layout: post
---
Last time we described the `graphics pipeline` and the `Metal pipeline`. It is time we looked deeper inside the pipeline, and understand how vertices are really processed at a lower level. For this, we need to learn a few `3D math` concepts such as __transformations__. 

In the world of `3D graphics` we often think in terms of __3__ or __4__ dimensions for our data. As you remember from our previous episodes, `location` and `color` were both of type __vector_float4__ (4-dimensional). In order to draw 3D geometry on the screen, vertices suffer a series of transformations - from `object space` to `world space`, then to `camera/eye space`, then to `clipping space`, then to `normalized device coordinates` space, and finally to `screen space`. We are only looking at the first stage in this episode.

The vertices of our `triangle` are expressed in terms of an `object space` (local coordinates). They are currently specified about the triangle's origin which lies at the center of the screen. In order to position and move the triangle in a larger scene (world space), we need to apply `transformations` to these vertices. The `transformations` we will look at are: __scaling__, __translation__ and __rotation__.

The __translation matrix__ is an __identity matrix__ (with values of __1__ on its main diagonal) where positions __[12]__, __[13]__ and __[14]__ (in `column-major order` they are the equivalent of the `[3]`, `[7]` and `[11]` positions) are populated with the values of a __D__ vector representing the __distance__ the vertex would be moved to, on the respective __x__, __y__, __z__ axes.

{% highlight swift %} 
| 1     0     0    Dx |
| 0     1     0    Dy |
| 0     0     1    Dz |
| 0     0     0     1 |
{% endhighlight %}

The __scaling matrix__ is an __identity matrix__ where positions __[0]__, __[5]__ and __[10]__ are populated with the values of a __S__ vector representing the __scale__ the vertex would be zoomed in/out to. The __x__, __y__, __z__ vector values are usually the same __float__ value since scaling is done proportionally on all axes.

{% highlight swift %} 
| Sx    0     0     0 |
| 0     Sy    0     0 |
| 0     0     Sz    0 |
| 0     0     0     1 |
{% endhighlight %}

The __rotation matrix__ is an __identity matrix__ where depending on which axis we are rotating about, different positions are being populated with either the __sinus__ or __cosinus__ of the __angle__ we are rotating with. If we are rotating about the __x__ axis, positions __[5]__, __[6]__, __[9]__ and __[10]__ are populated. If we are rotating about the __y__ axis, positions __[0]__, __[2]__, __[8]__ and __[10]__ are populated. Finally, if we are rotating about the __z__ axis, positions __[0]__, __[1]__, __[4]__ and __[5]__ are populated. Remember, these positions need to be transposed into `column-major order`.

{% highlight swift %} 
| 1     0     0     0 |
| 0    cos  -sin    0 |
| 0    sin   cos    0 |
| 0     0     0     1 |

| cos   0    sin    0 |
| 0     1     0     0 |
| -sin  0    cos    0 |
| 0     0     0     1 |

| cos  -sin   0     0 |
| sin  cos    0     0 |
| 0     0     1     0 |
| 0     0     0     1 |
{% endhighlight %}

Alright, we had enough math for a whole week, so let's put these matrices into code. We will continue with the code from where we left off after [part 3](https://github.com/Swiftor/Metal/tree/master/ch04). It comes in handy for us to create a `struct` named __Matrix__ that will include these `transformations`:

{% highlight swift %} 
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
        return matrix
    }
}
{% endhighlight %}

Let's walk over this code. We first create the `struct` and declare an `array` of `floats`. Then we provide an initializer for it, which is the __identity matrix__ (all __1's__ on the diagonal). Next, we create the transformation matrices. Finally, we create a __modelMatrix__ which will combine all the transformations into a single output matrix. 

In order to have these transformation work, we need to send them to the `GPU` via a `shader`. In order to do that we first need to create a new buffer. Let's name it __uniform_buffer__. `Uniforms` are constructs we can use when we want to send data the applies to the entire model rather than to each vertex. It only makes sense that we save space by using `uniforms` instead and sending one final `model matrix` containing all the transformations. So at the very beginning of our `MetalView` class, create the new buffer:

{% highlight swift %} 
var uniform_buffer: MTLBuffer!
{% endhighlight %}

Inside the __createBuffers()__ function, allocate memory for the buffer, enough to hold a __4x4__ matrix:

{% highlight swift %} 
uniform_buffer = device!.newBufferWithLength(sizeof(Float) * 16, options: [])
let bufferPointer = uniform_buffer.contents()
memcpy(bufferPointer, Matrix().modelMatrix(Matrix()).m, sizeof(Float) * 16)
{% endhighlight %}

Inside the __sendToGPU()__ function, after setting the `vertex_buffer` in the `command encoder`, also set the __uniform_buffer__:

{% highlight swift %} 
command_encoder.setVertexBuffer(uniform_buffer, offset: 0, atIndex: 1)
{% endhighlight %}

Finally, let's move to __Shaders.metal__ for the last part of the configuration. Below the `Vertex` struct, create a new struct named __Uniforms__ that will hold our model matrix:

{% highlight swift %} 
struct Uniforms {
    float4x4 modelMatrix;
};
{% endhighlight %}

Modify the `vertex shader` to include the transformations we passed along from the `CPU`:

{% highlight swift %} 
vertex Vertex vertex_func(constant Vertex *vertices [[buffer(0)]],
                          constant Uniforms &uniforms [[buffer(1)]],
                          uint vid [[vertex_id]])
{
    float4x4 matrix = uniforms.modelMatrix;
    Vertex in = vertices[vid];
    Vertex out;
    out.position = matrix * float4(in.position);
    out.color = in.color;
    return out;
}
{% endhighlight %}

All we did here was to pass __uniforms__ as the __2nd__ argument (buffer), and then multiply the model matrix with the vertices. If you run the app now, you will see our good old triangle friend, taking the entire space of the view.

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter05_1.png "1")

Let's __scale__ it down to a quarter of its original size. Add this line to the __modelMatrix__ function:

{% highlight swift %} 
matrix = scalingMatrix(matrix, 0.25)
{% endhighlight %}

Run the app again and notice that the triangle is way smaller now:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter05_2.png "2")

Next, let's __translate__ the triangle up on the __y__ axis, and move it half the screen size:

{% highlight swift %} 
matrix = translationMatrix(matrix, float3(0.0, 0.5, 0.0))
{% endhighlight %}

Run the app again and notice that the triangle is now higher than before:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter05_3.png "3")

Finally, let's __rotate__ the triangle about the __z__ axis:

{% highlight swift %} 
matrix = rotationMatrix(matrix, float3(0.0, 0.0, 0.1))
{% endhighlight %}

Run the app again and notice that the triangle is now also rotated:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter05_4.png "4")

Next week we will finally get to drawing 3D objects (such as cubes or spheres) with `z-depth`. The [source code](https://github.com/Swiftor/Metal/tree/master/ch05) is posted on Github as usual.

Until next time!