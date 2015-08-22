---
published: true
title: iOS persistence with NSUserDefaults
layout: post
---
We are going to look into how persistence is useful in preserving data between closing and re-opening an app. In Storyboard add a button and a slider. Create a an action named __saveValue__ for the button and an outlet named __sliderView__ for the slider, and finally create a global variable named __SliderValueKey__ to hold the slider value.

User Defaults is the easiest way in iOS to save app information associated with user preferences. We can store primitive types and object types. It behaves like a dictionary — data is stored as keys and values. iOS keeps a database of these user preference values for each app on the device. Naturally, the values are persisted from one run of an app to another. The __NSUserDefaults__ class implements the `Singleton` design pattern. That means that there is a class method that will return a reference to the same user defaults object, regardless of where we invoke it in our program.

{% highlight swift %}
let SliderValueKey = "Slider Value Key"

@IBOutlet weak var sliderView: UISlider!

override func viewDidLoad() {
    super.viewDidLoad()
    sliderView.value = NSUserDefaults.standardUserDefaults().floatForKey(SliderValueKey)
    println("Slider value at load time: \(sliderView.value)")
}

@IBAction func saveValue(sender: UIButton) {
    println("Slider value at save time: \(sliderView.value)")
    NSUserDefaults.standardUserDefaults().setFloat(sliderView.value, forKey: SliderValueKey)
}
{% endhighlight %}

To test the code, run the app and notice that a value of `0.0` is displayed in the console for the slider. Now move the slider to the left or to the right, then press the button to save the current value of the slider. The new value will be noted in the console. Next, close the app and run it again. As you expect, the start value will now be the last value we had before closing the app. This is the most basic form of persistence.

Let's look at how we can write and read files from disk, as another form of persistence. Every time the user writes to disk, the Documents directory will be used.

{% highlight swift %}
override func viewDidLoad() {
    super.viewDidLoad()
    if NSFileManager.defaultManager().fileExistsAtPath(fileURL().path!) {
        println("The file \((fileURL().pathComponents?.last)!) already exists!")
    } else {
        println("Creating the file \((fileURL().pathComponents?.last)!) on disk.")
        NSFileManager.defaultManager().createFileAtPath(fileURL().path!, contents: nil, attributes: nil)
    }
}

func fileURL() ->  NSURL {
    let filename = "demo.txt"
    let dirPath = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true)[0] as! String
    let pathArray = [dirPath, filename]
    let fileURL =  NSURL.fileURLWithPathComponents(pathArray)!
    return fileURL
}
{% endhighlight %}

The first time you run the app you will see in the console:
{% highlight swift %}
    Creating the file demo.txt on disk.
{% endhighlight %}
If you run the app again, this time you will see in the console:
{% highlight swift %}
    The file demo.txt already exists!
{% endhighlight %}

You can read more about [NSUserDefaults](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/) and [NSFileManager](https://developer.apple.com/library/ios/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html) if you're interested in exploring these topics more.

Until next time!
