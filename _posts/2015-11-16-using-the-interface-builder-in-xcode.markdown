---
published: true
title: Using the Interface Builder in Xcode
layout: post
---
In this article I am going to talk about the `Interface Builder` and its main features that `iOS` developers use to design the user interface in their apps. A user interface file has either a __.storyboard__ or __.xib__ file extension. A `xib` file usually specifies one view controller or menu bar. A `storyboard` can have more view controllers and segues between those view controllers. By selecting the user interface file in the project navigator, the file’s contents open in `Interface Builder` in the editor area of the workspace window. 

## 1. Auto Layout ##

Let's start by creating a `UIView` that extends between the left and right margins of the screen and that it also edges the top of the screen, and for fun just give it a blue background. Below, create a `UIButton` that has the height __44__ and which also extends between the left and right margins of the screen but this time it edges the bottom of the screen. 

Create three constraints for each the view and the button, all with value __0__ for the margins they’re extending to, one constraint for the button height, and one more __0__ vertical spacing constraint between the view and the button as showed below:

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib1.png "IB1")

Run the project on various simulator sizes and orientations, and notice how the uiview and button are both filling the screen, while the button is maintaining its height. This is basic `Auto Layout` in action. You can also see various screen sizes and orientations, without having to run the app, by using the `Preview` mode of the storyboard which can be chosen by switching to the `Show the Assistant editor` mode:

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib2.png "IB2")

## 2. Stack Views ##

Let’s add two more buttons aligned horizontally with the initial button. First, remove all the button constraints except the one for `height`, as we will not need them anymore. Then drag the right margin of the button to shorten its width to about a quarter of its size. To clone this button, simply press and hold the `Option` key while dragging the button next to itself, thus creating another one with the same size. Repeat once more for the third button:

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib3.png "IB3")

Next, we want them to all have equal widths so they can fill the screen horizontally. We could just create constraints for each of them to match the width of the others, or we can use the new tool in `iOS 9`, the `Stack View`. Press and hold the `Command` key while selecting all the buttons, one at a time. With all the buttons selected, click the first icon (of the four) named __Stack__, which is located in the right bottom corner of the storyboard. Voila! The buttons are all stacked now and we do not need to arrange them in any way anymore.

You will notice the buttons are neither equally spaced in the new stack, nor are they filling the view horizontally. To fix this, let’s select both the `UIView` and the `UIStackView` so we can stack them again as explained above. This time it will be a vertical stack view. Once created, select it. In `Attributes Inspector` choose `Fill` in the `Alignment` field. This will ensure the view will fill all the vertical space except the __44__ pixel points taken by the button. Next, with the first (horizontal) stack view selected, in the `Attributes Inspector` choose `Fill Equally` in the `Distribution` field:

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib4.png "IB4")

All we are left to do now is set constraints for the two stack views. Instead of creating all the constraints manually, we can easily let the `Interface Builder` do it for us by selecting the rightmost icon this time, named __Resolve Auto Layout Issues__, and by choosing __Add Missing Constraints__ under `All Views in View Controller`. Now we are all set with a stacked view that has `Auto Layout` working for all simulator sizes and orientations:

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib5.png "IB5")

## 3. IBDesignable and IBInspectable ##

What if we were able to see in real time the effect of the changes we're making in storyboard and in code? Good news, this is possible by using `IBDesignable` and `IBInspectable`. I will talk about them in a part two of this article.

Until next time!