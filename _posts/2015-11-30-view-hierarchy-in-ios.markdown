---
published: true
title: View hierarchy in iOS
layout: post
---
This week we will look into how view hierarchy works in `iOS`. Specifically, we will see views that are not necessarily part of a view controller as we are used to see. In `Xcode` create a new `Single View Application` project. Go to the storyboard and add a `UIView` directly to the `View Controller Scene`, like in this screenshot:

![alt text](https://github.com/Swiftor/ViewHierarchy/raw/master/images/1.png "1")

You will notice that the view was added outside of the main scene. It is still a part of the scene, just not a part of the view controller. Let’s add a Horizontal Stack View to the view we just created. Set all constraints to `0` and also set it to `Fill equally`. From `Resolve Auto Layout Issues` choose `Update Frames` so our stack view fills the view. Next, drop two buttons in the stack view and name them __Yes__ and __No__, respectively. Add one more button in the view controller, right in the middle of it, add the missing constraints and name it __Are you ready?__ or anything else you wish. Your storyboard should now look like this:

![alt text](https://github.com/Swiftor/ViewHierarchy/raw/master/images/2.png "2")

Next, let’s add an `IBOutlet` named __secondView__ for the view and an `IBAction` named __buttonPressed__ for the central button. Inside this method let’s add the subview to our view:

{% highlight swift %}
@IBAction func buttonPressed(sender: UIButton) {
    view.addSubview(secondView)
}
{% endhighlight %}

Run the app and notice that the two buttons from the second view are showing, but most likely that is not the place where we want them:

![alt text](https://github.com/Swiftor/ViewHierarchy/raw/master/images/3.png "3")

Let’s also create an `IBOutlet` named __button__ for the central button so we can refer to it next when building constraints. What we would like to do is have the second view appear on top of the central button once we tap it. Add the following lines to our action method:

{% highlight swift %}
@IBAction func buttonPressed(sender: UIButton) {
    if sender.selected {
        secondView.removeFromSuperview()
        sender.selected = false
    } else {
        view.addSubview(secondView)
        secondView.translatesAutoresizingMaskIntoConstraints = false
        let bottomConstraint = secondView.bottomAnchor.constraintEqualToAnchor(button.topAnchor)
        let leftConstraint = secondView.leftAnchor.constraintEqualToAnchor(button.leftAnchor)
        let rightConstraint = secondView.rightAnchor.constraintEqualToAnchor(button.rightAnchor)
        let heightConstraint = secondView.heightAnchor.constraintEqualToConstant(44)
        NSLayoutConstraint.activateConstraints([bottomConstraint, leftConstraint, rightConstraint, heightConstraint])
        sender.selected = true
    }
}
{% endhighlight %}

We check if the state of the central button is selected or not and if it’s not, we anchor the second view’s bottom to the button’s top and matched its left, right and height properties. If you run the app now, you will see the second view shows like a contextual menu when we tap our central button:

![alt text](https://github.com/Swiftor/ViewHierarchy/raw/master/images/4.png "4")

Until next time!