---
published: true
title: Object communication in iOS with Delegation
layout: post
---
In the last two posts about object communication in `iOS`, we looked at `KVO` and the `Notification Center`. In the last part of this series we will look at `Delegation`. The __delegate__ pattern is simple, yet powerful and present everywhere in iOS. Delegation is when a controller defines a __protocol__ that describes what a delegate object must do in order to be allowed to respond to a controller's events. In short, delegation allows one object to act on behalf of, or in conjunction with another object.

The delegating object holds a reference to a delegate and when it needs to, the delegating object sends a message to its delegate that informs it of an occurring event. The delegate then responds to the message accordingly by returning specific data, or by performing an update of itself or another object. This direct one-to-one connection between both objects can lead to custom object communication that doesn't require a third party to manage.

In `KVO` and `Notification Center` (both observer patterns), an observer sees a particular event and then does something with it. With `delegation`, a delegate handles a particular event and takes ownership of the completion handler. When we want to have complete control over how and when communication between two objects should occur, delegation should be used. When architecting a project, the possibility of using `delegation` should be explored first before considering `KVO` and `Notification Center`.

Let’s create a new `Single View Application` project and then create a new class named __Model__ that will implement a `protocol` and will have a __delegate__ property:

{% highlight swift %}
protocol ModelDelegate {
    func respond()
}

class Model: NSObject {    
    var delegate: ModelDelegate?
    
    func delegateResponded() {
        print("Delegate said: ")
        delegate?.respond()
    }
}
{% endhighlight %}

We also created a method that we will call from `ViewController` as a delegate and which calls the protocol method __respond()__ which as you can see it is not implemented here yet. Then in `ViewController`, we need to conform to the __ModelDelegate__ protocol, and tell it that this class will be its `delegate`. You will now have to implement the `protocol` required method:

{% highlight swift %}
class ViewController: UIViewController, ModelDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
        var model = Model()
        model.delegate = self
        model.delegateResponded()
    }
    
    func respond() {
        println("Hello, Protocol!")
    }
}
{% endhighlight %}

Run the app and you should see this output:

{% highlight swift %}
Delegate said: Hello, Protocol!
{% endhighlight %}

To prove that it is not working without being a `delegate` of the `Model` class, comment out that line: 

{% highlight swift %}
//model.delegate = self
{% endhighlight %}

If you run the app again, you will notice that the output changes to just:

{% highlight swift %}
Delegate said:
{% endhighlight %}

This happened because `Model` does not know who its delegate is anymore, so it cannot find a method named __respond()__ implemented anywhere. The [source code](https://github.com/mhorga/Delegation) is available on Github.

Until next time!