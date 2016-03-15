---
published: true
title: A first look at the Swift Express web server
layout: post
---
This week we are going to look at a great `Swift` initiative -- the __Swift Express__ web server -- a project run by [Crossroad Labs](http://crossroadlabs.xyz). `Swift Express` inherits the power of the `Play Framework` and the simplicity of the `Express.js` framework which makes it great for `Swift`-based web applications. A powerful feature that sets `Swift Express` aside from other `Swift` backend solutions is the synchronous-vs-asynchronous functional approach. `OS X` and `iOS` developers also benefit from the familiar `Swift + Xcode` development environment they're already used to, and they will appreciate the fact that `Swift Express` provides an easy way to create `RESTful`,  `JSON` and `XML APIs` using `Swift`.

Alright, let's dive in and take a look at `Swift Express`. You need to have `Xcode 7.2` or newer. Next, run these commands in a terminal:

{% highlight swift %} 
$ xcode-select --install
$ brew update
$ brew tap crossroadlabs/tap
$ brew install swift-express
{% endhighlight %}

We're now ready to create a project:

{% highlight swift %}
$ swift-express init HelloExpress
$ cd HelloExpress
$ swift-express bootstrap
$ open HelloExpress.xcodeproj 
{% endhighlight %}

As soon as `Xcode` opens your project, go to __main.swift__ and notice we already have all the necessary code we need in there in order to start the server:

{% highlight swift %}
import Express

let app = express()
app.views.register(StencilViewEngine())
app.get("/assets/:file+", action: StaticAction(path: "public", param:"file"))
app.get("/") { (request:Request<AnyContent>)->Action<AnyContent> in
    return Action<AnyContent>.render("index", context: ["hello": "Hello,", "swift": "Swift", "express": "Express!"])
}
app.listen(9999).onSuccess { server in
    print("Express was successfully launched on port", server.port)
}
app.run()
{% endhighlight %}

All you need to do now is run the project. In the console you should see this message:

{% highlight text %} 
Express was successfully launched on port 9999
{% endhighlight %} 

Now open your browser and point it to [localhost:9999](http://localhost:9999) and you should see this page:

![Swift Express](http://i.imgur.com/CDJEr3h.png "Swift Express")

Let's do some more! Right above the __app.listen__ line add this new API call we are just creating now:

{% highlight swift %} 
app.get("/new") { request in
    return Action.ok(request.query["message"]?.first)
}
{% endhighlight %}

You can also update and start the server from the terminal, not just in Xcode:

{% highlight text %}
$ swift-express build
$ swift-express run 
{% endhighlight %}

Now, test it in the browser at [localhost:9999/new?message=Hello](http://localhost:9999/new?message=Hello). You should see a white page with just __Hello__ written on it, or whatever string you choose to put after the __=__ character in the `URL`. You can also create pages in the good ol' way, so add this new API call to your class:

{% highlight swift %} 
app.get("/hello") { request in
    return Action.ok(AnyContent(str: "<h1>Hello Express!!!</h1>", contentType: "text/html"))
}
{% endhighlight %}

Again, build and run your project from command line, and then point your browser to [localhost:9999/hello](http://localhost:9999/hello). You should see only the header message __Hello Express!!!__.

Ok, you might say, but this is a web service and we want it to let us get data feeds from it. Easy task! First, we need to register the JSON view in the system, so put this line next to the other registered view:

{% highlight swift %} 
app.views.register(JsonView()):
{% endhighlight %}

Second, create a new API call and make sure it generates JSON-friendly data, such as a dictionary.  

{% highlight swift %} 
app.get("/election") { request in
    let candidates = [
        [ "name": "Bernie" ],
        [ "name": "Hillary" ],
        [ "name": "Donald" ],
    ]
    return Action.render(JsonView.name, context: candidates)
}
{% endhighlight %}

Build and run your project again, and point your browser to [localhost:9999/election](http://localhost:9999/election). The page should return the un-formatted feed:

{% highlight swift %} 
[{"name":"Bernie"},{"name":"Hillary"},{"name":"Donald"}]
{% endhighlight %}

In conclusion, I was really impressed by the ease and the practicality of using `Swift Express` to generate a web service in a matter of minutes! Not only we have the convenience of writing all our app code in Swift, but now we can also write our back-end structure in Swift as well. For more information about the open source project, see the [Swift Express](https://github.com/crossroadlabs/Express) documentation. You can look at the complete list of features and see the roadmap. You can (and should) also contribute to this project!

Until next time!