---
published: true
title: Using MetalKit part 15
layout: post
---
At the end of `part 13` we concluded we can make our planet look more realistic in two ways: either apply a texture to it, or add some noise to the `planet` color. We showed how to add noise in `part 14`. This week we look at __textures__ and __samplers__. Textures are useful because they can provide a greater level of detail to surfaces than color computing for each vertex could.

Let's pick up where we left off in [Part 13](http://mhorga.org/2016/05/25/using-metalkit-part-13.html) since we do not need the noise code this time. First, in `MetalView.swift` let's remove the `mouseDown` function as we are not going to need it anymore. Also remove the `mouseBuffer` and `pos` variables, as well as any references to them in the code. Then, create a new texture object:

{% highlight swift %}var texture: MTLTexture!
{% endhighlight %}

Next, replace this line (it's likely you removed it already in the above cleaning step):

{% highlight swift %}commandEncoder.setBuffer(mouseBuffer, offset: 0, atIndex: 2)
{% endhighlight %}

with this line:

{% highlight swift %}commandEncoder.setTexture(texture, atIndex: 1)
{% endhighlight %}

and also change the buffer index for our `timer` from __1__ to __0__:

{% highlight swift %}commandEncoder.setBuffer(timerBuffer, offset: 0, atIndex: 0)
{% endhighlight %}

I added an image named __texture.jpg__ in the `Resources` folder of our playground, but you can add yours instead if you want. Let's create a function that sets up or texture using this image:

{% highlight swift %}func setUpTexture() {
    let path = NSBundle.mainBundle().pathForResource("texture", ofType: "jpg")
    let textureLoader = MTKTextureLoader(device: device!)
    texture = try! textureLoader.newTextureWithContentsOfURL(NSURL(fileURLWithPath: path!), options: nil)
}
{% endhighlight %}

Next, call this function in our `init` function:

{% highlight swift %}override public init(frame frameRect: CGRect, device: MTLDevice?) {
    super.init(frame: frameRect, device: device)
    registerShaders()
    setUpTexture()
}
{% endhighlight %}

Now, let's clean our kernel in __Shaders.metal__ to only include these lines:

{% highlight swift %}kernel void compute(texture2d<float, access::write> output [[texture(0)]],
                    texture2d<float, access::read> input [[texture(1)]],
                    constant float &timer [[buffer(1)]],
                    uint2 gid [[thread_position_in_grid]])
{
    float4 color = input.read(gid);
    gid.y = input.get_height() - gid.y;
    output.write(color, gid);
}
{% endhighlight %}

You will first notice that we get the `input` texture through the __[[texture(1)]]__ attribute since that is the index we set it to in the command encoder. Also, the access we requested for it is __read__. Then we read it into the `color` variable, however, it comes in flipped upside-down. In order to fix this, on the next line we just flip the __Y__ coordinate for each pixel. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter15_1.png "1")

If you open the image and compare it with the output, you will notice it is now correctly oriented. Next, we want to bring back our planet and the dark sky around it. Replace the `output` line with this block of code:

{% highlight swift %}int width = input.get_width();
int height = input.get_height();
float2 uv = float2(gid) / float2(width, height);
uv = uv * 2.0 - 1.0;
float radius = 0.5;
float distance = length(uv) - radius;
output.write(distance < 0 ? color : float4(0), gid);
{% endhighlight %}

This code looks familiar since we already discussed in the previous chapters how to create the planet and the black space surrounding it. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter15_2.png "2")

So far so good! We next want to make our planet rotate again. Replace the `output` line with this block of code:

{% highlight swift %}uv = fmod(float2(gid) + float2(timer * 100, 0), float2(width, height));
color = input.read(uint2(uv));
output.write(distance < 0 ? color : float4(0), gid);
{% endhighlight %}

This code again looks familiar from the previous chapters where we discussed how to use `timer` to animate the planet. The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter15_3.gif "3")

This is a bit awkward! The output looks like someone would walk in a dark cave, next to the wall and carrying a torch. Replace the last three lines we added with this block of code:

{% highlight swift %}uv = uv * 2;
radius = 1;
constexpr sampler textureSampler(coord::normalized,
                                 address::repeat,
                                 min_filter::linear,
                                 mag_filter::linear,
                                 mip_filter::linear );
float3 norm = float3(uv, sqrt(1.0 - dot(uv, uv)));
float pi = 3.14;
float s = atan2( norm.z, norm.x ) / (2 * pi);
float t = asin( norm.y ) / (2 * pi);
t += 0.5;
color = input.sample(textureSampler, float2(s + timer * 0.1, t));
output.write(distance < 0 ? color : float4(0), gid);
{% endhighlight %}

First, we  scale down to half the size of the texture and set the radius to __1__ so we can match the planet object size with the texture size. Then comes the magic. Let me introduce the __sampler__. A `sampler` is an object that contains various rendering states that a texture needs to configure: its coordinates, the addressing mode (set to `repeat` here) and the filtering method (set to `linear` here). Next, we calculate the `normal` at each point on the sphere, then we compute the angles around the sphere using the normals. Finally, we calculate the `color` by sampling it instead of reading it as we did before. There is one more thing to do. In the kernel list of arguments, let's also reconfigure the texture access to `sample` instead of `read`. Replace this line:
 
{% highlight swift %}texture2d<float, access::read> input [[texture(1)]],
{% endhighlight %}
 
with this line:

{% highlight swift %}texture2d<float, access::sample> input [[texture(1)]],
{% endhighlight %}

The output image should look like this:

![alt text](https://github.com/MetalKit/images/raw/master/chapter15_4.gif "4")

Now this is what I call a realistic planet surface! The [source code](https://github.com/MetalKit/metal) is posted on Github as usual.

Until next time!
