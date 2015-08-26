---
published: false
title: iOS persistence with NSCoder and NSKeyedArchiver
layout: post
---
As you have seen in the last post, [NSUserDefaults](http://mhorga.org/2015/08/20/ios-persistence-with-nsuserdefaults.html) is not as sophisticated as we want it to be in more complex scenarios such as persisting an `object graph` like an array of arrays, for example. Often times we get into situations where we do not see app persistence even though we should. 

Let's start with an example. When you create a `Master-Detail` project in `Xcode` you will notice that all the date entries you create will be lost if you run the app again. If we take a closer look at the code, inside `MasterViewController` we notice that a mutable array named __objects__ is used to store objects of the type `NSDate` every time we click the __+__ button at the right end of the navigation bar. So `objects` will be a proper object graph candidate for us to persist.

When the app starts for the first time there is no file to store the array, so we first need to construct the full path. We can add a calculated `Swift` property to the `MasterViewController` that specifies the location of a file name we choose, and then place it inside the `Documents` directory:

{% highlight swift %}
var filePath : String {
    let manager = NSFileManager.defaultManager()
    let url = manager.URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask).first as! NSURL
    return url.URLByAppendingPathComponent("objectsArray").path!
}
{% endhighlight %}

The root of the graph for this app is the __objects__ array. We can persist the array using a class method on the `NSKeyedArchiver` and add this line to the __insertNewObject()__ method:

{% highlight swift %}
NSKeyedArchiver.archiveRootObject(objects, toFile: filePath)
{% endhighlight %}

To unarchive the array from the file we use a conditional optional check in case our file does not exist, and add it to the __viewDidLoad()__ method:

{% highlight swift %}
if let array = NSKeyedUnarchiver.unarchiveObjectWithFile(filePath) as? [AnyObject] {
    objects = array
}
{% endhighlight %}

Now run the app, create a few date entries, close the app and run it again. You should now see that the array is being persisted. That was easy. But what about complex objects graphs such as arrays of custom objects? In this case we need to complement the use of `NSKeyedArchiver` and `NSKeyedUnarchiver` with protocols they will need to implement, __NSCoder__ and __NSDecoder__ respectively.

Until next time!