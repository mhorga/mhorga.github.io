---
published: false
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

    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
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

Next let's use protocols in three different ways. First, create a Swift file named ZipCodeDelegate:

{% highlight swift %} 

class ZipCodeDelegate: NSObject, UITextFieldDelegate {

    override init() {
        super.init()
    }
} 

{% endhighlight %}

Since this class conforms to the UITextFieldDelegate protocol we can also use ZipCodeDelegate type as protocol. Now create a second Swift file named CashDelegate:

{% highlight swift %} 

class Cash: NSObject {
    
    var delegate: CashDelegate?
    
    override init() {
        super.init()
    }
}

protocol CashDelegate: UITextFieldDelegate {
    
}

{% endhighlight %}

Notice that this time the Cash class does not conform to UITextFieldDelegate anymore. Instead we created a protocol named CashDelegate that conforms to UITextFieldDelegate. Inside the Cash class we declare a delegate variable of type CashDelegate so we can use it later in other classes that want to conform to the CashDelegate protocol.

Let's head back to the view controller and declare a couple of class properties for the delegates:

{% highlight swift %} 

    // Delegate objects
    let zipCodeDelegate = ZipCodeDelegate()
    let cashDelegate = Cash().delegate

{% endhighlight %}

In the _viewDidLoad()_ method set the delegates:

{% highlight swift %} 

    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.textField1.delegate = self
        self.textField2.delegate = zipCodeDelegate
        self.textField3.delegate = cashDelegate
    }

{% endhighlight %}

Notice that the first text field's delegate is set to __self__.

Until next time!