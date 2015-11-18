---
published: false
title: Using the Interface Builder in Xcode
layout: post
---
In this article I am going to talk about the `Interface Builder` and its main features that `iOS` developers use to design the user interface in their apps. A user interface file has either a __.storyboard__ or __.xib__ file extension. A `xib` file usually specifies one view controller or menu bar. A `storyboard` can have more view controllers and segues between those view controllers. By selecting the user interface file in the project navigator, the file’s contents open in `Interface Builder` in the editor area of the workspace window. 

1. Auto Layout

![alt text](https://github.com/Swiftor/InterfaceBuilder/raw/master/images/ib1.png "IB1")

- create a uiview that extends between the left and right margins of the screen and also edges the top of the screen, and give it a blue background
- below, create a button that has height 44 and also extends between the left and right margins of the screen but this time it edges the bottom of the screen
- create three constraints for each the uiview and the button, all with value 0 for the margins they’re extending to, one constraint for the button height, and one more 0 vertical spacing constraint between the uiview and the button
- run the project on various simulator sizes and orientations, and notice how the uiview and button are both filling the screen, while the button is maintaining its height

2. stack view

- let’s add 2 more buttons aligned horizontally with the initial button. first, remove all the constraints for the button (except the one for height) as we will not need them anymore. then drag the right margin of the button to shorten its width to about a quarter of its size. 
- to clone this button easily, simply press and hold the Option key while dragging the button next to itself, thus creating another one with the same size. repeat once more for a third button.
- next we want them to all have equal widths so they can fill the screen horizontally. we could just create constraints for each of them to match the width of the other, or we can use the new tool in iOS 9, the stack view
- press and hold Command while selecting all 3 buttons, one at a time. with all buttons selected, click the first icon (of the four) named Stack, which is located in the right bottom corner of the storyboard
- you will notice the buttons are neither equally spaced in the new stack, nor are they filling the the view horizontally. 
- to fix this, let’s select both the uiview and the stack view and stack them again as explained above. this time it will be a vertical stack view. in the Attributes Inspector panel choose Fill in the Alignment field. next, with the first (horizontal) stack view selected, in the Attributes Inspector choose Fill Equally in the Distribution field. 
- all we are left to do now is to set constraints for the stack view. instead of creating all the constraints manually, we can easily let the interface builder do it for us by selecting the rightmost icon this time, named Resolve Auto Layout Issues, and choose Add Missing Constraints under All Views in View Controller.
- now we are all set with a stacked view that has auto-layout set for any simulator sizes and orientations. 

7. ibdesignable and ibinspectable

- If you need to create code for a custom view that runs only in Interface Builder, call that code from the method prepareForInterfaceBuilder. Although it’s compiled for runtime, code called from prepareForInterfaceBuilder never gets called except by Interface Builder at design time.

Until next time!