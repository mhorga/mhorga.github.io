---
published: true
title: Protocols and Delegates
layout: post
---
A good friend and great mentor, [@getaaron](https://twitter.com/getaaron), taught me that when you write code in which two objects communicate, you could run into the risk of increasing __coupling__ (how much two classes need to know about each other to work) and reducing __cohesion__ (how closely related the code in one class is). Good code _minimizes coupling_ and _maximizes cohesion_. One way to achieve this is through _delegation using protocols_.

A __protocol__ defines requirements (properties and methods) that suit specific task or functionality. The protocol only describes what an implementation will look like. A class, structure or enumeration could then adopt the protocol to provide an implementation of those requirements. A type that satisfies the requirements of a protocol is said to __conform__ to that protocol.

The UITextField class is the most common protocol we could refer to. UITextField knows nothing about view controllers. The view controller only knows a minimal amount about the UITextField class (e.g. it doesn't know how to do spell-check). This is good because despite the lack of knowledge about each other, the two classes are able to communicate perfectly. They have _low coupling_.

We start with an empty view controller template like this:

{% highlight swift %} 

import UIKit

class ViewController: UIViewController {

    // Outlets
    
    // Delegate objects
    
    // Life Cycle Methods
    
    // Delegate Methods
}

{% endhighlight %}

In the storyboard, create three text field objects. In the view controller create outlets for all the text fields: 

{% highlight swift %} 

    // Outlets
    @IBOutlet weak var textField1: UITextField!
    @IBOutlet weak var textField2: UITextField!
    @IBOutlet weak var textField3: UITextField!    

{% endhighlight %}

Next let's use protocols in three different ways for each of the three text fields. First, create a Swift file named FirstDelegate:

{% highlight swift %} 

import UIKit

class FirstDelegate: NSObject, UITextFieldDelegate {
    override init() {
        super.init()
    }
    
    func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String) -> Bool {
        textField.textColor = UIColor.greenColor()
        return true
    }
} 

{% endhighlight %}

Since this class conforms to the UITextFieldDelegate protocol we can also use FirstDelegate type as protocol, so we implemented its method named _textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String)_ and the only thing this method will do is change the text field's text color to _green_. Now create a second Swift file named SecondDelegate:

{% highlight swift %} 

import UIKit

class Second: NSObject {
    override init() {
        super.init()
    }
}

protocol SecondDelegate {
    func protocolMethod(textField: UITextField)
}

{% endhighlight %}

Notice that this time the Second class does not conform to UITextFieldDelegate anymore. Instead we created a protocol named SecondDelegate so we can use it later in other classes that want to conform to the SecondDelegate protocol. Inside this protocol we declare a new method _protocolMethod()_ which conforming classes will need to implement. Let's head back to the view controller and declare a class property for the FirstDelegate protocol:

{% highlight swift %} 

    // Delegate objects
    let firstDelegate = FirstDelegate()

{% endhighlight %}

In the _viewDidLoad()_ method set the delegates:

{% highlight swift %} 

    // Life Cycle Methods
    override func viewDidLoad() {
        super.viewDidLoad()
        textField1.delegate = self
        textField2.delegate = firstDelegate
        protocolMethod(textField3)
    }

{% endhighlight %}

Notice that the first text field's delegate is set to __self__. This means the view controller itself is the delegate in this case, and later it will implement methods from the protocol it conforms to. We want our view controller to conform to the other two protocols we need, so change the class signature accordingly:

{% highlight swift %} 

class ViewController: UIViewController, UITextFieldDelegate, SecondDelegate {

{% endhighlight %}

You will notice we now see an error:

`Type 'ViewController' does not conform to protocol 'SecondDelegate'`

To fix this, we need to implement the delegate methods for the protocols we conformed to:

{% highlight swift %} 

    // Delegate Methods
    func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String) -> Bool {
        textField.textColor = UIColor.redColor()
        return true
    }
    
    func protocolMethod(textField: UITextField) {
        textField.textColor = UIColor.blueColor()
    }

{% endhighlight %}

Now run the program and try typing text in each of the three text fields. You will notice that the first one is red, the second one is green and the third one is blue. To recap, we conformed to _UITextFieldDelegate_ to color the text, then we used a delegate for the _FirstDelegate_ class, and finally we conformed to SecondDelegate and implemented a new method for this protocol.

Until next time!