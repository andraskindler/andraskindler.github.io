---
layout: post
title:  How to use the PagerTabStrip and the PagerTitleStrip
date:   2013-06-22 11:00:00
categories: Android
excerpt: The PagerTabStrip and PagerTitleStrip can be used to show titles above the ViewPager.
permalink: /blog/2013/how-to-use-the-pagertabstrip-and-the-pagertitlestrip/
deprecated: true
---
The ViewPager is one of the best and most versatile part of the Android UI, it enables the developer/designer to show more content on a small screen than possible. The user can swipe between pages, but it is important that the app shows some hints for a proper user experience. Can I swipe some more or is it the last page? What awaits on the previous and next pages?

The support library has two built-in solutions for this situation by providing a View which shows the title of the current page while the previous and next titles are visible on the sides. The PagerTitleStrip was introduced in revision 6 (december 2011), followed briefly by the PagerTabStrip, which was added in revision 9 (june 2012). The latter is a far more powerful tool then the former: it's interactive (if you click the titles, the pager will set scroll to the proper item accordingly) and has more options to customize appearance. So I recommend the PagerTabStrip, but to be fair, this post will show how to use both.

We'll start with the titles, which come from a PagerAdapter subclass: you have to override the _getPageTitle()_ method, which returns the title of the corresponding page. The titles can come from a class variable if you have an object behind each page, but a common solution is to use a static String array containing the titles in the adapter. An example:

```
private static final String[] titles = { "one", "two", "three", "four", "five" };
//...

@Override
public CharSequence getPageTitle(int position) {
    return titles[position];
}
```

The next step is adding the strip, which can be done quite easily in a layout xml, just add the PagerTabStrip or the PagerTitleStrip to the ViewPager as a child. See below for a PagerTabStrip; usage is the same for a PagerTitleStrip. By default, the strip goes above the ViewPager, but you can position it below by adding the _android:layout_gravity="bottom"_ attribute to the Tab- or TitleStrip.

```
<android.support.v4.view.ViewPager
    android:id="@+id/vp_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.PagerTabStrip
        android:id="@+id/pts_main"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</android.support.v4.view.ViewPager>
```

And that's it, the only thing left to do is to customize. The possibilities are adequate, but if you're looking for a more flexible solution, you should consider looking into Jake Wharton's [ViewPagerIndicator](https://github.com/JakeWharton/Android-ViewPagerIndicator). The following list shows what is possible:

*   setting the background color or resource
*   setting the textsize and -color
*   the spacing between titles can be adjusted with the _setTextSpacing()_ method
*   positioning the strip above or below the ViewPager with the layout_gravity attribute
*   the _setNonPrimaryAlpha()_ method sets the alpha value (aka the transparency) for non-current titles left and right to the current title
*   setting the indicator color or resource (the rectangle below the titles) (_PagerTabStrip only_)
*   use the _setDrawFullUnderline()_ method to enable or disable the divider below the strip (_PagerTabStrip only_)

This is how to use said methods in Java:

```
final PagerTabStrip strip = PagerTabStrip.class.cast(root.findViewById(R.id.pts_main));
strip.setDrawFullUnderline(false);
strip.setTabIndicatorColor(Color.DKGRAY);
strip.setBackgroundColor(Color.GRAY);
strip.setNonPrimaryAlpha(0.5f);
strip.setTextSpacing(25);
strip.setTextSize(TypedValue.COMPLEX_UNIT_DIP, 16);
```