---
published: true
title: View navigation in iOS
layout: post
---
By the end of this article you will have completed a really fun project, while also learning how navigation works in iOS. Start by creating a new _Single View Application_ project in _Xcode_. Go to the storyboard and select the view controller. On the Xcode menu, go to _Editor → Embed In → Navigation Controller_. You will notice now the view has a navigation bar. Inside the view drag a text view and two buttons. Set the necessary constraints. Write __You're on the Home screen. What do you want to do next?__ in the text view. Name one of the buttons __Go to scene 2__ and the other __Go to scene 3__. 

Now copy and paste four times our view controller so we can preserve its elements. To copy an entire view controller, with your mouse or touchpad drag a rectangle around it in the Storyboard so everything is selected, then just press command-c followed by command-v. The new view controller will be pasted on top of the old one, so just drag it aside to see them both. Let's give names to all the scenes in the Attributes Inspector → Title so they will now show as __scene 1, scene 2, scene 3, scene 4, scene 5__. 

Next we need to add _Triggered Segues_ for the two buttons on the home screen. To do this, control + click on the first button in the home scene (_scene 1_) and drag a line from Triggered Segue → action to _scene 2_. Do the same for the second button to _scene 3_. Now go to _scene 2_ and delete the two buttons. In the text area write __You're on the 2nd screen. This is an end of a road.__ Then go to _scene 3_ and write in the text area __You're on the 3rd screen. What do you want to do next?__ Name the two buttons __Go to scene 4__ and __Go to scene 5__. Add _Triggered Segues_ for the two buttons so they will present _scene 4_ and _scene 5_. The text in the text view for these last two scenes will say __You're on the 4th screen. This is an end of a road.__ and __You're on the 5th screen. This is an end of a road.__ respectively. The final storyboard should look like this:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/project2.png "Storyboard")

So why did we do all this, you might be asking now... because this could be the beginning of a storyboard for a complex _adventure game_. As you might have already thought about, the message in each scene could give you a clue, and then present you with two choices that could determine how the game continues/ends. How easy and fun is to make this type of game only with a storyboard, right? There is one more thing we need to add to our scenes to make the game complete, a __Start Over__ button. Since all our scenes are of type _ViewController_ because we cloned them, we can simply add the button programmatically like this:

{% highlight swift %} 
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        self.navigationItem.rightBarButtonItem = UIBarButtonItem(title: "Start Over", style: .Plain, target: self, action: "startOver")
    }
    
    func startOver() {
        if let navigationController = self.navigationController {
            navigationController.popToRootViewControllerAnimated(true)
        }
    }
}
{% endhighlight %}

What we did here was to add a button at the right side end of the navigation bar and make it respond to an action method named __startOver__. This method simply checks if we there is a navigation controller and dismiss the view controllers recursively, one by one, until we reach the root view controller which in our case is the home scene (_scene 1_). To prove that all the view controllers are dismissed we can write a simple debugging method like this:

{% highlight swift %}     
    deinit {
        print("deallocated")
    }
{% endhighlight %}

If you run the project and navigate all the way to _scene 4_ or _scene 5_ and then click the _Start Over_ button, you will notice that __deallocated__ is printed out twice, because the _scene 4/5_ is dismissed first, and then _scene 3_ is dismissed before reaching back to the home scene (_scene 1_). You now have the barebones for creating a great adventure game with lots and lots of scenes. To make the app more interesting, you could add some images beside the existing text. To make it even more interesting you could use a table view on the home scene to present a set of different adventures, instead of just this one.

Until next time!
