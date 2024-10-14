---
layout: post
title: Tinting drawables
date: 2015-08-20 12:00:00
categories: android
excerpt: A short post about how to tint drawables and bitmaps to match the current theme.
permalink: /blog/2015/tinting_drawables/
deprecated: true
---
When designing the *themes* section of Ready Contacts app, we wanted to come up with a way of changing not just the basic color aspects of the app, but the icons and other drawables as well. Using the usual method, this would mean creating a png for each color, and switching between them based on the selected theme - a huge overhead in code, and a also an increase in apk size. We also wanted to make adding colors in the future easy, without having to create new resources each and every time.

## DrawableCompat
Google introduced the [DrawableCompat](https://developer.android.com/reference/android/support/v4/graphics/drawable/DrawableCompat.html) class in the v4 support library, which introduces tinting capabilities for pre-Lollipop devices. It has a full-fledged API, even supports tint lists, and mirroring for RTL layouts, but it was a bit heavyweight for our use case, and also you have to wrap the current Drawable with the `wrap()` method.

## Introducing the TintedBitmapDrawable
So we came up with our own implementation, a lightweight [BitmapDrawable](http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html) subclass called **TintedBitmapDrawable**, with an overridden `draw()` method taking care of tinting using a [LightingColorFilter](http://developer.android.com/reference/android/graphics/LightingColorFilter.html). It only consists of three functions, so it doesn't mean a significant increase in the method count. The tint color can be specified in one of the two extra constructors or via the `setTint()` method.

```
public final class TintedBitmapDrawable extends BitmapDrawable {
  private int tint;
  private int alpha;

  public TintedBitmapDrawable(final Resources res, final Bitmap bitmap, final int tint) {
    super(res, bitmap);
    this.tint = tint;
    this.alpha = Color.alpha(tint);
  }

  public TintedBitmapDrawable(final Resources res, final int resId, final int tint) {
    super(res, BitmapFactory.decodeResource(res, resId));
    this.tint = tint;
    this.alpha = Color.alpha(tint);
  }

  public void setTint(final int tint) {
    this.tint = tint;
    this.alpha = Color.alpha(tint);
  }

  @Override public void draw(final Canvas canvas) {
    final Paint paint = getPaint();
    if (paint.getColorFilter() == null) {
      paint.setColorFilter(new LightingColorFilter(tint, 0));
      paint.setAlpha(alpha);
    }
    super.draw(canvas);
  }
}
```

And this is how to use it:

```
tintedDrawable = new TintedBitmapDrawable(resources, R.drawable.ic_arrow_back_white_24dp, Color.GREEN);
```

## Benefits and tips

* Works for images with white and transparent colors.
* No need for multiple drawables of the same icon when an app supports themes, meaning the apk occupies less space.
* Perfect fit with Google's [material icon pack](https://www.google.com/design/icons/), just download the white .png version and tint it accordingly.
* Also an epic fit with the [Palette library](https://developer.android.com/reference/android/support/v7/graphics/Palette.html).
* If used with list items, cache the drawables.
* Also available in the ToolBar, if assembled programmatically instead of using a menu.xml.
* Create a StateListDrawable with multiple states of different colored drawables from the same image, thus reducing apk size.
