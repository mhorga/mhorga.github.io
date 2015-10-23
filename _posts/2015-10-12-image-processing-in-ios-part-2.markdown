---
published: false
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

We need a `Pixel` struct to represent an image, and also its `width` and `height`.

{% highlight swift %} 
init?(image: UIImage) {
    guard let cgImage = image.CGImage else { return nil }
    width = Int(image.size.width)
    height = Int(image.size.height)
    let bitsPerComponent = 8
    let bytesPerPixel = 4
    let bytesPerRow = width * bytesPerPixel
    let imageData = UnsafeMutablePointer<Pixel>.alloc(width * height)
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    var bitmapInfo: UInt32 = CGBitmapInfo.ByteOrder32Big.rawValue
    bitmapInfo |= CGImageAlphaInfo.PremultipliedLast.rawValue & CGBitmapInfo.AlphaInfoMask.rawValue
    guard let imageContext = CGBitmapContextCreate(imageData, width, height, bitsPerComponent, bytesPerRow, colorSpace, bitmapInfo) else { return nil }
    CGContextDrawImage(imageContext, CGRect(origin: CGPointZero, size: image.size), cgImage)
    pixels = UnsafeMutableBufferPointer<Pixel>(start: imageData, count: width * height)
}

func toUIImage() -> UIImage? {
    let bitsPerComponent = 8
    let bytesPerPixel = 4
    let bytesPerRow = width * bytesPerPixel
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    var bitmapInfo: UInt32 = CGBitmapInfo.ByteOrder32Big.rawValue
    bitmapInfo |= CGImageAlphaInfo.PremultipliedLast.rawValue & CGBitmapInfo.AlphaInfoMask.rawValue
    let imageContext = CGBitmapContextCreateWithData(pixels.baseAddress, width, height, bitsPerComponent, bytesPerRow, colorSpace, bitmapInfo, nil, nil)
    guard let cgImage = CGBitmapContextCreateImage(imageContext) else {return nil}
    let image = UIImage(CGImage: cgImage)
    return image
}
{% endhighlight %}