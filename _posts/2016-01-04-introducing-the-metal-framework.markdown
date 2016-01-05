---
published: false
title: Introducing the Metal framework
layout: post
---
As I promised last time, I am starting today a new series about the [__Metal framework__](https://developer.apple.com/metal/) that was announced at the [`WWDC 2014`](https://developer.apple.com/videos/play/wwdc2014-603/) initially only for `iOS`, but later on also for `OS X` and `tvOS`. `Metal` is an interface for accessing the `GPU (Graphics Processing Unit)` inside your machine. The main advantages of using `Metal` are:

- provides the lowest overhead access to the `GPU`, hence it reduces all bottlenecks usually caused by data transferring between the `CPU` and `GPU` in other frameworks. 
- provides up to __10__ times the number of draw calls compared to `OpenGL`. `Metal`, however, is not cross-platform as `OpenGL` is, so it is not meant to be a replacement for `OpenGL`.
- allows to also run `compute` applications with performance levels comparable to similar technologies such as `CUDA` or `OpenCL`.
- has custom shader language that allows shaders precompiling so they are a lot faster at run time. 
- has built-in memory and resource management, particularized to these platforms.

Since `Metal` does not run on the `Xcode` simulator, and since we cannot assume all our readers have an `iOS` device that has an __A7__ `CPU` or newer, we will rather create an `OS X` project instead. In `Xcode` create a `Cocoa Application`. In the storyboard, drag and drop a `Label` onto the `View Controller`. Center it, enlarge it so we can make sure we can display 2 lines of text, and set necessary constraints. When you're done, the storyboard should look like this: 

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter01_1.png "1")

Next, go to __ViewController.swift__ and create an `IBOutlet` for the label we just created. You can name it simply __label__ or anything else you wish. Finally, let's write some code. Your class should look like this:

{% highlight swift %} 
import Cocoa // contains Metal

class ViewController: NSViewController {

    @IBOutlet weak var label: NSTextField!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        let device = MTLCreateSystemDefaultDevice()
        label.stringValue = "Your GPU name is:\n\(device!.name!)"
    }
}
{% endhighlight %}

Let's explain the lines of code above.

If you run the project, you should be able to see the following output:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter01_2.png "2")

There is not much to see, but for now you learned how to "talk" to the `GPU` at the lowest possible level. In the next episode we will learn how we can send data to the `GPU` and get results back from it. The project [source code](https://github.com/Swiftor/Metal/tree/master/ch01) is posted on Github, as usual.

Until next time!