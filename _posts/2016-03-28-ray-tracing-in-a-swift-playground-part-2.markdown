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
    let lower_left_corner = float3(x: -2.0, y: 1.0, z: -1.0) // Y is reversed
    let horizontal = float3(x: 4.0, y: 0, z: 0)
    let vertical = float3(x: 0, y: -2.0, z: 0)
    let origin = float3()
    for i in 0..<width {
        for j in 0..<height {
            let u = Float(i) / Float(width)
            let v = Float(j) / Float(height)
            let r = ray(origin: origin, direction: lower_left_corner + u * horizontal + v * vertical)
            let col = color(r)
            pixel = Pixel(red: UInt8(col.x * 255), green: UInt8(col.y * 255), blue: UInt8(col.z * 255))
            pixels[i + j * width] = pixel
        }
    }
    let bitsPerComponent = 8
    let bitsPerPixel = 32
    let rgbColorSpace = CGColorSpaceCreateDeviceRGB()
    let bitmapInfo = CGBitmapInfo(rawValue: CGImageAlphaInfo.PremultipliedLast.rawValue)
    let providerRef = CGDataProviderCreateWithCFData(NSData(bytes: pixels, length: pixels.count * sizeof(Pixel)))
    let image = CGImageCreate(width, height, bitsPerComponent, bitsPerPixel, width * sizeof(Pixel), rgbColorSpace, bitmapInfo, providerRef, nil, true, CGColorRenderingIntent.RenderingIntentDefault)
    return CIImage(CGImage: image!)
}
{% endhighlight %}

Let's time the execution again:

{% highlight swift %}
let width = 800
let height = 400
let t0 = CFAbsoluteTimeGetCurrent()
let image = imageFromPixels(width, height)
let t1 = CFAbsoluteTimeGetCurrent()
t1-t0
image
{% endhighlight %}

Much better! In my case the running time went from `5 seconds` down to only __0.1 seconds__. Ok, enough with the cleanup. Let's do some graphics instead! We would like to draw more than just one sphere, perhaps way many more spheres. One neat trick to simulate the horizon if to draw a really huge sphere. Then we can put our smaller sphere on top of it to achieve a `sitting-on-the-ground` effect. 

For this, we need to abstract our current sphere code into a generic class. Let's name it __objects.swift__ since we will probably create other type of volumes in future beside spheres. Next, inside `ray.swift` delete the __hit_sphere__ function since we are going to create a new, similar function inside `objects.swift`. Then, we need to create a new struct which represents a `hit` event:

{% highlight swift %}
struct hit_record {
    var t: Float
    var p: float3
    var normal: float3
    init() {
        t = 0.0
        p = float3(x: 0.0, y: 0.0, z: 0.0)
        normal = float3(x: 0.0, y: 0.0, z: 0.0)
    }
}
{% endhighlight %}

Next, we need to create a protocol named __hitable__ so various classes can conform to. This protocol only contains the __hit__ function:

{% highlight swift %}
protocol hitable {
    func hit(r: ray, _ tmin: Float, _ tmax: Float, inout _ rec: hit_record) -> Bool
}
{% endhighlight %}

The next obvious step is to implement a __sphere__ class:

{% highlight swift %}
class sphere: hitable  {
    var center = float3(x: 0.0, y: 0.0, z: 0.0)
    var radius = Float(0.0)
    init(c: float3, r: Float) {
        center = c
        radius = r
    }
    func hit(r: ray, _ tmin: Float, _ tmax: Float, inout _ rec: hit_record) -> Bool {
        let oc = r.origin - center
        let a = dot(r.direction, r.direction)
        let b = dot(oc, r.direction)
        let c = dot(oc, oc) - radius*radius
        let discriminant = b*b - a*c
        if discriminant > 0 {
            var t = (-b - sqrt(discriminant) ) / a
            if t < tmin {
                t = (-b + sqrt(discriminant) ) / a
            }
            if tmin < t && t < tmax {
                rec.t = t
                rec.p = r.point_at_parameter(rec.t)
                rec.normal = (rec.p - center) / float3(radius)
                return true
            }
        }
        return false
    }
}
{% endhighlight %}

As you might notice, the `hit` function is quite similar to the __hit_sphere__ function we deleted from the `ray.swift` file, except we are now looking at hits that only occur during the interval `tmax - tmin`.

Finally, we need a way to add multiple objects to a list. An array of `hitables` seems to be the right choice:

{% highlight swift %}
class hitable_list: hitable  {
    var list = [hitable]()
    func add(h: hitable) {
        list.append(h)
    }
    func hit(r: ray, _ tmin: Float, _ tmax: Float, inout _ rec: hit_record) -> Bool {
        var hit_anything = false
        for item in list {
            if (item.hit(r, tmin, tmax, &rec)) {
                hit_anything = true
            }
        }
        return hit_anything
    }
}
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing3.png "Raytracing 3")
![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing4.png "Raytracing 4")
![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing5.png "Raytracing 5")

{% highlight swift %}
{% endhighlight %}

Stay tuned for the next part of this series, where we will look into specular lights, transparency, refraction and reflection. The [source code](https://github.com/Swiftor/Raytracing) is posted on Github as usual.

Until next time!