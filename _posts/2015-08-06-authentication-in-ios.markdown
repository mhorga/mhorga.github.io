---
published: false
title: Authentication in iOS
layout: post
---
In a recent post I have written about [APIs and networking in iOS](http://mhorga.org/2015/07/28/apis-and-networking-in-ios.html). In that discussion we learned how to _anonymously_ access data. Today we will learn how to authenticate and access _user_ data. Before moving on, we should make a distinction between __authorization__ which means _allowing someone to perform a certain action_ while __authentication__ means _validating someone's identity_. An example of authorization is _OAuth_ when users log in using third party services (e.g. login with a Twitter or Facebook account). In order to authenticate, a user needs to first have an account. For this project we will create an account for [TheMovieDB API](https://www.themoviedb.org/documentation/api). Once the user logs in, a session has to be created so the user can interact with the API methods in order to get the desired information.

Let's first create a _Single View Application_ project in _Xcode_. In the storyboard add 2 text fields and set their placeholders to _username_ and _password_ respectively. Then add a button and name it _Login_. Finally, add a label under the login button so we can display debug messages when needed. Make sure to add all the necessary constraints. Then go to the view controller and add outlets for the text fields and label. Also create an action for the button:

{% highlight swift %}
    @IBOutlet weak var usernameTextField: UITextField!
    @IBOutlet weak var passwordTextField: UITextField!
    @IBOutlet weak var debugTextLabel: UILabel!
    
    @IBAction func loginButton(sender: UIButton) {
        if usernameTextField.text!.isEmpty {
            debugTextLabel.text = "Username Empty."
        } else if passwordTextField.text!.isEmpty {
            debugTextLabel.text = "Password Empty."
        } else {
            // create a session here
        }
    }
{% endhighlight %}

The next step would be for us to create a session after logging in was successful. The [API Sessions](https://www.themoviedb.org/documentation/api/sessions) documentation tells us what the steps are:

- Step 1: Create a new request token
- Step 2: Ask the user for permission via the API ("Login")
- Step 3: Create a session ID

For the first step, let's write a method named __getRequestToken__ which constructs the necessary URL to get a token. We would then call this method inside the _loginButton_ action method, right after the comment (_// create a session here_). We learned in the [APIs and networking in iOS](http://mhorga.org/2015/07/28/apis-and-networking-in-ios.html) post how to make a network call using _NSURLSession_ so let's just repeat those steps. Let's also add a few constants and variables we need:

{% highlight swift %}
    let apiKey = "YOUR_API_KEY"
    let getTokenMethod = "/authentication/token/new"
    let baseURLSecureString = "https://api.themoviedb.org/3"
    var requestToken: String?

    func getRequestToken() {
        let urlString = baseURLSecureString + getTokenMethod + "?api_key=" + apiKey
        let url = NSURL(string: urlString)!
        let request = NSMutableURLRequest(URL: url)
        request.addValue("application/json", forHTTPHeaderField: "Accept")
        let session = NSURLSession.sharedSession()
        let task = session.dataTaskWithRequest(request) { data, response, downloadError in
            if let error = downloadError {
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Login Failed. (Request token.)"
                }
                print("Could not complete the request \(error)")
            } else {
                let parsedResult = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
                if let requestToken = parsedResult["request_token"] as? String {
                    self.requestToken = requestToken
                    dispatch_async(dispatch_get_main_queue()) {
                        self.debugTextLabel.text = "got request token: \(requestToken)"
                    }
                } else {
                    dispatch_async(dispatch_get_main_queue()) {
                        self.debugTextLabel.text = "Login Failed. (Request token.)"
                    }
                    print("Could not find request_token in \(parsedResult)")
                }
            }
        }
        task.resume()
    }
{% endhighlight %}

Try logging in using dummy credentials and you should see a successful message printed on the label.

{% highlight swift %}

{% endhighlight %}

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator4.png "Login")

Until next time!