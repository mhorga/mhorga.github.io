---
published: false
title: Cocoa programming in the terminal with Swift 2
layout: post
---
I was trying to make [PracticalSwift](http://practicalswift.com/2014/06/27/a-minimal-webkit-browser-in-30-lines-of-swift/?replytocom=927#respond)'s simple browser work with Swift 2.0 and it turned out the changes were only minimal. In order to be able to run Cocoa programs written in Swift 2.0 you still need to install Xcode 7 but we will not use Xcode at all in this article. Check if you have Swift 2.0 installed and where it is located:

{% highlight swift %}
$  swift -version
Apple Swift version 2.0 (swiftlang-700.0.47.4 clang-700.0.59.1)
Target: x86_64-apple-darwin15.0.0
$  which swift
/usr/bin/swift
{% endhighlight %}

Now create a new Swift file and make sure the first line contains the __shebang__ symbol __(#!)__ followed by the path we found above. This line tells the Swift compiler to run the file without compiling it:

{% highlight swift %}
#!/usr/bin/swift
{% endhighlight %}

Next we importing WebKit and set up the application. We could have just imported Cocoa because that is all we need to run a graphical Cocoa application, but we also need WebKit later, and it turns out WebKit already imports Cocoa so we can save one redundant import this way. Every Cocoa app needs exactly one instance of NSApplication instantiated. Then we set the activation policy for this app to _regular_ which means this app will appear in the Dock:

{% highlight swift %}
import WebKit
let application = NSApplication.sharedApplication()
application.setActivationPolicy(NSApplicationActivationPolicy.Regular)
{% endhighlight %}

Until next time!