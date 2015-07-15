---
published: false
title: Protocols and Delegates
layout: post
---
A good friend and great mentor, [@getaaron](https://twitter.com/getaaron), taught me that when you write code in which two objects communicate, you could run into the risk of increasing __coupling__ (how much two classes need to know about each other to work) and reducing __cohesion__ (how closely related the code in one class is). Good code _minimizes coupling_ and _maximizes cohesion_. One way to achieve this is through _delegation using protocols_.

A __protocol__ defines requirements (properties and methods) that suit custom functionality. The protocol only describes what an implementation will look like. A class, structure or enumeration could then adopt the protocol to provide an implementation of those requirements. A type that satisfies the requirements of a protocol is said to __conform__ to that protocol. Use the _protocol_ keyword to declare a protocol. 

The { get } following the simpleDescription property indicates that it is read-only, meaning that the value of the property can be viewed, but never changed.

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

Until next time!