---
published: false
title: Using MetalKit part 5
layout: post
---
Last time we described the `graphics pipeline` as well as the `Metal pipeline`. It is time we looked deeper inside the pipeline, and understand how vertices are really processed at a lower level. For this, we need to learn a few `3D math` concepts such as __transformations__. 

In the world of `3D graphics` we often think in terms of __3__ or __4__ dimensions for our data. As you remember from our previous episodes, `location` and `color` were both of type __vector_float4__ (4-dimensional). In order to draw 3D geometry on the screen, vertices suffer a series of transformations - from `object space` to `world space`, then to `camera/eye space`, then to `clipping space`, then to `normalized device coordinates` space, and finally to `screen space`. We are only looking at the first stage in this episode.

The vertices of our `triangle` are expressed in terms of an `object space` (local coordinates). They are currently specified about the triangle's origin which lies at the center of the screen. In order to position and move the triangle in a larger scene (world space), we need to apply `transformations` to these vertices. The `transformations` we will look at are: __scaling__, __translation__ and __rotation__.

{% highlight swift %} 
|  1 0 0 Dx  |
|  0 1 0 Dy  |
|  0 0 1 Dz  |
|  0 0 0 1   |
{% endhighlight %}

{% highlight swift %} 
|  Sx 0  0  0  |
|  0  Sy 0  0  |
|  0  0  Sz 0  |
|  0  0  0  1  |
{% endhighlight %}

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

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter05_1.png "1")

{% highlight swift %} 
{% endhighlight %}

The [source code](https://github.com/Swiftor/Metal/tree/master/ch05) is posted on Github as usual.

Until next time!
