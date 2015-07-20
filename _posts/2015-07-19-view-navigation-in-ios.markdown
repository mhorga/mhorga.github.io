---
published: false
title: View navigation in iOS
layout: post
---
Start by creating a new _Single View Application_ project in _Xcode_. Go to the storyboard and select the view controller. On the Xcode menu, go to Editor --> Embed In --> Navigation Controller. You will notice now the view has a navigation bar. Inside the view drag a text view and two buttons. Set the necessary constraints. Write "What do you want to do next?" in the text view. Name one of the buttons "Go to screen 2" and the other "Go to screen 3". Now copy and paste four times the only view we have so far, so we can preserve the text view and the buttons we already set up, to save time. To copy an entire view controller, drag a rectangle around it in the Storyboard to select everything, then just press command-c followed by command-v. The new view controller will be pasted on top of the old one, so just drag it aside to see both.