---
published: false
title: Authentication in iOS
layout: post
---
In a recent post I have written about [APIs and networking in iOS](http://mhorga.org/2015/07/28/apis-and-networking-in-ios.html). In that discussion we learned how to _anonymously_ access data. Today we will learn how to authenticate and access _user_ data. Before moving on, we should make a distinction between __authorization__ which means _allowing someone to perform a certain action_ while __authentication__ means _validating someone's identity_. An example of authorization is _OAuth_ when users log in using third party services (e.g. login with a Twitter or Facebook account). 

In order to authenticate, a user needs to first have an account. For this project we will create an account for [TheMovieDB API](https://www.themoviedb.org/documentation/api). Once the user logs in, a session has to be created so the user can interact with the API methods in order to get the desired information.

{% highlight swift %}

{% endhighlight %}

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/simulator4.png "Login")

Until next time!