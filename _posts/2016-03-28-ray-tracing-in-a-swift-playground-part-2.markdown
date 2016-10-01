---
published: true
title: Ray tracing in a Swift playground part 2
layout: post
---
Let's continue working on our `ray tracer` and pick up where we left off last week. I want to thank `Caroline`, `Jessy`, `Jeff` and `Mike` for providing valuable feedback and performance improvement suggestions while working on this project. 

First, as usual, we will do some code cleanup. In the first part we used the __vec3.swift__ class because we wanted to understand the underlying data structures and operations between them, however, there is already a framework called __simd__ which helps us do all the `math` we need. So rename `vec3.swift` to __ray.swift__ since this class will only contain code related to the `ray` struct. Next, delete the `vec3` struct as well as all the operations at the end. You should only retain the __ray__ struct, as well as the __color__ function. 

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

Notice it takes __5 seconds__ (at least that is my case). This happens because in `Swift` arrays are defined as structs actually, and structs are always passed `by value` in Swift which means a copy of the array will me made when passing it, and copying a huge array is a performance bottleneck. There are two ways to fix this. One, the most elegant, is to wrap everything inside a `class` and make the array a class `property`. This way, the array would not need to be passed anymore between the local functions. The second way, is easier to implement and we will go with this one to save space in this article. All we need to do is combine the two functions into one like this:

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

For this, we need to abstract our current sphere code into a generic class. Let's name it __objects.swift__ since we will probably create other type of volumes in future beside spheres. Next, inside `objects.swift` we need to create a new struct which represents a `hit` event:

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

As you might notice, the `hit` function is quite similar to the __hit_sphere__ function we deleted from `ray.swift`, except we are now looking at hits that only occur during the interval `tmax - tmin`. Next, we need a way to add multiple objects to a list. An array of `hitables` seems to be the right choice:

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

Back to `ray.swift`, we need to modify the `color` function to factor a `hit-record` variable into the color calculation:

{% highlight swift %}
func color(r: ray, world: hitable) -> float3 {
    var rec = hit_record()
    if world.hit(r, 0.0, Float.infinity, &rec) {
        return 0.5 * float3(rec.normal.x + 1, rec.normal.y + 1, rec.normal.z + 1);
    } else {
        let unit_direction = normalize(r.direction)
        let t = 0.5 * (unit_direction.y + 1)
        return (1.0 - t) * float3(x: 1, y: 1, z: 1) + t * float3(x: 0.5, y: 0.7, z: 1.0)
    }
}
{% endhighlight %}

Finally, back to `pixel.swift` we need to change the `imageFromPixels` function to allow the including of more objects:

{% highlight swift %}
public func imageFromPixels(width: Int, _ height: Int) -> CIImage {
    ...
    let world = hitable_list()
    var object = sphere(c: float3(x: 0, y: -100.5, z: -1), r: 100)
    world.add(object)
    object = sphere(c: float3(x: 0, y: 0, z: -1), r: 0.5)
    world.add(object)
    for i in 0..<width {
        for j in 0..<height {
            let u = Float(i) / Float(width)
            let v = Float(j) / Float(height)
            let r = ray(origin: origin, direction: lower_left_corner + u * horizontal + v * vertical)
            let col = color(r, world: world)
            pixel = Pixel(red: UInt8(col.x * 255), green: UInt8(col.y * 255), blue: UInt8(col.z * 255))
            pixels[i + j * width] = pixel
        }
    }
    ...
}
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing3.png "Raytracing 3")

Nice! If you look closely you will notice the edges exhibit the `aliasing` effect, and this happens because we do not have any blending of colors for the pixels on the edge. To overcome this, we need to sample the color multiple times by randomly generating values that are within the range we want, so we can blend them together and achieve an `anti-aliasing` effect. 

But first, let's also create a __camera__ class inside `ray.swift` as it will turn handy later. Practically, we just move the improvised camera we had inside the `imageFromPixels` function and put it in its right place:

