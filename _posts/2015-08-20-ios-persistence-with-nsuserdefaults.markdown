---
published: false
title: iOS persistence with NSUserDefaults
layout: post
---
We are going to talk about the app lifecycle (the famous methods in the AppDelegate class), and how persistence is useful when transitioning between the various app states. In Storyboard add a button and a slider. Create a an action named __saveValue__ for the button and an outlet named __sliderView__ for the slider. Also create a global variable named __SliderValueKey__ to hold the slider value.

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

To test the code, run the app and notice that a 0 value is displayed in the console for the slider. Now move the slider to the left or to the right, then press the button to save the current value of the slider. The new value will be noted in the console. Next, close the app and run it again. As you expect, the start value will now be the last value we had before closing the app. This is the most basic form of persistence.

Let's look at how we can write and read files from disk, as another form of persistence.

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

The first time you run the app you will see in the console:
    Creating the file demo.txt on disk.
If you run the app again, this time you will see in the console:
    The file demo.txt already exists!

Until next time!