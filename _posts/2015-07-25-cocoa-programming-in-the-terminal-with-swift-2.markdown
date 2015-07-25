---
published: true
title: Cocoa programming in the terminal with Swift 2
layout: post
---
I was trying to make [PracticalSwift](http://practicalswift.com/2014/06/27/a-minimal-webkit-browser-in-30-lines-of-swift/?replytocom=927#respond)'s simple web browser work with __Swift 2.0__ and it turned out the changes were only minimal. In order to be able to run __Cocoa__ programs written in Swift 2.0 you still need to install _Xcode 7_ but we will not use Xcode at all in this article. Check that you have Swift 2.0 installed, and find where it is located:

{% highlight swift %}
$  swift -version
Apple Swift version 2.0 (swiftlang-700.0.47.4 clang-700.0.59.1)
Target: x86_64-apple-darwin15.0.0
$  which swift
/usr/bin/swift
{% endhighlight %}

Now create a new Swift file in the terminal:

{% highlight swift %}
$ vi browser.swift
{% endhighlight %}

As a quick __vi__ refresher, to insert text use the __i__ key, and when you are done press the __Esc__ key to exit the from the _edit mode__ and then type __:wq__ to save and quit _vi_. Make sure the first line contains the __shebang__ symbol __(#!)__ followed by the path we found above. This line tells the Swift compiler to run the file without compiling it:

{% highlight swift %}
#!/usr/bin/swift
{% endhighlight %}

Next we importing _WebKit_ and set up the application. We could have just imported Cocoa because that is all we need to run a graphical Cocoa application, but we also need WebKit later, and it turns out WebKit already imports Cocoa so we can save one redundant import this way. Every Cocoa app needs exactly one instance of _NSApplication_ instantiated. Then we set the activation policy of this app to __regular__ which means this app will appear in the _Dock_:

{% highlight swift %}
import WebKit
let application = NSApplication.sharedApplication()
application.setActivationPolicy(NSApplicationActivationPolicy.Regular)
{% endhighlight %}

Then we create a window for our web browser, set its size and style, center it, set a title for it, and finally show it:

{% highlight swift %}
let window = NSWindow()
window.setContentSize(NSSize(width:800, height:600))
window.styleMask = NSTitledWindowMask | NSClosableWindowMask | NSMiniaturizableWindowMask
window.center()
window.title = "Minimal Swift WebKit Browser"
window.makeKeyAndOrderFront(window)
{% endhighlight %}

Next we create and connect a window delegate that will terminate the application when the close window button is clicked:

{% highlight swift %}
class WindowDelegate: NSObject, NSWindowDelegate {
    func windowWillClose(notification: NSNotification) {
        NSApplication.sharedApplication().terminate(0)
    }
}
let windowDelegate = WindowDelegate()
window.delegate = windowDelegate
{% endhighlight %}

Finally, we create an application delegate that we need in order to create the WebView object. Then we connect the application delegate, and set the app up so we can run it:

{% highlight swift %}
class ApplicationDelegate: NSObject, NSApplicationDelegate {
    var _window: NSWindow
    init(window: NSWindow) {
        self._window = window
    }
    func applicationDidFinishLaunching(notification: NSNotification) {
        let webView = WebView(frame: self._window.contentView.frame)
        self._window.contentView.addSubview(webView)
        webView.mainFrame.loadRequest(NSURLRequest(URL: NSURL(string: "https://www.google.com/ncr")!))
    }
}
let applicationDelegate = ApplicationDelegate(window: window)
application.delegate = applicationDelegate
application.activateIgnoringOtherApps(true)
application.run()
{% endhighlight %}

Exit the _edit mode_, save and quit _vi_. Finally, change the permissions for our file to make it executable, and then run it. 

{% highlight swift %}
$ chmod 755 browser.swift
$ ./browser.swift
{% endhighlight %}

There is our beautiful web browser :-)

Until next time!