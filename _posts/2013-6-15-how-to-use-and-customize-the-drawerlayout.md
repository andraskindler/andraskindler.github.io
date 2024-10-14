---
layout: post
title:  How to use and customize the DrawerLayout
date:   2013-06-15 08:00:00
categories: Android
excerpt: Tutorial about using the DrawerLayout in Android apps.
permalink: /blog/2013/how-to-use-and-customize-the-drawerlayout/
deprecated: true
---
The DrawerLayout was introduced in the latest support library (revision 13), and has many similarities with the SlidingPaneLayout. It is used to add navigation capabilities and to display more information in the same activity, with just a swipe from the sides - a great way to add extra functionality without stuffing the screen with controls. Just tell the user the panels are there, otherwise no one will know about them.

The first question is which side should you use in your application? Position the DrawerLayout on the left (start) of the screen if it contains navigation actions. If this is the case, you might want to check out Google's official [NavigationDrawer tutorial](http://developer.android.com/design/patterns/navigation-drawer.html). If the DrawerLayout contains actions specific to the current content, it should go on the right (end) side. For example if you create an app for playing music videos, you could put artist infos or playback options into the DrawerLayout. The good news is, you can use both sides in one DrawerLayout.

There are no options for putting a DrawerLayout to the two other sides, and there are good reasons for that. The top area is reserved for the dropdown status bar, a DrawerLayout here would confuse the user, not to mention it will be hard to use if the app isn't fullscreen. The bottom part is for Now (if you have a phone with on-screen navigation buttons, like a Nexus 4): a swype gesture brings up Google's search service.

So I'll start the example with adding three panels to a DrawerLayout. The first is always the main content, it has a width and height of the parent (aka match_parent). I added a View to the left and a fragment to the right, just to demonstrate that DrawerLayout works seamlessly with fragments. The first has a cyan background color, the latter is blue, and the main content is green.

```
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:id="@+id/dl_main"
    android:layout_height="match_parent" >

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/fl_main"
        android:background="#00ff00" />

    <View
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="end"
        android:background="#0000ff" />

    <fragment
        android:id="@+id/f_main_left"
        android:name="andraskindler.example.fragment.MockFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start" />

</android.support.v4.widget.DrawerLayout>
```

And that's all, the DrawerLayout is fully functional: swipe from the left for the fragment, from the right for the view. Of course if you want to get serious with some fancy effects, you'll have to get your hands dirty (with a few extra lines of code, that is). Some ideas for customization:

*   Listen for open and close events via a [DrawerListener](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html). Just override the required methods and implement your stuff.
*   If you want to open/close the drawers with a click of a button instead of a swipe, you can find everything you need [here](http://developer.android.com/training/implementing-navigation/nav-drawer.html).
*   use the setDrawerLockMode() method to enable or disable interaction with a drawer
*   You can change the scrim color, which overlays the main content while a drawer is open. Just use the _setScrimColor()_ method.
*   You can also change the shadow of the drawers with the setDrawerShadow() method. This requires a drawable resource (either from the /res folder or from code, just make sure it has a non-zero width). This can be done both for the left and right drawers, set the gravity parameter acordingly.

If you see stuttering and glitches, you might want to consider Google's advice, cited from the official documentation:
*Avoid performing expensive operations such as layout during animation as it can cause stuttering; try to perform expensive operations during the STATE_IDLE state*.
