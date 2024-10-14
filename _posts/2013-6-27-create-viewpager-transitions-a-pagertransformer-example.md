---
layout: post
title:  "Create ViewPager transitions: a PagerTransformer example"
date:   2013-06-27 10:00:00
categories: Android
excerpt: Fancy swipe effects on the ViewPager with the PagerTransformer.
permalink: /blog/2013/create-viewpager-transitions-a-pagertransformer-example/
deprecated: true
---
The ViewPager is a great UI element, providing a smooth swipe animation for switching between pages. But what if a different scrolling effect is required? Fear not, the [support library](http://developer.android.com/tools/extras/support-library.html) provides a useful solution called the [PagerTransformer](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html). It was introduced in revision 11, and is supported from API level 11 (Honeycomb) or greater.

Usage is pretty straightforward, just attach a PageTransformer to the ViewPager:

```
viewpager.setPageTransformer(false, new ViewPager.PageTransformer() {
    @Override
    public void transformPage(View page, float position) {
        // do transformation here
        }
});
```

The _transformPage()_ method has a View and a position parameter. The former represents the current view or fragment, while the latter contains its position. Scrolling events are triggered by both the starting page and the target page, the corresponding _transformPage() _calls occur simultaneously. A value of zero means the current page is in the center, 1 means a full page offset to the right side, -1 means a full page offset to the left side. Quoted from the developer site:
> The position parameter indicates where a given page is located relative to the center of the screen. It is a dynamic property that changes as the user scrolls through the pages. When a page fills the screen, its position value is 0. When a page is drawn just off the right side of the screen, its position value is 1. If the user scrolls halfway between pages one and two, page one has a position of -0.5 and page two has a position of 0.5.
It is a good idea to normalize the position, so you don't have to bother with the numbers being negative or positive. It's pretty easy, just do this in every call:

```
final float normalizedposition = Math.abs(Math.abs(position) - 1);
```

Now you have a variable that goes from 0 to 1 (and the other way, respectively) if the user scrolls the ViewPager. What you do with it is up to your imagination; here are some basic examples. First, we'll fade the pages in and out:

```
@Override
public void transformPage(View page, float position) {
    final float normalizedposition = Math.abs(Math.abs(position) - 1);
    page.setAlpha(normalizedposition);
}
```

These lines perform a scaling effect from and to 50%:

```
@Override
public void transformPage(View page, float position) {
final float normalizedposition = Math.abs(Math.abs(position) - 1);
    page.setScaleX(normalizedposition / 2 + 0.5f);
    page.setScaleY(normalizedposition / 2 + 0.5f);
}
```

The last example rotates the pages around their Z axis by 30 degrees; you don't need to normalize for this one. The effect is similar to the _cover flow_ UI pattern:

```
@Override
public void transformPage(View page, float position) {
    page.setRotationY(position * -30);
}
```

Of course rotating the views or fragments around the axes is also possible, but the effect might be a bit overwhelming for the user, so I don't recommend it. Anyway, it can be done with the other _setRotation()_ methods.

These are just some basic examples, you can do much neater stuff with the PagerTransformer. You can also combine it with a [PagerTitleStrip](http://andraskindler.com/blog/2013/how-to-use-the-pagertabstrip-and-the-pagertitlestrip/) or a [PagerTabStrip](http://andraskindler.com/blog/2013/how-to-use-the-pagertabstrip-and-the-pagertitlestrip/).