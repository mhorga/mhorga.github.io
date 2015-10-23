---
published: true
title: Image processing in iOS part 2
layout: post
---
Last week we looked into how we could store pixel information in proper data structures. This week we will continue the image processing topic by looking into how to convert a `UIImage` into a `Core Graphics` image, change some pixel values to obtain nice effects, and then convert it back into a `UIImage`. In this part of the series we will only look at converting the `UIImage` into a `CGImage` and then back into a `UIImage`, leaving the fun part for next week.

Let's create a new `struct` named __RGBA__:

{% highlight swift %} 
struct RGBA {
    var pixels: UnsafeMutableBufferPointer<Pixel>
    var width: Int
    var height: Int
}
{% endhighlight %}

We need a pointer of `Pixel` type to represent an image as an array, and also its `width` and `height`. Next we need an __init()__ method so we can easily create `RGBA` structs:

{% highlight swift %} 
init?(image: UIImage) {
    guard let cgImage = image.CGImage else { return nil } // 1
    width = Int(image.size.width)
    height = Int(image.size.height)
    let bitsPerComponent = 8 // 2
    let bytesPerPixel = 4
    let bytesPerRow = width * bytesPerPixel
    let imageData = UnsafeMutablePointer<Pixel>.alloc(width * height)
    let colorSpace = CGColorSpaceCreateDeviceRGB() // 3
    var bitmapInfo: UInt32 = CGBitmapInfo.ByteOrder32Big.rawValue
    bitmapInfo |= CGImageAlphaInfo.PremultipliedLast.rawValue & CGBitmapInfo.AlphaInfoMask.rawValue
    guard let imageContext = CGBitmapContextCreate(imageData, width, height, bitsPerComponent, bytesPerRow, colorSpace, bitmapInfo) else { return nil }
    CGContextDrawImage(imageContext, CGRect(origin: CGPointZero, size: image.size), cgImage) // 4
    pixels = UnsafeMutableBufferPointer<Pixel>(start: imageData, count: width * height)
}
{% endhighlight %}

Let's break this method into pieces so we can better understand what it does. First, we convert the `UIImage` to a `CGImage` object, and get the image’s `width` and `height`. Second, for the 32-bit `RGBA` color space, we define a few parameters, calculate the number of `bytesPerRow` of the image, and we allocate an array pixels to store the pixel data. Third, we redraw image for correct pixel format by creating an `RGB` color space and a `CGBitmapContext`, passing in the pixels pointer as the buffer to store the pixel data this context holds. And fourth, we draw the input image into the context and this populates pixels with the pixel data of the image in the format we specified when we created the context.

Finally, we need one more method to convert the `CGImage` back to a `UIImage`:

{% highlight swift %} 
func toUIImage() -> UIImage? {
    let bitsPerComponent = 8 // 1
    let bytesPerPixel = 4
    let bytesPerRow = width * bytesPerPixel
    let colorSpace = CGColorSpaceCreateDeviceRGB() // 2
    var bitmapInfo: UInt32 = CGBitmapInfo.ByteOrder32Big.rawValue
    bitmapInfo |= CGImageAlphaInfo.PremultipliedLast.rawValue & CGBitmapInfo.AlphaInfoMask.rawValue
    let imageContext = CGBitmapContextCreateWithData(pixels.baseAddress, width, height, bitsPerComponent, bytesPerRow, colorSpace, bitmapInfo, nil, nil)
    guard let cgImage = CGBitmapContextCreateImage(imageContext) else {return nil} // 3
    let image = UIImage(CGImage: cgImage)
    return image
}
{% endhighlight %}

Let's also break this method into pieces to explain what the code does, but you will notice it is very similar to the code in the `init()` method. First, for the 32-bit `RGBA` color space, we define parameters and calculate the `bytesPerRow` of the image. Second, we create an `RGB` color space, a CGBitmapContext, and pass in the pixels pointer as the buffer to store the pixel data this context holds. And third, we create and return a new `UIImage`.

Now add an image to your project/playground which is named __image.png__ for example, and then add these lines:

{% highlight swift %} 
let image = UIImage(named: "image")!
let rgba = RGBA(image: image)!
let newImage = rgba.toUIImage()
{% endhighlight %}

If you used a `Swift` playground, you should be able to see the same image rendered twice, which means the conversion worked well in both directions. In the last part of this series we will look into how to manipulate pixel info to get nice image effects.

Until next time!