---
published: false
title: IBDesignable and IBInspectable
layout: post
---
Last time we looked at two of the features of the `Interface Builder`: the `Auto Layout` and the `Stack Views`. Today we will look at one last _trick_, the __@IBDesignable__ and __@IBInspectable__ attributes. Let's start by creating a new project in `Xcode`. We would like to have a custom view that consists of an image and some sort of subtitle for it. We are interested in seeing how they look at design time, before we even build and run the project. But stay with me here, you're so close to finding out how to do that!

Let's create a new class and name it __CustomView.swift__. Inside, let's create a `UIImageView` and a `UILabel` and then add them to our view:

{% highlight swift %}
class CustomView : UIView {
    
    var label: UILabel!
    var imageView: UIImageView!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        addSubviews()
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        addSubviews()
    }
    
    func addSubviews() {
        imageView = UIImageView()
        addSubview(imageView)
        label = UILabel()
        addSubview(label)
    }
}
{% endhighlight %}

Let's also add the __layoutSubviews()__ method to configure the image and label: 

{% highlight swift %}
override func layoutSubviews() {
    super.layoutSubviews()
    layer.borderWidth = 0.5
    layer.cornerRadius = 5
    imageView.frame = self.bounds
    imageView.contentMode = UIViewContentMode.ScaleAspectFit
    label.frame = CGRectMake(0, bounds.size.height - 40, bounds.size.width, 40)
    label.backgroundColor = UIColor.blackColor().colorWithAlphaComponent(0.1)
    label.textAlignment = .Center
    label.textColor = .grayColor()
}
{% endhighlight %}

In the storyboard let's add a `UIView` and in `Identity Inspector` change its _class_ to __CustomView__, and finally set constraints for it, like this:

![alt text](https://github.com/Swiftor/IBDesignable-and-IBInspectable/raw/master/images/ib1.png "IB1")

Both the imageview and the label are displayed now, but they are both empty! So here comes the magic. Modify the class signature to include the __@IBDesignable__ attribute and add the following method:

{% highlight swift %}
@IBDesignable class CustomView : UIView {
    
    override func prepareForInterfaceBuilder() {
        super.prepareForInterfaceBuilder()
        label.text = " iPhone 6s Plus"
        let url = "http://i.telegraph.co.uk/multimedia/archive/03058/iphone_6_3058505b.jpg"
        imageView.image = UIImage(data: NSData(contentsOfURL: NSURL(string: url)!)!)
    }
...
{% endhighlight %}

Now go to the storyboard and, voila! There is an image and text underneath. And all this was accomplished by using the `IBDesignable` attribute. There is a catch about the __prepareForInterfaceBuilder()__ method though. Although it’s compiled for runtime, code called from `prepareForInterfaceBuilder` never gets called except by the `Interface Builder` at design time, so if you run the app nothing will displayed at this time. Go ahead and see for yourself.



{% highlight swift %}

{% endhighlight %}

Until next time!