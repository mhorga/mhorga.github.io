---
published: false
title: The Model I/O framework
layout: post
---
__Model I/O__ was introduced in `2015` for `iOS 9` and `OS X 10.11` and it is a framework that helps us create more realistic and interactive graphics. We can use it to import/export `3D` assets, to describe lighting, materials and environments, to bake lights, to subdivide and voxelize meshes, and for physical based rendering. Model I/O easily integrates our assets with our code in various `3D APIs`:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_1.png "Model I/O integration")

In order to import an asset we simply do:

{% highlight swift %}var url = URL(string: "/Users/YourUsername/Desktop/imported.obj")
let asset = MDLAsset(url: url!)
{% endhighlight %}

To export an asset we do:

{% highlight swift %}url = URL(string: "/Users/YourUsername/Desktop/exported.obj")
try! asset.export(to: url!)
{% endhighlight %}

Model I/O will save both the __.obj__ file and an additional __.mtl__ file that contains information about the object materials, such as in this example:

{% highlight text %}# Apple ModelI/O MTL File: exported.mtl

newmtl material_1
	Kd 0.8 0.8 0.8
	Ka 0 0 0
	Ks 0 0 0
	ao 0 0 0
	subsurface 0 0 0
	metallic 0 0 0
	specularTint 0 0 0
	roughness 0.9 0 0
	anisotropicRotation 0 0 0
	sheen 0.05 0 0
	sheenTint 0 0 0
	clearCoat 0 0 0
	clearCoatGloss 0 0 0
{% endhighlight %}

###Step 1: set up the render pipeline state

###Step 2: set up the asset initialization

__Ambient Occlusion__ is a measure of geometry obstruction, and it tells us how much of the ambient light actually reaches any given pixel or point of our object, and and how much of this light is blocked by surrounding meshes. `Model I/O` provides a `UV` mapper that creates a `2D` texture and wraps it around the object's `3D` mesh. For every pixel in the texture we can compute the ambient occlusion value, which is a just one extra float added for each vertex:

{% highlight swift %}mesh.generateAmbientOcclusionVertexColors(withQuality: 1, attenuationFactor: 0.98, objectsToConsider: [mesh], vertexAttributeNamed: MDLVertexAttributeOcclusionValue)
{% endhighlight %}

###Step 3: set up `MetalKit` mesh and submesh objects

###Step 4: set up `Metal` rendering and drawing of meshes

![alt text](https://github.com/MetalKit/images/raw/master/modelio_6.png "6")

{% highlight swift %}
{% endhighlight %}

Until next time!