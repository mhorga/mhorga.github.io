---
published: true
title: Authentication in iOS
layout: post
---
In a recent post I wrote about [APIs and networking in iOS](http://mhorga.org/2015/07/28/apis-and-networking-in-ios.html). In that discussion we learned how to _anonymously_ access data. Today we will learn how to authenticate and access personalized _user_ data. Before moving on, we need to look at the difference between __authorization__ which means _allowing someone to perform a certain action_ while __authentication__ means _validating someone's identity_. An example of authorization is _OAuth_ when users log in using third party services (e.g. login with a Twitter or Facebook account). In order to authenticate, a user needs to first have an account. For this project we will create an account for [TheMovieDB API](https://www.themoviedb.org/documentation/api). Once the user logs in, a session has to be created so the user can interact with the API methods in order to get the desired information.

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
- Step 2: Ask the user for permission via the API
- Step 3: Create a session ID
- Step 4 (optional): Get the user id
- Step 5 (optional): Display user information

For _Step 1_, let's write a method named __getRequestToken__ which constructs the necessary URL to get a token. We would then call this method inside the _loginButton_ action method, right after the comment line:
`// create a session here`
We learned in the [APIs and networking in iOS](http://mhorga.org/2015/07/28/apis-and-networking-in-ios.html) post how to make a network call using _NSURLSession_ so let's just repeat those steps. Let's also add a few constants and variables we need:

{% highlight swift %}
let apiKey = "YOUR_API_KEY"
let getTokenMethod = "authentication/token/new"
let baseURLSecureString = "https://api.themoviedb.org/3/"
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
                // we will soon replace this successful block with a method call
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

For _Step 2_, let's log in using the token we got in the first step. Replace the successful block above with a call:
`self.loginWithToken(self.requestToken!)`
to a new method that we will create next:

{% highlight swift %}
let loginMethod = "authentication/token/validate_with_login"

func loginWithToken(requestToken: String) {
    let parameters = "?api_key=\(apiKey)&request_token=\(requestToken)&username=\(self.usernameTextField.text!)&password=\(self.passwordTextField.text!)"
    let urlString = baseURLSecureString + loginMethod + parameters
    let url = NSURL(string: urlString)!
    let request = NSMutableURLRequest(URL: url)
    request.addValue("application/json", forHTTPHeaderField: "Accept")
    let session = NSURLSession.sharedSession()
    let task = session.dataTaskWithRequest(request) { data, response, downloadError in
        if let error = downloadError {
            dispatch_async(dispatch_get_main_queue()) {
                self.debugTextLabel.text = "Login Failed. (Login Step.)"
            }
            print("Could not complete the request \(error)")
        } else {
            let parsedResult = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
            if let success = parsedResult["success"] as? Bool {
                // we will soon replace this successful block with a method call
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Login status: \(success)"
                }
            } else {
                if let status_code = parsedResult["status_code"] as? Int {
                    dispatch_async(dispatch_get_main_queue()) {
                        let message = parsedResult["status_message"]
                        self.debugTextLabel.text = "\(status_code): \(message!)"
                    }
                } else {
                    dispatch_async(dispatch_get_main_queue()) {
                        self.debugTextLabel.text = "Login Failed. (Login Step.)"
                    }
                    print("Could not find success in \(parsedResult)")
                }
            }
        }
    }
    task.resume()
}
{% endhighlight %}

This time you will have to use your real credentials to log in. If everything went right you should see a successful message telling you that you are now logged in. 

For _Step 3_, we need to get a session ID. Replace the successful login block in the code above with a call:
`self.getSessionID(self.requestToken!)`
to a new method that we will create next:

{% highlight swift %}
let getSessionIdMethod = "authentication/session/new"
var sessionID: String?

func getSessionID(requestToken: String) {
    let parameters = "?api_key=\(apiKey)&request_token=\(requestToken)"
    let urlString = baseURLSecureString + getSessionIdMethod + parameters
    let url = NSURL(string: urlString)!
    let request = NSMutableURLRequest(URL: url)
    request.addValue("application/json", forHTTPHeaderField: "Accept")
    let session = NSURLSession.sharedSession()
    let task = session.dataTaskWithRequest(request) { data, response, downloadError in
        if let error = downloadError {
            dispatch_async(dispatch_get_main_queue()) {
                self.debugTextLabel.text = "Login Failed. (Session ID.)"
            }
            print("Could not complete the request \(error)")
        } else {
            let parsedResult = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
            if let sessionID = parsedResult["session_id"] as? String {
                self.sessionID = sessionID
                // we will soon replace this successful block with a method call
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Session ID: \(sessionID)"
                }
            } else {
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Login Failed. (Session ID.)"
                }
                print("Could not find session_id in \(parsedResult)")
            }
        }
    }
    task.resume()
}
{% endhighlight %}

