---
published: true
title: The Model I/O framework
layout: post
---
__Model I/O__ was introduced in `2015` for `iOS 9` and `OS X 10.11` and it is a framework that helps us create more realistic and interactive graphics. We can use it to import/export `3D` assets, to describe lighting, materials and environments, to bake lights, to subdivide and voxelize meshes, and for physical based rendering. Model I/O easily integrates our assets with our code in various `3D APIs`:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_1.png "1")

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

Integrating `Model I/O` with `Metal` takes four steps:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_2.png "2")

###Step 1: set up the render pipeline state

First we create a vertex descriptor so we can pass input to the vertex function/shader. The vertex descriptor is needed to describe the vertex attribute inputs to a render state pipeline. We need `3 x 4` bytes for the vertex position, `4 x 1` byte for color, `2 x 2` bytes for the texture coordinates and `4 x 1` byte for ambient occlusion. At the end we tell the descriptor how large (__24__) our `stride` is in total:

{% highlight swift %}let vertexDescriptor = MTLVertexDescriptor()
vertexDescriptor.attributes[0].offset = 0
vertexDescriptor.attributes[0].format = MTLVertexFormat.float3 // position
vertexDescriptor.attributes[1].offset = 12
vertexDescriptor.attributes[1].format = MTLVertexFormat.uChar4 // color
vertexDescriptor.attributes[2].offset = 16
vertexDescriptor.attributes[2].format = MTLVertexFormat.half2 // texture
vertexDescriptor.attributes[3].offset = 20
vertexDescriptor.attributes[3].format = MTLVertexFormat.float // occlusion
vertexDescriptor.layouts[0].stride = 24
let renderPipelineDescriptor = MTLRenderPipelineDescriptor()
renderPipelineDescriptor.vertexDescriptor = vertexDescriptor
let rps = device.newRenderPipelineStateWithDescriptor(renderPipelineDescriptor)
{% endhighlight %}

###Step 2: set up the asset initialization

We need to also create a `Model I/O` vertex descriptor to describe the layout of the vertex attributes in a mesh. We are using a model named __Farmhouse.obj__ that also has a texture __Farmhouse.png__ (both already added to the sample project for you):

{% highlight swift %}let desc = MTKModelIOVertexDescriptorFromMetal(vertexDescriptor)
var attribute = desc.attributes[0] as! MDLVertexAttribute
attribute.name = MDLVertexAttributePosition
attribute = desc.attributes[1] as! MDLVertexAttribute
attribute.name = MDLVertexAttributeColor
attribute = desc.attributes[2] as! MDLVertexAttribute
attribute.name = MDLVertexAttributeTextureCoordinate
attribute = desc.attributes[3] as! MDLVertexAttribute
attribute.name = MDLVertexAttributeOcclusionValue
let mtkBufferAllocator = MTKMeshBufferAllocator(device: device!)
let url = Bundle.main.url(forResource: "Farmhouse", withExtension: "obj")
let asset = MDLAsset(url: url!, vertexDescriptor: desc, bufferAllocator: mtkBufferAllocator)
{% endhighlight %}

Next, we load the texture for our asset:

{% highlight swift %}let loader = MTKTextureLoader(device: device)
let file = Bundle.main.path(forResource: "Farmhouse", ofType: "png")
let data = try Data(contentsOf: URL(fileURLWithPath: file))
let texture = try loader.newTexture(with: data, options: nil)
{% endhighlight %}

###Step 3: set up `MetalKit` mesh and submesh objects

We are now creating the meshes and submeshes needed in the final, fourth step. We also compute the __Ambient Occlusion__, which is a measure of geometry obstruction, and it tells us how much of the ambient light actually reaches any given pixel or point of our object, and and how much of this light is blocked by surrounding meshes. `Model I/O` provides a `UV` mapper that creates a `2D` texture and wraps it around the object's `3D` mesh. For every pixel in the texture we can compute the ambient occlusion value, which is a just one extra float added for each vertex:

