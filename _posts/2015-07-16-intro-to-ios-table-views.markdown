---
published: false
title: Intro to iOS table views
layout: post
---
Start with a new _Single View Application_ project. Go straight to the storyboard and add a table view to the view controller. On top of the table view, add a table view cell. Select the view controller. Under the _Identity Inspector_ make sure the _Class_ is set to _ViewController_. Select the table view cell. Under the _Attributes Inspector_ change _Style_ to _Subtitle_ and the _Identifier_ to _MyCellReuseIdentifier_. You are done with the storyboard configuration now, and it should look like this:

![storyboard](https://github.com/mhorga/mhorga.github.io/blob/master/images/project1.png "Storyboard")

Now go to the ViewController file.