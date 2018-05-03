+++
author = "Sebastian Suchanowski"
categories = ["Autolayout", "XCode", "iOS", "Tutorial"]
date = "2018-04-27"
description = "Fixing ambiguous scrollable content error using autolayout."
featured = "uiscrollview-has-ambiguous-scrollable-content-height.jpg"
featuredalt = "UIScrollView has an ambiguous scrollable content height"
featuredpath = "img"
linktitle = ""
title = "UIScrollView has an ambiguous scrollable content height"
type = "post"
+++


This error showed to us by XCode means in plain English that scroll view is not able to determine its height (or width) using autolayout. There is a very simple solution to this problem and I will walk you thru it.

## Our goal
To illustrate how properly use scroll view with autolayout we will build a simple UIViewController with potentially (dynamically loaded) long text and button at the end of the view.

<center>
![](/img/uiscrollview-fix-result.jpg)
</center>

## Step by step solution
Firstly let's start with adding all controls to our storyboard - we need here:

* UIScrollView
* UIView
* UILabel
* UIButton

Inside scroll view, we're gonna place a view which will take a role of content holder here and next we will place both label and button inside our content view. At the end it should look like this:

<center>
![](/img/uiscrollview-fix-view-hierarchy.jpg)
</center>

Now let's take care of setting up the autolayout for each view. Starting with UIScrollView we want to add 4 constraints, each with value 0 to its parent. Next, we will do the same for the UIView.

<center>
![](/img/uiscrollview-fix-constraints.jpg)
</center>

After that we will set constraints for UILabel and UIButton, idea is to have a 20px padding from each side. So label will have 3 constraints set to its container view and one set to button being on the bottom. We do a similar thing to UIButton.

<center>
![](/img/uiscrollview-fix-end-result-constraints.jpg)
</center>

And as we decided that we want to have a vertical scroll we need to do two final steps:
* Set UIView (content view) width to be equal as main UIView of the UIViewController - we need that to show scroll view that we don't want to scroll horizontally by setting a constant size here
* Set "Bounce Vertically" setting on UIScrollView to have this bounce effect even if our content is not exceeding the height of the screen

We should end up with something like this:

<center>
![](/img/uiscrollview-fix-result-animation.gif)
</center>

Full source code for this can be found on [GitHub](https://github.com/Synappse/tutorials/tree/master/ios/01-ambiguous-scrollable-content-height).
