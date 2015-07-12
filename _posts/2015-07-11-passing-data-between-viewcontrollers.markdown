---
published: false
title: Passing data between View Controllers
layout: post
---
When preparing the incoming view controller, we need to first decide how do we want to configure our data passing:

- using only an __action__
- using an __action__ and a __segue__
- using only a __segue__

Assume we have a __FirstViewController__ and we would like it to be able to pass data to a __SecondViewController__. The first view of the _storyboard_ (connected to FirstViewController) has three _buttons_ (created as __@IBOutlet__) of which the first two are connected to an _action_ method named __sendData()__. This means, when we press any of these two buttons, this __@IBAction__ method will execute. The SecondViewController has a property named __receivedData__ (initialized to 2 - we will understand later why) and the second view has a label created as an _outlet_ that we could use to display the passed data, and a button connected to an action method __goBack()__ to go back to the first screen. Here are the two view controllers:

{% highlight swift %} 
class FirstViewController: UIViewController {
    @IBOutlet weak var buttonOne: UIButton!
    @IBOutlet weak var buttonTwo: UIButton!
    @IBOutlet weak var buttonThree: UIButton!
    
    @IBAction func sendData(sender: UIButton) {
    }
}

class SecondViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    var receivedData = 2

    override func viewDidLoad() {
        super.viewDidLoad()
        label.text = receivedData
    }

    @IBAction func goBack(sender: UIButton) {
        self.dismissViewControllerAnimated(true, completion: nil)
    }
}
{% endhighlight %}

Now let's focus on the first button. We want to send some data to the SecondViewController when it's pressed. For this we need to first give the second view an identifier named _SecondViewController_ (or some other name of your choice) under the _Identity Inspector_ in the storyboard. Now we can instantiate the second view by its identifier like this:

{% highlight swift %} 
    @IBAction func sendData(sender: UIButton) {
        if sender == buttonOne {
            let controller = self.storyboard?.instantiateViewControllerWithIdentifier("SecondViewController") as! SecondViewController
            controller.receivedData = 1
            self.presentViewController(controller, animated: true, completion: nil)
        }
    }
{% endhighlight %}

For the second button we need to create a segue named __ButtonTwoSegue__ that connects the first view to the second view. Now, inside the _sendData()_ method right after the _if_ statement append an __else__ statement where we handle the action for our second button:

{% highlight swift %} 
    @IBAction func sendData(sender: UIButton) {
        if sender == buttonOne {
            let controller = self.storyboard?.instantiateViewControllerWithIdentifier("SecondViewController") as! SecondViewController
            controller.receivedData = 1
            self.presentViewController(controller, animated: true, completion: nil)
        } 
        else if sender == buttonTwo {
            // notice that we do not change receivedData here - we leave it to its default value of 2
            performSegueWithIdentifier("ButtonTwoSegue", sender: self)
        } 
        // else, do nothing else anymore - for the third button we create another method
    }
{% endhighlight %}

For the third button we need to create another segue named __ButtonThreeSegue__ that connects buttonThree this time (not the first view) to the second view. Now, we override the method _prepareForSegue()_ method right after the _if_ statement append an __else__ statement where we handle the action for our second button:

{% highlight swift %} 
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if segue.identifier == "ButtonThreeSegue" {
            let controller = segue.destinationViewController as! ResultViewController
            controller.receivedData = 3
        }
    }
{% endhighlight %}

At this point you can run your project and try each of the three buttons to see how the label has a _receivedData_ value of 1, 2 or 3 depending on what button was pressed.

Until next time!