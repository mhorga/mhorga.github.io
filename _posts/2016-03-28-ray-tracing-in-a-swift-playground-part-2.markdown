---
published: false
title: Ray tracing in a Swift playground part 2
layout: post
---
Let's continue working on our `ray tracer` and pick up where we left off last week. I want to thank `Caroline`, `Jessy`, `Jeff` and `Mike` for providing valuable feedback and performance improvement suggestions while working on this project. 

First, as usual, we will do some code cleanup. In the first part we used the __vec3.swift__ class because we wanted to understand the underlying data structures and operations between them, however, there is already a framework called __simd__ which helps us do all the `math` we need. So rename `vec3.swift` to __ray.swift__ since this class will only contain code related to the `ray` struct. Next, delete the `vec3` struct as well as all the operations at the end. You should only retain the __ray__ struct, as well as the __color__ and __hit_sphere__ functions. 

Next, import the __simd__ framework and then replace `vec3` with __float3__ everywhere inside this file, and after that go to __pixel.swift__ and repeat this last step in there as well. We are now officially depending on __float3__ only! While still in the `pixel.swift` we need to address one more concern: passing the array between the two functions makes the rendering quite slow. Here is how to time your code in the playground:

{% highlight swift %}
let width = 800
let height = 400
let t0 = CFAbsoluteTimeGetCurrent()
var pixelSet = makePixelSet(width, height)
var image = imageFromPixels(pixelSet)
let t1 = CFAbsoluteTimeGetCurrent()
t1-t0
image
{% endhighlight %}

Notice it takes __5 seconds__ (at least that is my case). This happens because in `Swift` arrays are defined as structs actually, and structs are always passed `by value` in Swift which means a copy of the array will me made when passing it, and copying a huge array is a performance bottleneck. There are two ways to fix this. One, the most elegant, is to wrap everything inside a `class` and make the array a class `property`. This way, the array would not need to be passed anymore between the local functions. The second way, is easier to implement and we will go with this one to save time. All we need to do is combine the two functions into one like this:

{% highlight swift %}
public func imageFromPixels(width: Int, _ height: Int) -> CIImage {
    var pixel = Pixel(red: 0, green: 0, blue: 0)
    var pixels = [Pixel](count: width * height, repeatedValue: pixel)
    let lower_left_corner = vec3(x: -2.0, y: 1.0, z: -1.0)
    let horizontal = vec3(x: 4.0, y: 0, z: 0)
    let vertical = vec3(x: 0, y: -2.0, z: 0)
    let origin = vec3()
    for i in 0..<width {
        for j in 0..<height {
            let u = Double(i) / Double(width)
            let v = Double(j) / Double(height)
            let r = ray(origin: origin, direction: lower_left_corner + u * horizontal + v * vertical)
            let col = color(r)
            pixel = Pixel(red: UInt8(col.x * 255), green: UInt8(col.y * 255), blue: UInt8(col.z * 255))
            pixels[i + j * width] = pixel
        }
    }
    let bitsPerComponent = 8
    let bitsPerPixel = 32
    let rgbColorSpace = CGColorSpaceCreateDeviceRGB()
    let bitmapInfo = CGBitmapInfo(rawValue: CGImageAlphaInfo.PremultipliedLast.rawValue) // alpha is last
    let providerRef = CGDataProviderCreateWithCFData(NSData(bytes: pixels, length: pixels.count * sizeof(Pixel)))
    let image = CGImageCreate(width, height, bitsPerComponent, bitsPerPixel, width * sizeof(Pixel), rgbColorSpace, bitmapInfo, providerRef, nil, true, CGColorRenderingIntent.RenderingIntentDefault)
    return CIImage(CGImage: image!)
}
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/Swiftor/Raytracing/raw/master/images/raytracing.png "Raytracing 2")

{% highlight swift %}
{% endhighlight %}

Stay tuned for the next part of this series, where we will look into specular lights, transparency, refraction and reflection. The [source code](https://github.com/Swiftor/Raytracing) is posted on Github as usual.

Until next time!