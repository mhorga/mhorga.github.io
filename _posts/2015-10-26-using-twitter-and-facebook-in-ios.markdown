---
published: true
title: Using Twitter and Facebook in iOS
layout: post
---
In this post, we will look into integrating the `Social Framework` for `iOS`, so we can post to `Twitter` and `Facebook` from other applications we want to integrate with the `Social Framework`. To start, let's create a `Single View Application` in `Xcode`. In the storyboard let's add a `UITextView` connected to an `IBOutlet` named __socialTextView__ and a `UIButton` on top of the text view, and connected to an `IBAction` named __shareAction__.

Next, let's write some code. In the view controller file, inside the __shareAction()__ method let's create a `UIAlertController`:

{% highlight swift %}
let actionController = UIAlertController(title: "Share", message: "", preferredStyle: UIAlertControllerStyle.Alert)
presentViewController(actionController, animated: true, completion: nil)
{% endhighlight %}

This will create a small pop-up window showing us an alert titled __Share__ when we press the button. As you can see there is no way of getting rid of this alert, so let's also add a button named __Cancel__ which is actually a `UIAlertAction`. Now we can dismiss the alert when we want to return back to the previous screen:

{% highlight swift %}
let cancelAction = UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Default, handler: nil)
actionController.addAction(cancelAction)
{% endhighlight %}

Then, import the __Social__ framework so we can use its Twitter and Facebook components:

{% highlight swift %}
import Social
{% endhighlight %}

Next, we will add a `Twitter` action to our alert controller:

{% highlight swift %}
let tweetAction = UIAlertAction(title: "Tweet", style: UIAlertActionStyle.Default) { _ in
    if SLComposeViewController.isAvailableForServiceType(SLServiceTypeTwitter) {
        let tweet = SLComposeViewController(forServiceType: SLServiceTypeTwitter)
        var text = self.socialTextView.text
        if text.characters.count > 140 {
            text = (self.socialTextView.text as NSString).substringToIndex(140)
        }
        tweet.setInitialText(text)
        self.presentViewController(tweet, animated: true, completion: nil)
    } else {
        let alert = UIAlertController(title: "Accounts", message: "You are not signed in to Twitter.", preferredStyle: UIAlertControllerStyle.Alert)
        alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
        self.presentViewController(alert, animated: true, completion: nil)
    }
}
{% endhighlight %}

Let's digest this code block. First, we created an action like we did for `Cancel` but this time we wanted to do something in the completion handler so we created a block. Note that when you have a block as the last parameter of a function, it can be taken outside of the parentheses for convenience. Inside, we create a `SLComposeViewController` which is a customized type of view controller provided by the `Social` framework. When choosing the service type, we pick __SLServiceTypeTwitter__ this time. 

Next, we want to grab the text we typed in our text view (if any) and set it as out tweet message. However, keep in mind that `Twitter` limits your messages to only `140` so we're trimming our message if it's longer than that. Finally, we need to also let the users know they are not logged in `Twitter` if this is the first time they're using `Twitter` on this device/simulator. In order to log in to `Twitter` they would need to go to the `Settings` app and scroll down to `Twitter` to log in. One more thing to do, add the `Twitter` action to our action controller:

{% highlight swift %}
actionController.addAction(tweetAction)
{% endhighlight %}

If you run the app, you will probably see a message that you are not logged in to `Twitter` so go on and log in and then run the app again. This time you will see the `Twitter` window ready to post your message. Be careful what you're saying as this will get posted to your live `Twitter` account. Next, let's do the same for the `Facebook` integration:

{% highlight swift %}
let facebookAction = UIAlertAction(title: "Facebook", style: UIAlertActionStyle.Default) { _ in
    if SLComposeViewController.isAvailableForServiceType(SLServiceTypeFacebook) {
        let facebookSheet = SLComposeViewController(forServiceType: SLServiceTypeFacebook)
        facebookSheet.setInitialText(self.socialTextView.text)
        self.presentViewController(facebookSheet, animated: true, completion: nil)
    } else {
        let alert = UIAlertController(title: "Accounts", message: "You are not signed in to Facebook.", preferredStyle: UIAlertControllerStyle.Alert)
        alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
        self.presentViewController(alert, animated: true, completion: nil)
    }
}
actionController.addAction(facebookAction)
{% endhighlight %}

We don't do anything different here, except we change the service type to __SLServiceTypeFacebook__. Run the app again and assuming you logged in to `Facebook` already, you will see a similar type of pop-up window for posting to `Facebook`. Again, be careful what you type as this will be posted to your live `Facebook` account. 

Finally, let's also create a more general type of integration, so we are able to share our message with any other frameworks, not only with the `Social` framework:

{% highlight swift %}
let moreAction = UIAlertAction(title: "More", style: UIAlertActionStyle.Default) { _ in
    let moreVC = UIActivityViewController(activityItems: [self.socialTextView.text], applicationActivities: nil)
    self.presentViewController(moreVC, animated: true, completion: nil)
}
actionController.addAction(moreAction)
{% endhighlight %}

This time we create a `UIActivityViewController`. If you run the app now, you will notice our message is not presented to us anymore, and we instead see the extensions view popping up from the bottom of the screen. This gives us even more sharing options for future integrations with other applications.

The source code is available on [Github](https://github.com/Swiftor/SocialFramework).

Until next time!