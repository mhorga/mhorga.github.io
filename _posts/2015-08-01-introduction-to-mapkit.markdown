---
published: true
title: Introduction to MapKit
layout: post
---
This time, we will look into how maps, locations and annotations work in iOS. Start by creating a new _Single View Application_ project in _Xcode_. Go to the storyboard and drag a _Map Kit View_ from the _Object Library_ on top of the view controller. Make it fit the entire view and set necessary constraints. There is one more thing we need to do in the storyboard, create an outlet named __mapView__ for our _MKMapView_ object, and connect it to the view controller:

{% highlight swift %}
@IBOutlet weak var mapView: MKMapView!
{% endhighlight %}

In __viewDidLoad()__ let's create an array of dictionaries and hardcode a couple of locations into it:

{% highlight swift %}
let locations = [
    ["name" : "Apple Inc.",
    "latitude" : 37.33187,
    "longitude" : -122.02951,
    "mediaURL" : "http://www.apple.com"],
    ["name" : "BJ's Restaurant & Brewhouse",
    "latitude" : 37.33131,
    "longitude" : -122.03175,
    "mediaURL" : "http://www.bjsrestaurants.com"]
]
{% endhighlight %}

Next, we will create a _MKPointAnnotation_ object for each dictionary in the __locations__ array, and later we will store the object in an array:

{% highlight swift %}
var annotations = [MKPointAnnotation]()
{% endhighlight %}

Now let's iterate through our locations, populate the annotations with data, and store them in the __annotations__ array:

{% highlight swift %}
for dictionary in locations {
    let latitude = CLLocationDegrees(dictionary["latitude"] as! Double)
    let longitude = CLLocationDegrees(dictionary["longitude"] as! Double)
    let coordinate = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    let name = dictionary["name"] as! String
    let mediaURL = dictionary["mediaURL"] as! String
    let annotation = MKPointAnnotation()
    annotation.coordinate = coordinate
    annotation.title = "\(name)"
    annotation.subtitle = mediaURL
    annotations.append(annotation)
}
{% endhighlight %}

Next we add the annotations to the map view:

{% highlight swift %}
mapView.addAnnotations(annotations)
{% endhighlight %}

If you run the app now, you will notice that the map loads but it shows our current location and the annotations we set are not visible (unless you are located in Apple's headquarters). What we need to do is center our map on one of our annotations. For this to work, let's create the following method:

{% highlight swift %}
func centerMapOnLocation(location: MKPointAnnotation, regionRadius: Double) {
    let coordinateRegion = MKCoordinateRegionMakeWithDistance(location.coordinate,
        regionRadius * 2.0, regionRadius * 2.0)
    mapView.setRegion(coordinateRegion, animated: true)
}
{% endhighlight %}

This method takes in as arguments a location (annotation) and a radius in meters of the region we want to see in the map view. Next, we call this method in __viewDidLoad()__ right at the end of it, and we center on the Apple's headquarters, while setting a radius of one kilometer:

{% highlight swift %}
centerMapOnLocation(annotations[0], regionRadius: 1000.0)
{% endhighlight %}

Now if you run the app again you can finally see the two annotation pins, and you can click them to see the name and URL we set for each. There is one problem with the annotations, however. Probably you noticed you cannot load the URL in a browser. In order for this to work we need to conform to the __MKMapViewDelegate__ protocol:

{% highlight swift %}
class ViewController: UIViewController, MKMapViewDelegate {
{% endhighlight %}

Next we implement two of the delegate methods. First method configures the annotation with a small __info__ button on the right side of each pin:

{% highlight swift %}
func mapView(mapView: MKMapView, viewForAnnotation annotation: MKAnnotation) -> MKAnnotationView? {
    let reuseIdentifier = "pin"
    var pin = mapView.dequeueReusableAnnotationViewWithIdentifier(reuseIdentifier) as? MKPinAnnotationView
    if pin == nil {
        pin = MKPinAnnotationView(annotation: annotation, reuseIdentifier: reuseIdentifier) 
        pin!.pinColor = .Red
        pin!.canShowCallout = true
        pin!.rightCalloutAccessoryView = UIButton(type: .DetailDisclosure)
    } else {
        pin!.annotation = annotation
    }
    return pin
}
{% endhighlight %}

Notice the similarities between this method and the _cellForRowAtIndexPath_ method in a table view. The second method responds when the _info_ button is tapped:

{% highlight swift %}
func mapView(mapView: MKMapView, annotationView: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
    if control == annotationView.rightCalloutAccessoryView {
        let app = UIApplication.sharedApplication()
        app.openURL(NSURL(string: (annotationView.annotation!.subtitle!)!)!)
    }
}
{% endhighlight %}

There is one more thing we need to do before we can call these two methods. Our view controller needs to be set as the `MapKit` delegate. Add this line at the top of `ViewDidLoad`:

{% highlight swift %}
mapView.delegate = self
{% endhighlight %}

Now if you run the app and tap on the _info_ button in any annotation, the URL should load in a browser.

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator3.png "MapKit")

There are a few interesting features that could be implemented further on. Notice that the locations we hardcoded in the array are similar to the _JSON_ data that you can download from _Parse_. It would be great if we can get them from the cloud instead of hardcoding them. Another useful feature would be to implement _geocoding_ both forward and reverse, so you can either input the latitude and longitude to find an address, or the opposite, input an address to find the latitude and longitude. You could also implement searching and displaying of nearby __POI__ (points of interest). The possibilities are unlimited.

Until next time!