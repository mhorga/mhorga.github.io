---
published: false
title: Completion handlers in iOS
layout: post
---
It is extremely important to understand how asynchronous networking works. We are encountering it in almost all of our projects. There are three major types of asynchronous networking calls in iOS:

1. the `Delegate` pattern
2. the `Notification Center`
3. the `Closure`

In previous posts we already looked at the `delegate` pattern as well as `NSNotificationCenter`, so we are going to take a look at `closures`, which can take many forms, most common being `blocks` or `completion handlers`. To start, create a new project and in `View Controller` let's write a function that geocodes a location, that is, returns the geographical coordinates (`latitude` and `longitude`) for a city we give it as input.

{% highlight swift %}
func geocoding(location: String, completion: () -> ()) {
    CLGeocoder().geocodeAddressString(location, completionHandler: { (placemarks, error) in
        if placemarks?.count > 0 {
            let placemark = placemarks?[0]
            let location = placemark!.location
            let coordinate = location?.coordinate
            self.coordinates = (coordinate!.latitude, coordinate!.longitude)
            print("Inside completion handler: \(self.coordinates)")
            completion()
        }
    })
    print("Outside completion handler: \(self.coordinates)")
}
{% endhighlight %}

Then

{% highlight swift %}
var coordinates = (0.0 , 0.0)

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

What happened here was,

Until next time!