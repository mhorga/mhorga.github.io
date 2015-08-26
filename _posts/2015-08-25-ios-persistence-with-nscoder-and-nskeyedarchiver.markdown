---
published: false
title: iOS persistence with NSCoder and NSKeyedArchiver
layout: post
---
As you have seen in the last post, [NSUserDefaults](http://mhorga.org/2015/08/20/ios-persistence-with-nsuserdefaults.html) is not as sophisticated as we want it to be in more complex situations such as an `object graph` such as an array of arrays or dictionaries. Often times we get into situations where we do not see app persistence even though we should. For example, when you create a `Master-Detail` project in `Xcode` you will notice that all the entries you create will be lost if you run the app again.