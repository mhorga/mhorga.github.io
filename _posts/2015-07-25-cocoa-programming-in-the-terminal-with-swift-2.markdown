---
published: false
title: Cocoa programming in the terminal with Swift 2
layout: post
---
In order to be able to run Cocoa programs written in Swift 2.0 you still need to install Xcode 7 but we will not use Xcode at all in this article. Check if you have Swift 2.0 installed and where it is located:

{% highlight swift %}     
$  swift -version
Apple Swift version 2.0 (swiftlang-700.0.47.4 clang-700.0.59.1)
Target: x86_64-apple-darwin15.0.0
$  which swift
/usr/bin/swift
{% endhighlight %}

Now create a new Swift file and make sure the first line contains the __shebang__ symbol which tells the Swift compiler to run this file without compiling it:

{% highlight swift %}     
#!/usr/bin/swift
{% endhighlight %}

Next

Until next time!