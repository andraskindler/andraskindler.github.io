---
layout: post
title:  Translucent system bars + ListView done right
date:   2014-10-10 12:00:00
categories: Android
excerpt: A guide about how to properly use the translucent theme with ListViews or other vertically scrolling ViewGroups.
permalink: /blog/2014/translucent-system-bars-listview/
deprecated: true
---
About a year ago, Google introduced the translucent UI in [KitKat](https://developer.android.com/about/versions/kitkat.html), making the system bars transparent, with a hint of black gradient. After activating, apps are able to draw under the status bar (the one on top of the screen, with the notifications) and the navigation bar (this is optional, only present devices with no hardware buttons, at the bottom of the screen, contains the back, home and app switcher keys), which can result in a slick and modern user experience if done right.

But while the UI can spread under both bars, touch events in these areas will be ignored by the app, making the items in the bottom and top partially unclickable (see the screenshot on the left). This is why the layout has to be adjusted accordingly. But what can be done with a fullscreen ListView to keep the neat scrolling effect where items go under the status and nav bars, while the first and last items remain clickable?

The idea is to add a top and bottom padding matching the status and navigation bars (if they're translucent), and setting the _clipToPadding_ attribute to false. The latter means that when the list items and the padding moves simultaneously when scrolling the list.

As a first step, enable the translucent system bars in the _styles.xml_:

```
<style>
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
</style>
```

Next, we'll determine if the translucent system bars are enabled, and if so, apply the proper padding - the height of the status bar on top, the height of the navigation bar on bottom.

```
//...
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
    listView.setPadding(0, getStatusBarHeight(this), 0, getNavigationBarHeight(this));
    listView.setClipToPadding(false);            
}
//...
```

Calculating the paddings:

```
private int getStatusBarHeight(final Context context) {
    final int resourceId = context.getResources().getIdentifier("status_bar_height", "dimen", "android");
    if (resourceId > 0) {
        return context.getResources().getDimensionPixelSize(resourceId);
    }
    return 0;
}

private int getNavigationBarHeight(final Context context) {
    final int id = context.getResources().getIdentifier("config_enableTranslucentDecor", "bool", "android");
    if (id != 0 && context.getResources().getBoolean(id)) {
        final int resourceId = context.getResources().getIdentifier("navigation_bar_height", "dimen", "android");
        if (resourceId > 0) {
            return context.getResources().getDimensionPixelSize(resourceId);
        }
    }
    return 0;
}
```

That's all, translucent system bars and ListViews can work well together with this fairly simple solution. The code runs on APLI LEVEL < 19 as well.