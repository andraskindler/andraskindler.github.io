---
layout: post
title:  Using the SwipeRefreshLayout
date:   2014-4-13 15:06:00
categories: Android
excerpt: A post about implementing the pull-to-refresh pattern in Android apps with the SwipeRefreshLayout from the support library.
permalink: /blog/2014/playing-around-with-the-swiperefreshlayout/
deprecated: true
---
The pull to refresh is a UI pattern long since present in mobile applications - it makes the user able to refresh the current view with a vertical swipe. It's most commonly used in iOS apps, although it can also be found in quite a few Android apps. Not long ago Google made an Androidified version (first appeared in Google Now), where the refresh indicator is built into the ActionBar. Although a great library was available for a couple of months now ([ActionBar-PullToRefresh](https://github.com/chrisbanes/ActionBar-PullToRefresh) by Chris Banes), Google added an _official_ way to  to achieve this effect to the [Support Library v19.1.0](http://developer.android.com/tools/support-library/index.html), called [SwipeRefreshLayout](http://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html).

## How to use the SwipeRefreshLayout

SwipeRefreshLayout is fairy easy to use, just wrap it around a vertically scrollable View (like a ListView or ScrollView) for it to work correctly. The following example uses a ListView.

```
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/srl_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/lv_main"/>

</android.support.v4.widget.SwipeRefreshLayout>
```

The SwipeRefreshLayout takes care of detecting the vertical swipe, showing a progress animation on the ActionBar and also pushes down the layout. When a swipe occurs, the _OnRefreshListener_'s _onRefresh()_ method is triggered.

It is possible to trigger multiple _onRefresh()_ events while one is running, so you'll have to take care of everything to work correctly (RxJava's Subscription comes in handy). And while the refresh animation is triggered by a swipe, it will not stop when the task is finished, this has to be done manually with the _setRefreshing()_ method. Calling this function at startup (when the items are loading for the first time, without the user explicitly refreshing) can be a good way of providing consistent experience instead of using a ProgressBar.

You can set the refresh indicator colors with the _setColorScheme()_ method, which is a bit unflexible - the number of colors is always four, not more, not less. And an other thing: only resource identifiers supported, so the colors have to be defined in an xml, you can't supply them in code. 

```
final SwipeRefreshLayout swipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.srl_main);
swipeRefreshLayout.setColorScheme(R.color.blue, R.color.green, R.color.red, R.color.yellow);
swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
    @Override
    public void onRefresh() {
    // do work here
    });
```