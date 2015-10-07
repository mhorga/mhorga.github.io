---
published: false
title: Introduction to animations in iOS
layout: post
---
I have learned so many great things about UI and animations in iOS from my good friend [@chrisslowik](https://twitter.com/chrisslowik) so I thought I should share my knowledge about that. In the most basic form, animations in iOS could reduce to simply manipulating the colors and size of UI items over a short period of time.

Let's start by creating a Single View Application project in Xcode. In the storyboard drag a `Tap Gesture Recognizer` from the Object Library on top of the view controller. Aldo add a `View` object, align it to the top left corner of the view controller, and in the Attributes Inspector give it a `blue` background color. In the Size Inspector set a relatively small size for it, like for example `50 x 50` and set both the `X` and `Y` to `20`. Finally, let's create an outlet named __square__ for the view, and also an action named __tapped__ for the gesture recognizer.

Next, let's write some code! In the view controller class, inside __tapped(:)__ add these lines:

{% highlight swift %}
@IBAction func tapped(sender: UITapGestureRecognizer) {
    UIView.animateWithDuration(0.3, delay: 0, options: [UIViewAnimationOptions.BeginFromCurrentState], animations: {
        let scaleTransform = CGAffineTransformMakeScale(30, 30)
        self.square.transform = scaleTransform
        }, completion: nil
    )
}
{% endhighlight %}

We called the __animateWithDuration()__ method with a duration of `0.3` seconds during which time the scale of `square` increases 30 times on both X and Y axes. Simple enough, right? Let's add some more action to the view. For example, we can implement a second animation for the tap gesture and restore the view to its original size. For this, we need to create a `bool` variable named __zoomed__ for example that can keep track of whether our square is currently zoomed in or not. Let's add it at the top of the class, next to the outlet line:

{% highlight swift %}
var zoomed = false
{% endhighlight %}

{% highlight swift %}
{% endhighlight %}

Until next time!