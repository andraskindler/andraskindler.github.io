---
layout: post
title:  Creating View reflections
date:   2014-3-11 15:42:00
categories: Android
excerpt: This tutorial is about creating the faded reflection of any View subclass in Android apps.
permalink: /blog/2014/creating-view-reflections/
deprecated: true
---
Although it looks like the age of skeuomorphism and over-designed apps is finally over, you shouldn't forget everything you've learned in that era. One of the effects is reflection, which can be good and bad at the same time: overusing can make your UI look cheesy and bloated, but sometimes it's the right thing to use to boost the UX. This tutorial will show you how to create the faded reflection of any View subclass.

The procedure can be broken into three large steps:

1.  creating the piece of UI you want to be reflected, but not adding it to the root view
2.  layouting and rendering the view to capture it as a bitmap
3.  adding reflection to the bitmap and drawing it on the screen

## Assembling the UI

The first step is fairly simple. Do everything as usual, just don't add the root view into your layout hierarchy. This example features a simple TextView with white textcolor and red background, but the effect is not limited to a single View, you can even use complex layouts (ViewGroups).

```
final TextView textView = new TextView(this);
textView.setText("test");
textView.setLayoutParams(new FrameLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT, Gravity.CENTER));
textView.setTextColor(Color.WHITE);
textView.setTextSize(40);
textView.setBackgroundColor(Color.RED);
textView.setPadding(100, 20, 100, 20);
```

## Layouting and rendering

Layouting consists of two steps: measuring the view and layouting it. Measuring is done with the [measure()](http://developer.android.com/reference/android/view/View.html#measure(int, int)) method, which is responsible for determining the size requirements for the view and its subviews. After a _measure()_ call, the [getMeasuredWidth()](http://developer.android.com/reference/android/view/View.html#getMeasuredWidth()) and [getMeasuredHeight()](http://developer.android.com/reference/android/view/View.html#getMeasuredHeight()) calls will return the proper size, which you'll use in a [layout()](http://developer.android.com/reference/android/view/View.html#layout(int, int, int, int)) call. The _layout()_ method forces all views to position their children according to the size parameters.

```
final int measureSpec = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED);
textView.measure(measureSpec, measureSpec);
textView.layout(0, 0, textView.getMeasuredWidth(), textView.getMeasuredHeight());
```

Next, capture the view as a bitmap using the [draw()](http://developer.android.com/reference/android/view/View.html#draw(android.graphics.Canvas)) method, which renders the onto a Canvas.

```
final Bitmap b = Bitmap.createBitmap(textView.getWidth(), textView.getHeight(), Bitmap.Config.ARGB_8888);
final Canvas c = new Canvas(b);
textView.draw(c);
```

## Creating the reflection

The resulting reflection can be divided into four steps, meaning creating a reflection is a four pass process. You'll need to determine the size of the reflection (it might be a good idea to do this as a percentage of the original view) and the height of the gap, both should be in pixels. You'll also need a new bitmap (with a size of the original bitmap, the reflection and the gap combined), and its Canvas.

```
final int reflectionHeight = original.getHeight() * percentage;
Bitmap bitmapWithReflection = Bitmap.createBitmap(original.getWidth(), (original.getHeight() + reflectionHeight + gap), Bitmap.Config.ARGB_8888);
Canvas canvas = new Canvas(bitmapWithReflection);
```

First, draw the original bitmap:

```
canvas.drawBitmap(original, 0, 0, null);
```

Then add a gap, represented by a transparent Paint:

```
final Paint transparentPaint = new Paint();
transparentPaint.setARGB(0, 255, 255, 255);
canvas.drawRect(0, original.getHeight(), original.getWidth(), original.getHeight() + gap, transparentPaint);
```

Next stop: the reflection. We’ll use a modified identity matrix to flip the image along the X axis, then draw it on the canvas:

```
final Matrix matrix = new Matrix();
matrix.preScale(1, -1);
canvas.drawBitmap(Bitmap.createBitmap(original, 0, original.getHeight() - reflectionHeight, original.getWidth(), reflectionHeight, matrix, false), 0, original.getHeight() + gap, null);
```

Last but not least, add a finishing touch by fading out the reflection with a properly placed [LinearGradient](http://developer.android.com/reference/android/graphics/LinearGradient.html):

```
final Paint fadePaint = new Paint();
fadePaint.setShader(new LinearGradient(0, original.getHeight(), 0, original.getHeight() + reflectionHeight + gap, 0x70ffffff, 0x00ffffff, Shader.TileMode.CLAMP));
fadePaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
canvas.drawRect(0, original.getHeight(), original.getWidth(), bitmapWithReflection.getHeight() + gap, fadePaint);
```

And that's it, you got yourself a bitmap showing the contents of your view with a simple yet fancy reflection. Display it either in an ImageView, as a View background or in the _onDraw()_ method of a View subclass. Protip: call [recycle()](http://developer.android.com/reference/android/graphics/Bitmap.html#recycle()) on the original to aid memory management.