{% highlight swift %}
struct camera {
    let lower_left_corner: float3
    let horizontal: float3
    let vertical: float3
    let origin: float3
    init() {
        lower_left_corner = float3(x: -2.0, y: 1.0, z: -1.0)
        horizontal = float3(x: 4.0, y: 0, z: 0)
        vertical = float3(x: 0, y: -2.0, z: 0)
        origin = float3()
    }
    func get_ray(u: Float, _ v: Float) -> ray {
        return ray(origin: origin, direction: lower_left_corner + u * horizontal + v * vertical - origin);
    }
}
{% endhighlight %}

The `imageFromPixels` function now looks like this:

{% highlight swift %}
public func imageFromPixels(width: Int, _ height: Int) -> CIImage {
    ...
    let cam = camera()
    for i in 0..<width {
        for j in 0..<height {
            let ns = 100
            var col = float3()
            for _ in 0..<ns {
                let u = (Float(i) + Float(drand48())) / Float(width)
                let v = (Float(j) + Float(drand48())) / Float(height)
                let r = cam.get_ray(u, v)
                col += color(r, world)
            }
            col /= float3(Float(ns));
            pixel = Pixel(red: UInt8(col.x * 255), green: UInt8(col.y * 255), blue: UInt8(col.z * 255))
            pixels[i + j * width] = pixel
        }
    }
    ...
}
{% endhighlight %}

Notice that we use a variable named __ns__ and assign a value of __100__ to it so we can sample the color multiple times using randomly generated values as we discussed above. In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing4.png "Raytracing 4")

Much better looking! However, we notice our rendering took __7 seconds__ which can be reduced by using a smaller sample value, such as __10__. Alright, now that we have multiple rays per pixel, we can finally think of creating `matte` (diffuse) materials. This kind of materials do not emit any light and usually absorb all the light that is directed towards them and blend it with their own color. The light that reflects of a diffuse material has its direction randomized. We can compute this with the following function inside `objects.swift`:

{% highlight swift %}
func random_in_unit_sphere() -> float3 {
    var p = float3()
    repeat {
        p = 2.0 * float3(x: Float(drand48()), y: Float(drand48()), z: Float(drand48())) - float3(x: 1, y: 1, z: 1)
    } while dot(p, p) >= 1.0
    return p
}
{% endhighlight %}

Then, back to `ray.swift` we need to modify the `color` function to factor the new random function into the color calculation:

{% highlight swift %}
func color(r: ray, _ world: hitable) -> float3 {
    var rec = hit_record()
    if world.hit(r, 0.0, Float.infinity, &rec) {
        let target = rec.p + rec.normal + random_in_unit_sphere()
        return 0.5 * color(ray(origin: rec.p, direction: target - rec.p), world)
    } else {
        let unit_direction = normalize(r.direction)
        let t = 0.5 * (unit_direction.y + 1)
        return (1.0 - t) * float3(x: 1, y: 1, z: 1) + t * float3(x: 0.5, y: 0.7, z: 1.0)
    }
}
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing5.png "Raytracing 5")

If you forgot to decrease `ns` from `100` to `10` your rendering took somewhat around __18 seconds__! However, if you decreased the value, the rendering time is down to only about __1.9 seconds__ which is not too shabby for a basic matte surface ray tracer.

This image looks great, however, we can also get rid of those small ripples easily. Notice that inside the `color` function we set `Tmin` to be __0.0__ and this seems to disturb some of the cases where the color needs to be computed correctly. If we set `Tmin` to be very small but still positive, something like __0.01__, you will notice that the difference is highly noticeable!

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing6.png "Raytracing 6")

Now, this image looks gorgeous! Stay tuned for the next part of this series, where we will look into topics such as specular lights, transparency, refraction and reflection. The [source code](https://github.com/mhorga/Raytracing2) is posted on Github as usual.

Until next time!