If you log in again with your credentials, you should see the session ID printed on the label. A next logical step would be to get the user ID using the session ID we just got. Replace the successful block in the code above with a call:
`self.getUserID(self.sessionID!)`
to a new method that we will create next:

{% highlight swift %}
let getUserIdMethod = "account"
var userID: Int?

func getUserID(sessionID: String) {
    let urlString = baseURLSecureString + getUserIdMethod + "?api_key=" + apiKey + "&session_id=" + sessionID
    let url = NSURL(string: urlString)!
    let request = NSMutableURLRequest(URL: url)
    request.addValue("application/json", forHTTPHeaderField: "Accept")
    let session = NSURLSession.sharedSession()
    let task = session.dataTaskWithRequest(request) { data, response, downloadError in
        if let error = downloadError {
            dispatch_async(dispatch_get_main_queue()) {
                self.debugTextLabel.text = "Login Failed. (Get userID.)"
            }
            print("Could not complete the request \(error)")
        } else {
            let parsedResult = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
            if let userID = parsedResult["id"] as? Int {
                self.userID = userID
                // we will soon replace this successful block with a method call
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "your user id: \(userID)"
                }
            } else {
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Login Failed. (Get userID.)"
                }
                print("Could not find user id in \(parsedResult)")
            }
        }
    }
    task.resume()
}
{% endhighlight %}

Run the app again and if everything went right, you should see the user id displayed. Now that we got everything we need, we can display personalized information about the user. Replace the successful block in the code above with a call:
`self.completeLogin()`
to a new method that we will create next:

{% highlight swift %}
func completeLogin() {
    let getFavoritesMethod = "account/\(userID)/favorite/movies"
    let urlString = baseURLSecureString + getFavoritesMethod + "?api_key=" + apiKey + "&session_id=" + sessionID!
    let url = NSURL(string: urlString)!
    let request = NSMutableURLRequest(URL: url)
    request.addValue("application/json", forHTTPHeaderField: "Accept")
    let session = NSURLSession.sharedSession()
    let task = session.dataTaskWithRequest(request) { data, response, downloadError in
        if let error = downloadError {
            dispatch_async(dispatch_get_main_queue()) {
                self.debugTextLabel.text = "Cannot retrieve information about user \(self.userID)."
            }
            print("Could not complete the request \(error)")
        } else {
            let parsedResult = try! NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments) as! NSDictionary
            if let results = parsedResult["results"] as? NSArray {
                dispatch_async(dispatch_get_main_queue()) {
                    let firstFavorite = results.firstObject as? NSDictionary
                    let title = firstFavorite?.valueForKey("title")
                    self.debugTextLabel.text = "Title: \(title!)"
                }
            } else {
                dispatch_async(dispatch_get_main_queue()) {
                    self.debugTextLabel.text = "Cannot retrieve information about user \(self.userID)."
                }
                print("Could not find 'results' in \(parsedResult)")
            }
        }
    }
    task.resume()
} 
{% endhighlight %}

Run the application again and you should see the first movie from your favorites list printed on the label. For simplicity, I went to my online profile page and favorited only one movie as you can see from the image below:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator4.png "Login")

All the network calls we made so far are GET requests which means we have not written anything to our user account using the API. In order to be able to modify personalized information, such as adding or removing movies from the list of favorites, we need to make POST request using the same API. 

Until next time!
