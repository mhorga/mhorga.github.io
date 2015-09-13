---
published: true
title: Object communication in iOS with Notification Center
layout: post
---
`Notification Center` is an `Observer` pattern, just like `KVO` we looked at last time. The __NSNotificationCenter__ singleton allows us to broadcast information using an object called __NSNotification__. Observers will register with `NSNotificationCenter` and they will respond to certain notification events with a specific action. Whenever an `NSNotification` is posted, `Notification Center` goes through it's dispatch table and notifies any registered observer of that particular notification. There could be multiple observers for a single notification, all responding in their own unique ways. 

`Notification Center` should be used when you want to broadcast throughout your entire program that there are app-wide changes. This enables to you respond across multiple classes who are observing for the same notification. The biggest difference between `KVO` and `Notification Center` is that `KVO` tracks specific changes to an object, while `Notification Center` is used to track generic events, like when a button is pressed to post an action. And while `KVO` will provide you with information that's changed, `Notification Center` only tells you that the event occurred. It's up to you on how you want to respond to that event. Now you can pass extra data along with `NSNotification`, but you have to manually set that, versus it coming automatically with `KVO`.

You can access a singleton using `NSNotificationCenter.defaultCenter()`. To register with `Notification Center`, you need to call the `addObserver(observer:selector:name:object:)` method. Just like with `KVO`, if it is not required for you to continue observing with `Notification Center`, you'll want to unregister your observers using the `removeObserver(observer:name:object:)` method. It is more likely to receive unintended actions due to leaving observers registered with `Notification Center` in comparison to `KVO`. This is because `Notification Center` allows for duplicate observer registration, meaning that you can register to observe the same `NSNotification` more than once. It's common to register for notification observers in `viewDidLoad` and unregister them in `viewDidDisappear` and re-register them as your view comes back onto the stack. 

We're going to use `Notification Center` to have observers in both the source class of the notification, the __ViewController__ class, and a second class we’re going to name __Destination__. They will respond to the same event. We will post a `NSNotification` that will go through `NSNotificationCenter`'s dispatch table, and it'll inform the observers so that they can perform their individual actions. Let’s create a new `Single View Application` project. In the storyboard add a button with a connected `IBAction` named __submitAction__, and a label with a connected `IBOutlet` named __successLabel__. In `ViewController` let’s create a helper method for our new label:

{% highlight swift %}
func showSuccessLabel() {
    successLabel.text = "I got notification in source class."
    successLabel.alpha = 1.0
    UIView.animateWithDuration(2.0, animations: { () -> Void in
        self.successLabel.alpha = 0.0
    })
}
{% endhighlight %}

This method will simply show a message with a 2-second animation that consists in gradually changing the `alpha` for the label text, thus creating the effect of going from fully visible to completely invisible. Next we need a method that will be called when a notification is received:

{% highlight swift %}
func didReceiveSubmitNotification(notification: NSNotification) {
    showSuccessLabel()
}
{% endhighlight %}

Next, we need to create a class constant for the notification name:

{% highlight swift %}
let submitNotification = "submitNotification"
{% endhighlight %}

Now we can register an observer in __viewDidLoad()__:

{% highlight swift %}
NSNotificationCenter.defaultCenter().addObserver(self, selector: "didReceiveSubmitNotification:”, name: submitNotification, object: nil)
{% endhighlight %}

Finally, we need to fire the notification when the button is pressed, inside the __submitAction()__ method:

{% highlight swift %}
NSNotificationCenter.defaultCenter().postNotificationName(submitNotification, object: nil)
{% endhighlight %}

If you run the app now you will notice that the notification fires every time we make a transaction and the message is displayed with animation. What we can do now is to add an action in the `Destination` class as well and make it fire when the same notification is received. For this, in the `Destination` class create the same constant for the notification name:

{% highlight swift %}
let submitNotification = "submitNotification"
{% endhighlight %}

Also add a method that will be called when a notification is received:

{% highlight swift %}
func update() {
    print("I got notification in destination class.")
}

func didReceiveSubmitNotification(notification: NSNotification) {
    update()
}
{% endhighlight %}

Then we need to register an observer inside the __init()__ method:

{% highlight swift %}
override init() {
    super.init()
    NSNotificationCenter.defaultCenter().addObserver(self, selector: "didReceiveSubmitNotification:", name: submitNotification, object: nil)
}
{% endhighlight %}

Now all we need to do is create a `Destination` object in `ViewController`, and this will register an observer for the `Destination` class as well since was added in its __init()__ method:

{% highlight swift %}
let destination = Destination()
{% endhighlight %}

Don't forget to remove the observers once they fired, so we don't see unexpected behaviors. Run the project and notice that firing the notification when the button is pressed will send the message to both classes. Stay tuned for the last part of these series, next week.

Until next time!
