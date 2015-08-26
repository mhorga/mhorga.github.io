---
published: false
title: iOS persistence with NSCoder and NSKeyedArchiver
layout: post
---
As you have seen in the last post, [NSUserDefaults](http://mhorga.org/2015/08/20/ios-persistence-with-nsuserdefaults.html) is not as sophisticated as we want it to be in more complex scenarios such as persisting an `object graph` like an array of arrays, for example. Often times we get into situations where we do not see app persistence even though we should. 

Let's start with an example. When you create a `Master-Detail` project in `Xcode` you will notice that all the date entries you create will be lost if you run the app again. If we take a closer look at the code, inside `MasterViewController` we notice that a mutable array named __objects__ is used to store objects of the type `NSDate` every time we click the __+__ button at the right end of the navigation bar. So `objects` will be a proper object graph candidate for us to persist, and the right place to do it would be inside the __ insertNewObject()__ method.