---
published: true
title: iOS persistence with NSCoder and NSKeyedArchiver
layout: post
---
As you have seen in the last post, [NSUserDefaults](http://mhorga.org/2015/08/20/ios-persistence-with-nsuserdefaults.html) is not as sophisticated as we want it to be in more complex scenarios such as persisting an `object graph` like an array of arrays, for example. Often times we get into situations where we do not see app persistence even though we should. 

Let's start with an example. When you create a `Master-Detail` project in `Xcode` you will notice that all the date entries you create will be lost if you run the app again. If we look at the code inside `MasterViewController` we notice that a mutable array named __objects__ is used to store objects of the type `NSDate` every time we click the __+__ button at the right end of the navigation bar. So `objects` will be a proper object graph candidate for us to persist.

When the app starts for the first time there is no file to store the array, so we first need to construct the full path. Let's add a calculated `Swift` property to `MasterViewController` that specifies the location of a file name we choose, and then place it inside the `Documents` directory:

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

Now run the app, create a few date entries, close the app and run it again. You should now see that the array is being persisted. That was easy. But what about complex objects graphs such as arrays of custom objects? In this case we need to complement `NSKeyedArchiver` and `NSKeyedUnarchiver` with the __NSCoding__ protocol that we will need to implement. 

Let's create a new __Person__ class:

{% highlight swift %}
class Person : NSObject, NSCoding {
 
    struct Keys {
        static let Name = "name"
        static let Age = "age"
    }
    
    var name = ""
    var age = 0
    
    init(dictionary: [String : AnyObject]) {
        name = dictionary[Keys.Name] as! String
        age = dictionary[Keys.Age] as! Int
    }
    
    func encodeWithCoder(archiver: NSCoder) {
        archiver.encodeObject(name, forKey: Keys.Name)
        archiver.encodeObject(age, forKey: Keys.Age)
    }

    required init(coder unarchiver: NSCoder) {
        super.init()
        name = unarchiver.decodeObjectForKey(Keys.Name) as! String
        age = unarchiver.decodeObjectForKey(Keys.Age) as! Int
    }
}
{% endhighlight %}

You noticed we conformed this class to the `NSCoding` protocol so we needed to implement the two methods the protocol requires, __init(coder:)__ and __encodeWithCoder(archiver:)__. Assume that we have another array __persons__ in `MasterViewController` and another method that adds `Person` objects to this array. Having the `Person` class set up with an archiver for saving data and an unarchiver for retrieving saved data makes our task as easy as calling the archiver in the `viewWillAppear()` method:

{% highlight swift %}
NSKeyedArchiver.archiveRootObject(persons, toFile: filePath)
{% endhighlight %}

and the unarchiver in the `viewDidLoad()` method:

{% highlight swift %}
persons = NSKeyedUnarchiver.unarchiveObjectWithFile(filePath) as? [Person] ?? [Person]()
{% endhighlight %}

Stay tuned for the last part of this series.

Until next time!
