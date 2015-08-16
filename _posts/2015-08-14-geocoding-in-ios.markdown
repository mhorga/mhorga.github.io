---
published: true
title: Geocoding in iOS
layout: post
---
Not so long ago I wrote an [Introduction to MapKit](http://mhorga.org/2015/08/01/introduction-to-mapkit.html) and at the end of that article I was suggesting, as an improvement, the ability to obtain real location data from the web instead of hard-cording locations. In this article I want to introduce you to __geocoding__ as a way to obtain information about geographical locations. Geocoding can be __forward__, when we obtain the geographical coordinates (_latitude_ and _longitude_) from other location data such as postal addresses, or __reverse__, when we obtain the address of a location by having the latitude and longitude as inputs. 

In this article, I want to show how to do geocoding using both _Apple Maps_ and _Google Maps_. The major difference between the two is that for Apple Maps you do not need to access an external URL to get location data, while for Google Maps you do. I will start with Google Maps geocoding, so let's create a new Xcode project first. In _ViewController_ add two properties, one to hold the base URL for the Google Maps API, and a second one for your API key.

{% highlight swift %}
let baseUrl = "https://maps.googleapis.com/maps/api/geocode/json?"
let apikey = "YOUR_API_KEY"
{% endhighlight %}

Next, let's see how _forward geocoding_ is done using Google Maps. For that let's create a new method:

{% highlight swift %}
func getLatLngForZip(zipCode: String) {
    let url = NSURL(string: "\(baseUrl)address=\(zipCode)&key=\(apikey)")
    let data = NSData(contentsOfURL: url!)
    let json = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
    if let result = json["results"] as? NSArray {
        if let geometry = result[0]["geometry"] as? NSDictionary {
            if let location = geometry["location"] as? NSDictionary {
                let latitude = location["lat"] as! Float
                let longitude = location["lng"] as! Float
                print("\n\(latitude), \(longitude)")
            }
        }
    }
}
{% endhighlight %}

This method looks pretty straightforward. First it constructs a URL using the base URL, a zip code (provided as input argument), and the API key, then it serializes the data it finds at this URL into _JSON_ feed, and finally it parses the JSON response to obtain the latitude and longitude we were looking for. Now let's call this method in the __viewDidLoad()__ method:

{% highlight swift %}
getLatLngForZip("95014")
{% endhighlight %}

If you run the project now, you will see the following output in the console:

{% highlight swift %}
37.3132, -122.072
{% endhighlight %}

It worked! Next, let's write a method for _reverse geocoding_ using Google Maps:

{% highlight swift %}
func getAddressForLatLng(latitude: String, longitude: String) {
    let url = NSURL(string: "\(baseUrl)latlng=\(latitude),\(longitude)&key=\(apikey)")
    let data = NSData(contentsOfURL: url!)
    let json = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
    if let result = json["results"] as? NSArray {
        if let address = result[0]["address_components"] as? NSArray {
            let number = address[0]["short_name"] as! String
            let street = address[1]["short_name"] as! String
            let city = address[2]["short_name"] as! String
            let state = address[4]["short_name"] as! String
            let zip = address[6]["short_name"] as! String
            print("\n\(number) \(street), \(city), \(state) \(zip)")
        }
    }
}
{% endhighlight %}

This method is pretty similar to the first one. The only thing we changed is we provided the latitude and longitude as inputs to get the address information. Let's also call this method in __viewDidLoad()__:

{% highlight swift %}
getAddressForLatLng("37.331", longitude: "-122.031")
{% endhighlight %}

Run the project and you should see the following output at the console:

{% highlight swift %}
1 Infinite Loop, Cupertino, CA 95014
{% endhighlight %}

This is great! We were able to get the exact location of Apple's headquarters just by using the latitude and longitude. Ok, it's time to move on to using Apple Maps now and see how that works. First, let's import what we need:

{% highlight swift %}
import CoreLocation
import AddressBookUI
{% endhighlight %}

Then, let's write a method for _forward geocoding_ using Apple Maps:

{% highlight swift %}
func forwardGeocoding(address: String) {
    CLGeocoder().geocodeAddressString(address, completionHandler: { (placemarks, error) in
        if error != nil {
            print(error)
            return
        }
        if placemarks?.count > 0 {
            let placemark = placemarks?[0]
            let location = placemark?.location
            let coordinate = location?.coordinate
            print("\nlat: \(coordinate!.latitude), long: \(coordinate!.longitude)")
            if placemark?.areasOfInterest?.count > 0 {
                let areaOfInterest = placemark!.areasOfInterest![0]
                print(areaOfInterest)
            } else {
                print("No area of interest found.")
            }
        }
    })
}
{% endhighlight %}

Let's look at what this method does. First, it uses the __CLGeocoder__ class from the _CoreLocation_ framework to geocode an address we provide as input. In the method's _completion handler_ we parse the response and see what __placemark__ objects we found. We are particularly interested in the first one found, if there are more than one. Then we get the location coordinates and any areas of interest - as properties of the placemark. Now let's call this method in __viewDidLoad()__ again:

{% highlight swift %}
forwardGeocoding("1 Infinite Loop")
{% endhighlight %}

If you run the app now, you will see the data you would expect:

{% highlight swift %}
lat: 37.3316851, long: -122.0300674
Apple Inc.
{% endhighlight %}

Now let's write a method for _reverse geocoding_ using Apple Maps:

{% highlight swift %}
func reverseGeocoding(latitude: CLLocationDegrees, longitude: CLLocationDegrees) {
    let location = CLLocation(latitude: latitude, longitude: longitude)
    CLGeocoder().reverseGeocodeLocation(location, completionHandler: {(placemarks, error) -> Void in
        if error != nil {
            print(error)
            return
        }
        else if placemarks?.count > 0 {
            let pm = placemarks![0]
            let address = ABCreateStringWithAddressDictionary(pm.addressDictionary!, false)
            print("\n\(address)")
            if pm.areasOfInterest?.count > 0 {
                let areaOfInterest = pm.areasOfInterest?[0]
                print(areaOfInterest!)
            } else {
                print("No area of interest found.")
            }
        }
    })
}
{% endhighlight %}

The only detail we need to pay attention in this method is  __ABCreateStringWithAddressDictionary__ which is a class from the _AddressBookUI_ framework and which does the address parsing for us. This is really convenient as we do not need to care for any of the location fields if we just want to see the entire address. Let's also call this method in __viewDidLoad()__ one last time:

{% highlight swift %}
reverseGeocoding(37.3316851, longitude: -122.0300674)
{% endhighlight %}

Run the app and you will see this nicely formatted output in the console:

{% highlight swift %}
1 Infinite Loop
Cupertino‎ CA‎ 95014
United States
Apple Inc.
{% endhighlight %}

You can read more about [Apple Geocoding](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/LocationAwarenessPG/UsingGeocoders/UsingGeocoders.html) as well as [Google Geocoding](https://developers.google.com/maps/documentation/geocoding/intro). As a next step, you could take this information and place pins on a map view, for example, and show areas of interest. You can also implement a search feature for finding addresses based on your input. You could also implement an autocompletion feature for typing. There are many other opportunities you could think of.

Until next time!
