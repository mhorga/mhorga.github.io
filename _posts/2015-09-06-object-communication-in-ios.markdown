---
published: true
title: Object communication in iOS with KVO
layout: post
---
There are three common ways objects and classes can communicate in iOS: `Key-Value Observation`, `Notification Center` and `Delegation`. In this first part we will only look at the first pattern.

__Key-Value Observation (KVO)__ is an `Observer pattern` which provides a way for objects to know when properties are changed, and they can do so by subscribing to be notified of these changes when they occur. Any object that inherits from `NSObject` can use this resource at no extra cost. `KVO` is handy when at most a handful of objects need to be observed, because there is only a single method available for observation which could easily become stuffed with too many string-based key path condition matchings we need to make inside of this method.

We will use a simple bank account example to illustrate the `KVO` concept. Let's start by creating a new `Single View Application`. In the storyboard add a `UILabel`, a `UITextField` and a `UIButton`. In the `ViewController` class create an `IBOutlet` named __amountTextField__ and another one named __currentBalanceLabel__, and an `UIAction` named __submitAction()__ for the button. We are done with the storyboard so let's get to the coding part.

First, create a new class named __Account__. We just create two properties, one a constant and another one a variable to keep track of our account balance. Then we create an __init()__ method where we initialize the current balance to be the starting balance. Finally, we create an __update()__ method that we can use to modify our current balance based on an amount the user would input:

{% highlight swift %}
let startingBalance = 100.0
var currentBalance = 0.0

override init() {
    super.init()
    currentBalance = startingBalance
}

func update(amount: Double) {
    currentBalance += amount
}
{% endhighlight %}

So far so good. This class works just fine as is, except other classes are not able to communicate to it just yet. We will implement a notification system so it can communicate with other classes when changes occur. __Step 1__ is to create a key path that can be recognized by the communicating class: 

{% highlight swift %}
let currentBalanceKeyPath = "currentBalance"
{% endhighlight %}

Then, at the end of the `update()` method set the value of our observed property for this key path:

{% highlight swift %}
setValue(currentBalance, forKeyPath: currentBalanceKeyPath)
{% endhighlight %}

The remaining three steps are in the `ViewController` class. Let's create the same key path for consistency and also create an `Account` object:

{% highlight swift %}
let currentBalanceKeyPath = "currentBalance"
var account = Account()
{% endhighlight %}

In __viewDidLoad()__ update the label text with the current balance:

{% highlight swift %}
currentBalanceLabel.text = "Current balance: \(account.currentBalance)"
{% endhighlight %}

We are now ready for __Step 2__: register an observer for our key path property. We do this still inside the `viewDidLoad()` method:

{% highlight swift %}
account.addObserver(self, forKeyPath: currentBalanceKeyPath, options: NSKeyValueObservingOptions.Old | NSKeyValueObservingOptions.New, context: nil)
{% endhighlight %}

In __Step 3__ we implement an observing method so that observer gets notification when changes occur:

{% highlight swift %}
override func observeValueForKeyPath(keyPath: String, ofObject object: AnyObject, change: [NSObject : AnyObject], context: UnsafeMutablePointer<Void>) {
    if keyPath == currentBalanceKeyPath {
        if (account.currentBalance < 0) {
            currentBalanceLabel.textColor = UIColor.redColor()
        } else {
            currentBalanceLabel.textColor = UIColor.blackColor()
        }
        currentBalanceLabel.text = "Current balance: \(account.currentBalance)"
    }
}
{% endhighlight %}

All that matters inside this method is that we check whether the method's `keyPath` argument equals our own `currentBalanceKeyPath` and if it does, then we proceed with the work we need to do, which in our case is just use different colors for the label text, depending on when the amount is negative or positive. In the last step, __Step 4__, we remove the observer to make sure we don't see any unexpected behavior later on:

{% highlight swift %}
deinit {
    account.removeObserver(self, forKeyPath: currentBalanceKeyPath)
}
{% endhighlight %}

The only other thing we need to do before running the program is to configure the `IBAction` method we created in the beginning, so that the button can send a new amount to the `update()` method:

{% highlight swift %}
@IBAction func submitAction(sender: UIButton) {
    let amount = (amountTextField.text as NSString).doubleValue
    account.update(amount)
    amountTextField.text = nil
}
{% endhighlight %}

Now run the program and notice how various amounts are added/subtracted from the current balance. The label color also reflects whether we have a negative or a positive balance.

Until next time!
