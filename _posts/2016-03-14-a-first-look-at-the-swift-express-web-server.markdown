---
published: false
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

Until next time!

{% highlight swift %} 
{% endhighlight %}