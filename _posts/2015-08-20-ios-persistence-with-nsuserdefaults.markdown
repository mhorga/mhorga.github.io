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