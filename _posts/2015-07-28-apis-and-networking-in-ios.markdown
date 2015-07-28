---
published: false
title: APIs and networking in iOS
layout: post
---
In this short introduction to _networking_ in iOS, we will access images from _Flickr_. To start, go to the [Flickr API](flickr.com/services/api) web page and create an _API Key_ for your app. Once you have your key created, let's start by creating a new _Single View Application_ in _Xcode_. In the storyboard add an image view, a label and a button. Put the necessary constraints, and now let's connect the UI elements to our view controller. Create two _outlets_ for the image view and the label, and an _action_ for the button. The action will only serve one purpose: call the method that gets the images from Flickr.

On to the view controller, the skeleton code looks like this:

{% highlight swift %}     
class ViewController: UIViewController {
 
    let key = "YOUR_KEY_HERE"
    let galery_id = "5704-72157622566655097"
    
    @IBOutlet weak var photoImageView: UIImageView!
    @IBOutlet weak var photoTitle: UILabel!
    
    @IBAction func touchRefreshButton(sender: AnyObject) {
        getImage()
    }
    
    func getImage() {
        /* 1 - Initialize session and url */
        /* 2 - Initialize task for getting data */
        /* 3 - Parse the data */
        /* 4 - Grab a single, random image */
        /* 5 - Get the image url and title */
        /* 6 - If an image exists at the url, set the image and title */
        /* 7 - Execute the task */
    }
}
{% endhighlight %}

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator2.png "Flickr")

{% highlight swift %}     

{% endhighlight %}

Until next time!