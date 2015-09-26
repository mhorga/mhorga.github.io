---
published: true
title: Completion handlers in iOS
layout: post
---
It is extremely important to understand how asynchronous networking works. We are encountering it in almost all of our projects. There are three major types of asynchronous networking calls in iOS:

1. the `Delegate` pattern
2. the `Notification Center`
3. the `Closure` type

In previous posts we already looked at the `delegate` pattern as well as `NSNotificationCenter`, so we are going to take a look at `closures`, which can take many forms, most common being `blocks` or `completion handlers`. To start, create a new project and in `View Controller` let's write a function that geocodes a location, that is, returns the geographical coordinates (`latitude` and `longitude`) for a city we give it as input.

{% highlight swift %}
var coordinates = (0.0 , 0.0)

func geocoding(location: String, completion: () -> ()) {
    CLGeocoder().geocodeAddressString(location) { (placemarks, error) in
        if placemarks?.count > 0 {
            let placemark = placemarks?[0]
            let location = placemark!.location
            let coordinate = location?.coordinate
            self.coordinates = (coordinate!.latitude, coordinate!.longitude)
            print("Inside completion handler: \(self.coordinates)")
            completion()
        }
    }
    print("Outside completion handler: \(self.coordinates)")
}
{% endhighlight %}

The `Core Location` framework provides the `CLGeocoder` class whose method named `geocodeAddressString()` does the forward geocoding for us. It requires that we provide a string and a completion handler as input parameters. But since the completion handler is the last argument of the function, the compiler lets us modify the function signature, from having the completion handler inside the parentheses:

{% highlight swift %}
CLGeocoder().geocodeAddressString(location, completionHandler: { ... })
{% endhighlight %}

to taking it outside of the parentheses, and even omit its name altogether:

{% highlight swift %}
CLGeocoder().geocodeAddressString(location) { ... }
{% endhighlight %}

Now let's call the fuction inside `viewDidLoad()` and see the different behaviors:

{% highlight swift %}
override func viewDidLoad() {
    print("At load time: \(coordinates)")
    geocoding("New York, NY") {
        print("After handler completes: \(self.coordinates)")
    }
}
{% endhighlight %}

If you build and run the project you should see the following output:

{% highlight swift %}
At load time: (0.0, 0.0)
Outside completion handler: (0.0, 0.0)
Inside completion handler: (40.713054, -74.007228)
After handler completes: (40.713054, -74.007228)
{% endhighlight %}

So first we print the coordinates before even calling the geocoding function and the result is `(0.0, 0.0)` as expected, because we have not yet changed our global variable. Then the fun begins! Once we called the function, the code that finished first was the print statement outside the completion handler block, because the code inside the block was still running, so obviously the result is still `(0.0, 0.0)` as the block has not yet updated the global variable. However, we notice that inside the block, the result is correct. What we had to do then was to pass the result to the caller line inside `viewDidLoad()` and we achieved that by calling the completion handler which only has a print statement, as an argument for the `geocoding()` call. To break this call, just comment out the `completion()` line inside the `geocoding()` function and see the difference.

Until next time!
