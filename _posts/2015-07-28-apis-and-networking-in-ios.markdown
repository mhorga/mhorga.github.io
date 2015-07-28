---
published: true
title: APIs and networking in iOS
layout: post
---
In this short introduction to _networking_ in iOS, we will access images from _Flickr_. To start, go to the [Flickr API](flickr.com/services/api) web page and create an _API Key_ for your app. Once you have your key created, let's start by creating a new _Single View Application_ in _Xcode_. In the storyboard add an image view, a label and a button. Put the necessary constraints, and now let's connect the UI elements to our view controller. Create two _outlets_ for the image view and the label, and an _action_ for the button. The action will only serve one purpose: call the method that gets the images from Flickr.

In the view controller class, make sure to assign the API Key you created earlier to the __key__ variable. The __gallery_id__ represents just a sample gallery I chose but you could use yours if you have one. The skeleton code looks like this:

{% highlight swift %}     
class ViewController: UIViewController {
 
    let key = "YOUR_KEY_HERE"
    let gallery_id = "5704-72157622566655097"
    
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

Let's start writing our networking logic by grabbing the singleton instance of _NSURLSession_ and by creating the _URL_ and an instance of _NSURLRequest_. You will notice we hard coded the URL, but in general it is considered good practice to start with a _base url_ and then append _API parameters_ from a dictionary containing all the parameter _key-value pairs_ to construct the final URL. However, we will not cover this topic now, so let's see how the initialization looks like:

{% highlight swift %}     
/* 1 - Initialize session and url */
let session = NSURLSession.sharedSession()
let urlString = "https://api.flickr.com/services/rest?method=flickr.galleries.getPhotos&extras=url_m&format=json&nojsoncallback=1&gallery_id=\(gallery_id)&api_key=\(key)"
let url = NSURL(string: urlString)!
let request = NSURLRequest(URL: url)
{% endhighlight %}

In the next step, using the session and request, we instantiate the task for grabbing an image from Flickr:

{% highlight swift %}     
/* 2 - Initialize task for getting data */
let task = session.dataTaskWithRequest(request) {data, response, downloadError in
    if let error = downloadError {
        print("Could not complete the request \(error)", appendNewline: false)
    } else {
        /* steps 3 - 6 */
    }
}
{% endhighlight %}

In the completion handler, we parse the JSON response data and turn it into usable data:

{% highlight swift %}     
/* 3 - Parse the data */
let parsedResult: AnyObject!
do {
    parsedResult = try NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments)
} catch let error as NSError {
    print("\(error)", appendNewline: false)
    parsedResult = nil
} catch {
    fatalError()
}
if let photosDictionary = parsedResult.valueForKey("photos") as? NSDictionary {
    if let photoArray = photosDictionary.valueForKey("photo") as? [[String: AnyObject]] {        
        /* steps 4 - 6 */
    } else {
        print("Cant find key 'photo' in \(photosDictionary)", appendNewline: false)
    }
} else {
    print("Cant find key 'photos' in \(parsedResult)", appendNewline: false)
}
{% endhighlight %}

Since our request will return JSON for multiple images in our gallery, we just grab one of the images:

{% highlight swift %}     
/* 4 - Grab a single, random image */
let randomPhotoIndex = Int(arc4random_uniform(UInt32(photoArray.count)))
let photoDictionary = photoArray[randomPhotoIndex] as [String: AnyObject]
{% endhighlight %}

Then we grab the selected image's title and URL:

{% highlight swift %}
/* 5 - Get the image url and title */
let photoTitle = photoDictionary["title"] as? String
let imageUrlString = photoDictionary["url_m"] as? String
let imageURL = NSURL(string: imageUrlString!)
{% endhighlight %}

If there is data at the URL, then we update the photoImageView and photoTitle:

{% highlight swift %}
/* 6 - If an image exists at the url, set the image and title */
if let imageData = NSData(contentsOfURL: imageURL!) {
    dispatch_async(dispatch_get_main_queue(), {
        self.photoImageView.image = UIImage(data: imageData)
        self.photoTitle.text = photoTitle
    })
} else {
    print("Image does not exist at \(imageURL)", appendNewline: false)
}
{% endhighlight %}

At the very end of the class, right after the last curly brace that closes the task initialization, write the code to execute the task. We are now ready to execute our task and see the result:

{% highlight swift %}     
/* 7 - Execute the task */
task.resume()
{% endhighlight %}

If you got everything right, you should see a similar screen in your simulator or on your device:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator2.png "Flickr")

You can do more research into the Flickr API and implement new features such as being able to search for a specific user or image, instead of randomly displaying one.

Until next time!