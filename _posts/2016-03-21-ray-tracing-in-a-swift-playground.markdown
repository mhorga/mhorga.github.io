---
published: true
title: Ray tracing in a Swift playground
layout: post
---
Today we are porting a `ray tracer` from [Peter Shirley's mini book](http://www.amazon.com/Ray-Tracing-Weekend-Peter-Shirley-ebook/dp/B01B5AODD8), into a `Swift` playground. Since I am not going to describe what __Ray Tracing__ is and how it works, I am inviting you to go ahead and read this book as it's free for Kindle subscribers. If you are not a subscriber, just buy the book like I did. Paying $2.99 is absolutely worth your every penny if you are interested in this topic.

The first thing we would want to do is create a data structure to hold pixel information. In the playground, create a new file named __pixel.swift__ under the `Sources` folder. Next, let's write the __Pixel__ struct. It's just a simple struct with one variable for each of the [RGBA](https://en.wikipedia.org/wiki/RGBA_color_space) channels. We initialize the `alpha` channel to __255__ which means completely opaque on a `[0 - 255]` scale:

{% highlight swift %}
public struct Pixel {
    var r: UInt8
    var g: UInt8
    var b: UInt8
    var a: UInt8
    init(red: UInt8, green: UInt8, blue: UInt8) {
        r = red
        g = green
        b = blue
        a = 255
    }
}
{% endhighlight %}

Next, we need to create an array to hold all the pixels on the screen. To compute the color for each pixel we just set `Red` to be __0__ for all pixels, while `Green` goes from __0__ (no `Green` at all) in the left corner of the screen, to __255__ (full `Green`) in the right corner of the screen. Similarly, the `Blue` color goes from __0__ at the top of the screen, to __255__ at the bottom of the screen.

{% highlight swift %}
public func makePixelSet(width: Int, _ height: Int) -> ([Pixel], Int, Int) {
    var pixel = Pixel(red: 0, green: 0, blue: 0)
    var pixels = [Pixel](count: width * height, repeatedValue: pixel)
    for i in 0..<width {
        for j in 0..<height {
            pixel = Pixel(red: 0, green: UInt8(Double(i * 255 / width)), blue: UInt8(Double(j * 255 / height)))
            pixels[i + j * width] = pixel
        }
    }
    return (pixels, width, height)
}
{% endhighlight %}

Finally, we need a way to create a drawable image from pixels. The __Core Image__ framework provides the __CGImageCreate()__ method, which takes in several parameters necessary for image rendering:

{% highlight swift %}
public func imageFromPixels(pixels: ([Pixel], width: Int, height: Int)) -> CIImage {
    let bitsPerComponent = 8
    let bitsPerPixel = 32
    let rgbColorSpace = CGColorSpaceCreateDeviceRGB()
    let bitmapInfo = CGBitmapInfo(rawValue: CGImageAlphaInfo.PremultipliedLast.rawValue) // alpha is last
    let providerRef = CGDataProviderCreateWithCFData(NSData(bytes: pixels.0, length: pixels.0.count * sizeof(Pixel)))
    let image = CGImageCreate(pixels.1, pixels.2, bitsPerComponent, bitsPerPixel, pixels.1 * sizeof(Pixel), rgbColorSpace, bitmapInfo, providerRef, nil, true, CGColorRenderingIntent.RenderingIntentDefault)
    return CIImage(CGImage: image!)
}
{% endhighlight %}

Next, in the main playground page create the set of pixels for a window with given `width` and `height`, and then create the rendered image using this set:

{% highlight swift %}
let width = 800
let height = 400
var pixelSet = makePixelSet(width, height)
var image = imageFromPixels(pixelSet)
image
{% endhighlight %}

You should see the following image:

![alt text](https://github.com/MetalKit/images/blob/master/raytracing1.png?raw=true "Raytracing 1")

Ok, you might be wondering, but where does `ray tracing` come into play? Let's deal with that next. Under the `Sources` folder again, let's create a convenience class named __vec3.swift__ where we will have a few `math` helper methods:

{% highlight swift %}
struct vec3 {
    var x = 0.0
    var y = 0.0
    var z = 0.0
}

func * (left: Double, right: vec3) -> vec3 {
    return vec3(x: left * right.x, y: left * right.y, z: left * right.z)
}

func + (left: vec3, right: vec3) -> vec3 {
    return vec3(x: left.x + right.x, y: left.y + right.y, z: left.z + right.z)
}

func - (left: vec3, right: vec3) -> vec3 {
    return vec3(x: left.x - right.x, y: left.y - right.y, z: left.z - right.z)
}

func dot (left: vec3, _ right: vec3) -> Double {
    return left.x * right.x + left.y * right.y + left.z * right.z
}

func unit_vector(v: vec3) -> vec3 {
    let length : Double = sqrt(dot(v, v))
    return vec3(x: v.x/length, y: v.y/length, z: v.z/length)
}
{% endhighlight %}

Next, we need to create a __ray__ struct. It has as members an `origin`, a `direction` and a method that computes the `ray tracing` equation for any given parameter:

{% highlight swift %}
struct ray {
    var origin: vec3
    var direction: vec3
    func point_at_parameter(t: Double) -> vec3 {
        return origin + t * direction
    }
}
{% endhighlight %}

Then we need to compute the __color__ based on whether the `ray` hits a `sphere` which we create in the center of the screen:

{% highlight swift %}
func color(r: ray) -> vec3 {
    let minusZ = vec3(x: 0, y: 0, z: -1.0)
    var t = hit_sphere(minusZ, 0.5, r)
    if t > 0.0 {
        let norm = unit_vector(r.point_at_parameter(t) - minusZ)
        return 0.5 * vec3(x: norm.x + 1.0, y: norm.y + 1.0, z: norm.z + 1.0)
    }
    let unit_direction = unit_vector(r.direction)
    t = 0.5 * (unit_direction.y + 1.0)
    return (1.0 - t) * vec3(x: 1.0, y: 1.0, z: 1.0) + t * vec3(x: 0.5, y: 0.7, z: 1.0)
}
{% endhighlight %}

You noticed that we used another method, called __hit_sphere()__ to determine whether we hit the sphere with the ray, or whether we miss it:

{% highlight swift %}
func hit_sphere(center: vec3, _ radius: Double, _ r: ray) -> Double {
    let oc = r.origin - center
    let a = dot(r.direction, r.direction)
    let b = 2.0 * dot(oc, r.direction)
    let c = dot(oc, oc) - radius * radius
    let discriminant = b * b - 4 * a * c
    if discriminant < 0 {
        return -1.0
    } else {
        return (-b - sqrt(discriminant)) / (2.0 * a)
    }
}
{% endhighlight %}

Back in the __pixel.swift__ file, change the __makePixelSet(:)__ to include creating a `ray` for each pixel and compute its `color` before adding the pixel to the set:

{% highlight swift %}
public func makePixelSet(width: Int, _ height: Int) -> ([Pixel], Int, Int) {
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
    return (pixels, width, height)
}
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/MetalKit/images/blob/master/raytracing2.png?raw=true "Raytracing 2")

Stay tuned for a part 2 of this article, where we will calculate lights and shades for a more realistic image rendering. The [source code](https://github.com/MetalKit/raytracing) is posted on Github as usual.

Until next time!