---
layout: post
title: A look at the Palette class
date: 2014-7-4 15:06:00
categories: Android
excerpt: A look on the capabilities and performance of the Palette class from the support library.
permalink: /blog/2014/a-look-at-the-palette-class/
deprecated: true
---
Back in February, Chris Banes' two excellent blog posts (color matching [part one](http://chris.banes.me/2014/02/18/colour-matching/) and [part two](https://chris.banes.me/2014/03/10/colour-matching-pt-2/)) demonstrated how can we extract the dominant colors from a bitmap. This can be a powerful asset to the user experience of your app: think about dynamically colored UI elements, or a filter based on the dominant color of your bitmap. The Palette class from the new Support Library announced at Google I/O does the same. 

Currently there is no official documentation, you can refer to [this article](http://chris.banes.me/2014/07/04/palette-preview/) for some info on the features. The class itself can extract the dominant colors from a bitmap, including these prominent colors:

*   vibrant
*   vibrant dark
*   vibrant light
*   muted
*   muted dark
*   muted light

## Including Palette from the Support Library

The first step is including the library in your project. You can do this with the following Gradle dependency:

`compile 'com.android.support:palette-v7:+'`

## Palette in action

A Palette instance can be generated with static object methods, either with the _generate()_ or the _generateAsync()_ function. The difference between them is that with _generate()_ the developer has to take care of the wrapping it into an async thread, since this is a potentially long running operation which shouldn't run on the UI thread. On the other side, the _generateAsync()_ takes a _PaletteAsyncListener_ instance as parameter, which has an _onGenerated()_ callback, called when the palette generation is completed. The following example uses the latter.

```
Palette.generateAsync(BitmapFactory.decodeResource(getResources(), R.drawable.test),
  new Palette.PaletteAsyncListener() {
    @Override public void onGenerated(Palette palette) {
      // do something with the colors
    }
});
```

Note that if you want to extract all six prominent colors, you'll have to have a parameter larger than or equal to six. You can also specify how many colors should be generated - by default, this number is 15. The resulting Palette instance contains PaletteItems, which represent colors, with their RGB and HSL representation and population. The _getPalette()_ method returns the whole list, however this is not ordered by population. You can also retrieve the prominent colors, discussed above. While bitmap operations usually take some time, the generate method appeared almost instantaneous.

In conclusion, the Palette is an awesome tool to improve UX with dynamic, content-based colors. The performance impact is small (at least on a relatively high-end device), and the benefits look great, especially with the new theme and animation possibilities in the L dev preview.