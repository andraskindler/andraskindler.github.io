---
layout: post
title:  Animating the colors of a View
date:   2015-03-20 06:00:00
categories: android
excerpt: A tutorial about how to animate color parameters of Views using the ObjectAnimator, the ValueAnimator, and the ArgbEvaluator.
permalink: /blog/2015/animating_color_properties/
deprecated: true
---
We're working on a huge update for Ready Contact List, including a full material makeover. The app comes with basic theming options, meaning the prominent colors can be adjusted on-demand. We wanted to improve the user experience by crossfading between the new and old colors when the user changes something instead of just switching them. This post concludes what we are doing.

## ObjectAnimator
The easiest way of doing this is using an [ObjectAnimator](http://developer.android.com/reference/android/animation/ObjectAnimator.html). Since it can animate any property that has a setter, it can be used to animate certain colors of a view. The following piece of code fades the background color of a TextView from red to blue, in a single line.

`ObjectAnimator.ofInt(textView, "backgroundColor", Color.RED, Color.BLUE).start();`

Pretty straightforward, right? However, the ObjectAnimator has some usability and performance issues. One ObjectAnimator instance means only one animated property, multiple instances are necessary to animate different aspects of a view (or multiple views). Also, it uses a [reflection- or JNI-based mechanism](http://android-developers.blogspot.hu/2011/05/introducing-viewpropertyanimator.html) to turn the String property into a setter, for example the `setTextColor` parameter gets mapped to the `setTextColor()` function, causing quite the overhead. We can do better!

## ValueAnimator plus ArgbEvaluator
An other approach is to do things more manually, meaning a bit more code is required, but having more control over what's happening, and less performance-issues. The idea is that Android represents colors as integers holding 4 bytes of information, one for each component - alpha, red, blue and green. Fading between to colors stands for moving from the start color towards the end color, and on each tick, setting the color proportional to the time - this is exactly what a [ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html) and an [ArgbEvaluator](http://developer.android.com/reference/android/animation/ArgbEvaluator.html) does, if used together.

The ArgbEvaluator takes care of computing the proper color between the start and end values. It divides the start and end colors into individual values for the alpha, red, green and blue channels, then calculates the linearly interpolated result on each tick. The ArgbEvaluator is required because without it, the animator would treat the start and end values as integers instead of complex colors wrapped as integers, resulting in a lot flickering and unwanted colors.

The following code illustrates how to crossfade the text color of a TextView from red to blue.

```
final ValueAnimator colorAnimator = ValueAnimator.ofObject(new ArgbEvaluator(), Color.RED, Color.BLUE);
colorAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
  @Override
  public void onAnimationUpdate(ValueAnimator animator) {
    textView.setTextColor((int) animator.getAnimatedValue());
  }
});
colorAnimator.start();
```

The work is done in the AnimatorUpdateListener, so there's room for more than one property and/or View. Also, this can be used to animate the color properties of some non-traditional UI elements, like the status bar and the navigation bar. And there's no reflection involved!


Important: on Android versions less or equal to Jelly Bean (Android 4.1 or API level 16), a [bug](https://code.google.com/p/android/issues/detail?id=36158) is causing the ArgbEvaluator not to evaluate alpha value properly. If the alpha value is changing as well and API level <= 16 is supported as well, copying and using the [source code from AOSP](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/animation/ArgbEvaluator.java) does the trick. Thanks to [kevin_teslacoilsw](http://www.reddit.com/r/androiddev/comments/2zp7uu/animating_the_colors_of_a_view/)!

## Crossfading gradient colors
Animating the colors of a [GradientDrawable](http://developer.android.com/reference/android/graphics/drawable/GradientDrawable.html) is also possible with the technique detailed above, by calling the `setColors()` method in the listener with the proper colors.

## TransitionDrawable
The [TransitionDrawable](http://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html) can crossfade between two drawables, and while the ValueAnimator and ArgbEvaluator combo is a better fit for solid colors, this one can be used perfectly with bitmaps.

## Takeaways
If the colors of a layout change dynamically, animating these changes can do good to the user experience. The combination of the ValueAnimator and the ArgbEvaluator works great for fading colors of a View - I ended up using it as well.
