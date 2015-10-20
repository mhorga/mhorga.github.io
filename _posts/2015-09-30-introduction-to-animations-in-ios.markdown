---
published: true
title: Introduction to animations in iOS
layout: post
---
I have learned so many great things about UI and animations in iOS from my good friend [@chrisslowik](https://twitter.com/chrisslowik) so I thought I should share my knowledge about that. In the most basic form, animations in iOS could reduce to simply manipulating the colors and size of UI items over a short period of time.

Let's start by creating a Single View Application project in Xcode. In the storyboard drag a `Tap Gesture Recognizer` from the Object Library on top of the view controller. Also add a `View` object, align it to the top left corner of the view controller, and in the Attributes Inspector give it a `blue` background color. In the Size Inspector set a relatively small size for it, like for example `50 x 50` and set both the `X` and `Y` to `20`. Finally, let's create an outlet named __square__ for the view, and also an action named __tapped__ for the gesture recognizer.

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

We called the __animateWithDuration()__ method with a duration of `0.3` seconds during which time the scale of `square` increases 30 times on both X and Y axes. Run the program and notice when you tap on the screen, the square fills the screen with a smooth animation that takes under half a second. That was simple enough, right? 

Let's add some more action to the view. For example, we can implement a second animation for the tap gesture and restore the view to its original size. For this, we need to create a `bool` variable named __zoomed__ for example that can keep track of whether our square is currently zoomed in or not. Let's add it at the top of the class, next to the outlet line:

{% highlight swift %}
var zoomed = false
{% endhighlight %}

Then modify the __animateWithDuration()__ method to look like this:

{% highlight swift %}
if zoomed {
    UIView.animateWithDuration(0.3, delay: 0, options: [UIViewAnimationOptions.BeginFromCurrentState], animations: {
        let scaleTransform = CGAffineTransformIdentity
        self.square.transform = scaleTransform
        self.zoomed = false
        }, completion: nil
    )
} else {
    UIView.animateWithDuration(0.3, delay: 0, options: [UIViewAnimationOptions.BeginFromCurrentState], animations: {
        let scaleTransform = CGAffineTransformMakeScale(30, 30)
        self.square.transform = scaleTransform
        self.zoomed = true
        }, completion: nil
    )
}
{% endhighlight %}

Run the program again and now you will be able to tap once to zoom in, and then tap again to zoom out. How cool is that! But the fun is not over yet. Why don't we also change the color while at it? To make it even more interesting we will create another animation inside our current animation and have it start when the first one ends. For this, simply replace the `nil` value for the `completion` argument, with this closure, in the `if` block:

{% highlight swift %}
{ e in
    UIView.animateWithDuration(0.3, delay: 0, options: [UIViewAnimationOptions.BeginFromCurrentState], animations: {
        self.square.backgroundColor = UIColor.blueColor()
        }, completion: nil
    )
}
{% endhighlight %}

Do the same inside the `else` block but this time set the color to `.yellowColor()`. Run the program again and notice how nicely the color changes, fractions of a second after the scaling finished. You can go on and have even more fun at it by working with multiple views and colors. The complete [source code](https://github.com/Swiftor/AnimationIntro) for this project is available on Github.

Until next time!
