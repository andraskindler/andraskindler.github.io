---
layout: post
title:  Making live wallpaper apps respond to touches correctly
date:   2015-02-03 17:00:00
categories: android
tags: Live Wallpaper onOffsetsChanged
image: 2015-02-03-live-wallpaper-onoffsetchanged-scrolling.png
---
This post is about ensuring that a live wallpaper app scrolls properly on all Android devices. This is huge issue on certain launchers, as they are not calling the onOffsetsChanged() function correctly, which notifies the wallpaper app that the home screen was scrolled. Luckily, there is a workaround.
<!-- more -->

## The problem
We're working on a live wallpaper called [HPSTR](https://play.google.com/store/apps/details?id=com.hpstr.wllppr), which delivers filtered images with a sensation of motion. After starting private beta, we stumbled upon a weird issue. Everything was OK while we were testing on Nexus devices, but things got strange when we tried it on a Samsung Galaxy phone. Namely, the background stood still when we flipped through the homescreen. We experienced the same on an HTC phone.

A few minutes of debugging showed that the [onOffsetsChanged()](http://developer.android.com/reference/android/service/wallpaper/WallpaperService.Engine.html#onOffsetsChanged(float, float, float, float, int, int)) method was not called properly on the Samsung device. This function is responsible for signalling the wallpaper app that the home screen pages were scrolled, as a result of the launcher calling the [setWallpaperOffsets()](http://developer.android.com/reference/android/app/WallpaperManager.html#setWallpaperOffsets(android.os.IBinder, float, float)) method. Every time the home screen is scrolled, this function should be fired, informing the current wallpaper about the offset (both in pixels and percentage) and the number of pages in the launcher (through the `xOffsetStep` parameter). This doesn't work with Samsung and HTC default launchers - after they were replaced with something else from the Play Store, the issue disappeared, the behaviour of the wallpaper was OK again. Read more [here](http://developer.samsung.com/forum/thread/onoffsetschanged-not-called-in-live-wallpaper/77/217103?boardName=GeneralB&startId=zzzzz~).

Since our app depends on motion, we needed to figure this out. Luckily, there is a best-effort solution (besides changing the launcher of course).

<p align="center">
  <img src="hpstr_live_wallpaper.gif" alt="HPSTR Live Wallpaper">
  <br/>
  this is how <a href="https://play.google.com/store/apps/details?id=com.hpstr.wllppr" target="blank_">HPSTR Live Wallpaper</a> looks
</p>

## Working without onOffsetsChanged()
The idea is to process touch events in the `onTouchEvent()` method if `onOffsetChanged()` is not working. I know, there are a number of bad things with this approach (see the end of this post) - this is a best-effort workaround.

The first task is to determine if `onOffsetsChanged()` is working OK. The launchers which don't function properly tend to call this function with an `xOffset` of 0.0 or 0.5 - this is the cornerstone of the solution. There's no guarantee that this will work with any launcher. This part is based on [this StackOverflow-thread](http://stackoverflow.com/questions/14258234/onoffsetschanged-not-called-by-touchwiz):

{% highlight java %}
private class MyEngine extends GLEngine {
  boolean isOnOffsetsChangedWorking = false;
  // ...

  @Override public void onOffsetsChanged(final float xOffset, final float yOffset, final float xOffsetStep, final float yOffsetStep, final int xPixelOffset, final int yPixelOffset) {
    if (!isOnOffsetsChangedWorking && xOffset != 0.0f && xOffset != 0.5f) {
      isOnOffsetsChangedWorking = true;
    }
    if (isOnOffsetsChangedWorking) {
      // translate the wallpaper
    }
  }
}
{% endhighlight %}

The `isOnOffsetsChangedWorking` variable tells if `onOffsetsChanged()` is working OK. If not, the app will process touch events by passing the MotionEvent instance to a GestureDetector:

{% highlight java %}
@Override public void onTouchEvent(MotionEvent event) {
  if (!isOnOffsetsChangedWorking) {
    gestureDetector.onTouchEvent(event);
  }
}
{% endhighlight %}

Set offsetChangedWorking to false in `onResume()`, to make sure that the wallpaper can detect launcher changes.

## Implementing the GestureDetector
The idea behind the implementation is to imitate the `xOffset` param of the `onOffsetChanged()` method by assembling a similar variable (let's call it the same). This way there's no need to modify the code responsible for translating the background, it can take this value as parameter.

The `xOffset` variable represents a percentage, amounting to where the current position is relative to the whole homescreen. By scrolling through the pages, the value goes from 0.0 (first page) to 1.0 (last page). This is based on the number of pages - for example, if the launcher has 3 screens, the `xOffset` for each is 0.0, 0.5 and 1.0, with fractions in between. The problem here is that if the `onOffsetsChanged()` method is out of the picture, there's no way of telling the number of pages, so it's time to improvise. You can roll with a fixed amount, or make it adjustable in the settings (this is what we ended up doing). Then it's just some basic programming, each full-screen swipe scrolls the wallpaper by [width of the screen] / [number of pages]. To prevent overscrolling, keep the value between 0 and 1.

### onScroll()
The onScroll() function is responsible for detecting if the user is scrolling the homescreen.

{% highlight java %}
@Override public boolean onScroll(final MotionEvent e1, final MotionEvent e2, final float distanceX, final float distanceY) {
  final float newXOffset = xOffset + distanceX / width / NUMBER_OF_PAGES;
  if (newXOffset > 1) {
    xOffset = 1f;
  } else if (newXOffset < 0) {
    xOffset = 0f;
  } else {
    xOffset = newXOffset;
  }
  // translate by xOffset;
  return super.onScroll(e1, e2, distanceX, distanceY);
}
{% endhighlight %}

### onFling()
The onFling() method takes care of swiping between pages. The following implementation finds the closest page based on the direction of the fling and the current `xOffset`, then smoothly scrolls the wallpaper to position using a ValueAnimator. The duration cannot be calculated properly, since it varies by the speed of the swipe movement and the launcher itself; according to our experiments, 150 ms is a good compromise.

{% highlight java %}
@Override public boolean onFling(final MotionEvent e1, final MotionEvent e2, final float velocityX, final float velocityY) {
  float endValue = velocityX > 0
  ? (xOffset - (xOffset % (1 / NUMBER_OF_PAGES)))
  : (xOffset - (xOffset % (1 / NUMBER_OF_PAGES)) + (1 / NUMBER_OF_PAGES));

  if (endValue < 0f) {
    endValue = 0f;
  } else if (endValue > 1f) {
    endValue = 1f;
  }

  final ValueAnimator compatValueAnimator = ValueAnimator.ofFloat(xOffset, endValue);
  compatValueAnimator.setDuration(150);
  compatValueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override public void onAnimationUpdate(final ValueAnimator animation) {
      xOffset = (float) animation.getAnimatedValue();
      // translate by xOffset;
      });
  compatValueAnimator.start();
  };
}
{% endhighlight %}

## Detecting double tap events
[Muzei](https://play.google.com/store/apps/details?id=net.nurik.roman.muzei), one of my favourite live wallpapers can be triggered by double-tapping the homescreen, which gave us an idea to open the settings with the same gesture. This can also be active if you choose not to support launchers without a working `onOffsetsChanged()` method.

This can be achieved easily, just override `onDoubleTap()` in the GestureDetector, and start the Activity if an event is detected. And also, since you're starting the Activity from a Service, don't forget to add the `Intent.FLAG_ACTIVITY_NEW_TASK` flag.

{% highlight java %}
@Override public boolean onDoubleTap(final MotionEvent e) {
  // start the Activity
  return true;
}
{% endhighlight %}

## Enabling infinite scrolling
Some launchers have a feature called *infinite scrolling*, meaning if you scroll further than the last page, you'll end up on the first one, and vice versa. This can mess up the background image if you scroll in the same direction for too long.

Unfortunately, there's no way of detecting this, so we added a checkbox to toggle it. In code, all you have to do is to make the 0-1 value limit conditional in the methods of the GestureDetector.

## The end
This solution is far from perfect, but then again, without an operational onOffsetsChanged() method, there's no way of determining the number of pages, the precise scrolling offset, if the home button is pressed, or if infinity scrolling is enabled. However, supporting it is fairly straightforward, a well-coded GestureDetector can take care of most issues. The sample code is available via [this gist](https://gist.github.com/andraskindler/d90029a358189f9e96e8). And if you're interested, [try the app](https://play.google.com/store/apps/details?id=com.hpstr.wllppr)!
