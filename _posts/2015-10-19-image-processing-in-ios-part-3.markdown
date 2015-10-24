---
published: true
title: Image processing in iOS part 3
layout: post
---
In the last part of this series, we look at how to manipulate pixel information to get great image effects. Let's take up where we left off in the previous part by creating a `RGBA` image from our `UIImage`:

{% highlight swift %} 
let image = UIImage(named: "image")!
var rgba = RGBA(image: image)!
{% endhighlight %}

The origin point `0 x 0` in the `UIKit` framework is at the top left corner of the screen while in the `Core Graphics` and `Core Image` frameworks is at the bottom left corner. This comes in handy when we need to debug issues such as when an _upside-down_ image resulted unexpectedly. We have access to the 2D-array of pixels which we can easily access by creating an index:

{% highlight swift %} 
let index = y * rgba.width + x
{% endhighlight %}

So for example if we need to access the pixel located at `15 x 20` we would need to vertically move the index `20` rows down the `Y` axis, and horizontally `15` columns to the right, on the X axis. In other words, if we had an image sized `100 x 150` we would need to traverse `20` full rows and `15` more pixels on the next row, and the index would have a value of `2015 (= 20 x 100 + 15)`. Interesting coincidence! ;-)

Now we are able to access the pixel at the desired index, like this:

{% highlight swift %} 
var pixel = rgba.pixels[index]
{% endhighlight %}

If you remember from last time, the `RGBA` struct has a pointer to a `Pixel` type which acts like an array. The next thing we would like is to see what `color` and `alpha` values the pixel stores, and eventually change some of these values, like for example set the pixel color to full `green`, store the new color and then look at the image to see our glorious `green` pixel:

{% highlight swift %} 
pixel.red = 0
pixel.green = 255
pixel.blue = 0
rgba.pixels[index] = pixel
rgba = RGBA(image: image)!
{% endhighlight %}

Next, let's see what are the average color values for the entire image. For this we just add all the values for the same color and then divide it by the total number of pixels:

{% highlight swift %} 
var totalRed = 0
var totalGreen = 0
var totalBlue = 0

for y in 0..<rgba.height {
    for x in 0..<rgba.width {
        let index = y * rgba.width + x
        let pixel = rgba.pixels[index]
        totalRed += Int(pixel.red)
        totalGreen += Int(pixel.green)
        totalBlue += Int(pixel.blue)
    }
}

let pixelCount = rgba.width * rgba.height
let avgRed = totalRed / pixelCount
let avgGreen = totalGreen / pixelCount
let avgBlue = totalBlue / pixelCount
{% endhighlight %}

Now that we have the averages, we can try to get some image effect, such as for example increasing `contrast` by enhancing the pixel color value: 

{% highlight swift %} 
func contrast(image: RGBA) -> RGBA {
    for y in 0..<image.height {
        for x in 0..<image.width {
            let index = y * image.width + x
            var pixel = image.pixels[index]
            let redDelta = Int(pixel.red) - avgRed
            let greenDelta = Int(pixel.green) - avgGreen
            let blueDelta = Int(pixel.blue) - avgBlue
            pixel.red = UInt8(max(min(255, avgRed + 3 * redDelta), 0))
            pixel.green = UInt8(max(min(255, avgGreen + 3 * greenDelta), 0))
            pixel.blue = UInt8(max(min(255, avgBlue + 3 * blueDelta), 0))
            image.pixels[index] = pixel
        }
    }
    return image
}
{% endhighlight %}

What we do inside this method is to calculate a difference between the current color value and the color average, and then multiply it by `3` to enhance the color. The higher the multiplier value is, the more intense the contrast will be. In order to make sure we never get a negative value for the color, we use the `clamp` technique to always get a positive value and one that is always below `256`. Now apply the effect to the image:

{% highlight swift %} 
let newImage = contrast(rgba).toUIImage()
{% endhighlight %}

You should be able to see now the image with a nice contrast applied to it. The source code is available on [Github](https://github.com/Swiftor/ImageProcessing). A special `Thanks` to [@JackTripleU](https://twitter.com/JackTripleU) who inspired me to write this series.

Until next time!