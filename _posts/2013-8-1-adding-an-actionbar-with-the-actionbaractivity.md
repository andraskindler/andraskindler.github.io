---
layout: post
title:  "Adding an ActionBar with the ActionBarActivity"
date:   2013-08-01 10:00:00
categories: Android
excerpt: This tutorial shows how to use the ActionBarActivity from the support library in Android apps.
permalink: /blog/2013/adding-an-actionbar-with-the-actionbaractivity/
deprecated: true
---
The unveiling of Android 4.3 was followed by the arrival of a new Support Library ([revision 18](http://developer.android.com/tools/support-library/index.html)). Besides adding new features to the v4 library, Google introduced a new compatibility element in v7, which aims to bring the ActionBar UI pattern to pre-Honeycomb platforms. Requires API level 7, aka Android 2.1 / Eclair.

By the way, there was already an existing solution for this case: Jake Warton's [ActionBarSherlock](http://actionbarsherlock.com/) is widely used in apps, provides excellent features and easy usability; it might be a good idea to check it out if you're not already familiar with it.

Usage is pretty easy (and a bit similar to ActionBarSherlock). The first step of course is importing the library (in Eclipse: as library project, in Android Studio / IntelliJ: both as module and jar dependency). Adding an ActionBar requires three steps per activity:

*   Extending your activity from the ActionBarActivity class
*   Setting activity theme to a supported theme like Theme.AppCompat, Theme.AppCompat.Light or Theme.AppCompat.Light.DarkActionbar (these can be found [here](http://developer.android.com/reference/android/support/v7/appcompat/R.style.html))
*   Adding items in the onCreateOptionsMenu() method, supplied with a specific namespace

Sidenote: the [ActionBarActivity](http://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html) is a FragmentActivity subclass, meaning it supports fragments on platforms prior Honeycomb, so you won't lose fragment compatibility.

A short example, with a menu resource file called _menu.xml_:

```
public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // inflate the layout, etc...
        getSupportActionBar().setTitle("test");
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu, menu);
        return true;
    }
}
```

You can access the [ActionBar](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) with the getSupportActionBar() method (instead of the getActionBar(), available in the Activity class from Android 3.0). Many customizing features are available, including split actionbars, actionitems, using a custom view/theme, navigating up with the app icon, using the ShareActionProvider etc. The official ActionBar guide was updated with support package specific content, you can find more info on the subject [here](http://developer.android.com/guide/topics/ui/actionbar.html).

Declaring the items is almost the same as using the regular ActionBar: by creating a menu resource and using a custom namespace prefix at every actionbar-related attribute. An example:

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:yourapp="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/menu_item_1"
          yourapp:showAsAction="always"
          android:icon="@drawable/ic_launcher"
          android:title="@string/menu_item_1" />
</menu>
```