{% highlight swift %}let mesh = asset.object(at: 0) as? MDLMesh
mesh.generateAmbientOcclusionVertexColors(withQuality: 1, attenuationFactor: 0.98, objectsToConsider: [mesh], vertexAttributeNamed: MDLVertexAttributeOcclusionValue)
let meshes = try MTKMesh.newMeshes(from: asset, device: device!, sourceMeshes: nil)
{% endhighlight %}

###Step 4: set up `Metal` rendering and drawing of meshes

Finally, we configure the command encoder with the mesh data it needs to draw:

{% highlight swift %}let mesh = (meshes?.first)!
let vertexBuffer = mesh.vertexBuffers[0]
commandEncoder.setVertexBuffer(vertexBuffer.buffer, offset: vertexBuffer.offset, at: 0)
let submesh = mesh.submeshes.first!
commandEncoder.drawIndexedPrimitives(submesh.primitiveType, indexCount: submesh.indexCount, indexType: submesh.indexType, indexBuffer: submesh.indexBuffer.buffer, indexBufferOffset: submesh.indexBuffer.offset)
{% endhighlight %}

Next, we will work on our shader functions. First we set up our structs for the vertices and uniforms:

{% highlight swift %}struct VertexIn {
    float4 position [[attribute(0)]];
    float4 color [[attribute(1)]];
    float2 texCoords [[attribute(2)]];
    float occlusion [[attribute(3)]];
};

struct VertexOut {
    float4 position [[position]];
    float4 color;
    float2 texCoords;
    float occlusion;
};

struct Uniforms {
    float4x4 modelViewProjectionMatrix;
};
{% endhighlight %}

Notice that we are matching the information we set up in the vertex descriptor, with the `VertexIn` struct.
For the vertex function, we use a __[[ stage_in ]]__ attribute because we are passing per-vertex inputs as an argument to this function:

{% highlight swift %}vertex VertexOut vertex_func(const VertexIn vertices [[stage_in]],
                             constant Uniforms &uniforms [[buffer(1)]],
                             uint vertexId [[vertex_id]])
{
    float4x4 mvpMatrix = uniforms.modelViewProjectionMatrix;
    float4 position = vertices.position;
    VertexOut out;
    out.position = mvpMatrix * position;
    out.color = float4(1);
    out.texCoords = vertices.texCoords;
    out.occlusion = vertices.occlusion;
    return out;
}
{% endhighlight %}

The fragment function reads the per-fragment inputs passed from the vertex function and also processes the texture we passed via the command encoder: 

{% highlight swift %}fragment half4 fragment_func(VertexOut fragments [[stage_in]],
                             texture2d<float> textures [[texture(0)]])
{
    float4 baseColor = fragments.color;
    return half4(baseColor);
}
{% endhighlight %}

If you run the playground, you will see this output image:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_3.png "3")

That's a pretty dull white model. Let's apply the ambient occlusion to it by replacing the last line in the fragment function with these lines:

{% highlight swift %}float4 occlusion = fragments.occlusion;
return half4(baseColor * occlusion);
{% endhighlight %}

If you run the playground again, you will see this output image:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_4.png "4")

The ambient occlusion also seems a bit raw and that is because our model is quite flat, without any curves or surface irregularities which would give way more credit to the realism that the ambient occlusion brings. Next, let's apply the texture. Replace the last line in the fragment function with these lines: 

{% highlight swift %}constexpr sampler samplers;
float4 texture = textures.sample(samplers, fragments.texCoords);
return half4(baseColor * texture);
{% endhighlight %}    

If you run the playground again, you will see this output image:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_5.png "5")

The texture looks really great on this model, but it would looks even more realistic if we brought the ambient occlusion back. Replace the last line in the fragment function with these lines: 

{% highlight swift %}return half4(baseColor * occlusion * texture);
{% endhighlight %}

If you run the playground again, you will see this output image:

![alt text](https://github.com/MetalKit/images/raw/master/modelio_6.png "6")

Not bad for a few lines of code, right? `Model I/O` is such a great framework for `3D` graphics and game programmers. There are a couple of articles on the web about using `Model I/O` with `SceneKit`, however, I thought using it with `Metal` is even more interesting! The [source code](https://github.com/MetalKit/modelio) is posted on Github as usual.

Until next time!