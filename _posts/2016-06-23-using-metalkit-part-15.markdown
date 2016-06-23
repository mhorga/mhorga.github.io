---
published: false
title: Using MetalKit part 15
layout: post
---
At the end of `part 13` we said we can make our planet look more realistic in two ways: either apply a texture to it, or add some noise to the `planet` color. We showed how to add noise in `part 14`. This week we will learn about __textures__ and __samplers__.  Let's pick up where we left off in [Part 13](http://mhorga.org/2016/05/25/using-metalkit-part-13.html) since we do not need the noise code this time. 

First, let's clean our kernel to only include these lines:

{% highlight swift %}kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    constant float &timer [[buffer(1)]],
                    uint2 gid [[thread_position_in_grid]])
{
    int width = output.get_width();
    int height = output.get_height();
    float2 uv = float2(gid) / float2(width, height);
    uv = uv * 2.0 - 1.0;
    output.write(float4(0), gid);
}
{% endhighlight %}

Next, let's work in `MetalView.swift`. First create a new texture object:

var texture: MTLTexture!

Next, replace this line:

commandEncoder.setBuffer(mouseBuffer, offset: 0, atIndex: 2)

with this line:

commandEncoder.setTexture(texture, atIndex: 1)

Then, let's create a function that sets up or texture:

    func setUpTexture() {
        let path = NSBundle.mainBundle().pathForResource("texture", ofType: "jpg")
        let textureLoader = MTKTextureLoader(device: device!)
        texture = try! textureLoader.newTextureWithContentsOfURL(NSURL(fileURLWithPath: path!), options: nil)   
    }

Next, call this function in our `init` function:

    override public init(frame frameRect: CGRect, device: MTLDevice?) {
        super.init(frame: frameRect, device: device)
        registerShaders()
        setUpTexture()
    }

Finally, I added an image named __texture.jpg__ in the `Resources` folder of our playground, but you can add yours instead if you want.

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter13_6.gif "6")

The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!
 
{% highlight swift %}
{% endhighlight %}