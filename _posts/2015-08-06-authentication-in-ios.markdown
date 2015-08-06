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

{% highlight swift %}

{% endhighlight %}

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator4.png "Login")

Until next time!