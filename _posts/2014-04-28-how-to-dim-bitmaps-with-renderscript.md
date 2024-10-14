---
layout: post
title:  How to dim bitmaps with RenderScript
date:   2014-4-28 15:06:00
categories: Android
excerpt: A post about how to use Renderscript for image manipulation.
permalink: /blog/2014/how-to-dim-bitmaps-with-renderscript/
deprecated: true
---
RenderScript was introduced with Honeycomb, aka API level 11. It is an API designed for performance intensive computations by executing native code on the device. These segments run either on the CPU or the GPU - this is not for the developer to decide, it is determined in runtime. I won't go into more detail since there are a lot of comprehensive articles out there, for more info check out the documentation at the end of the post. This short tutorial is about how to write RenderScript code for simple image manipulation tasks.

Image manipulation problems can be highly parallelized, because (at least in this example) the same chunk of code is executed independently on each and every pixel. The runtime takes care of parallelizing and scheduling the most efficient way for the current device.

A lot of useful built-in scripts (called [intrinsics](http://android-developers.blogspot.com/2013/08/renderscript-intrinsics.html)) are available in out of the box, but some use cases require a different solution. The following section will show you how to create a script for dimming a bitmap by a specific percentage. It is more like a guideline on how to implement these sort of functions - the basics are given, writing your own requires _only_ an algorithm.

## Setting up the environment

Using RenderScript requires some extra effort:

*   Add the following lines to the _defaultConfig_ section of the _build.gradle_ script:

```
android {
    ....
    defaultConfig {
        renderscriptTargetApi 19
        renderscriptSupportMode true
        ...
    }
    ...
}
```

*   Create a _rs_ directory in the _src_ folder for the RenderScript files.

And you're ready to roll!

## Setting up the RenderScript code

The next step is writing the RenderScript code. The following lines dim every pixel of a bitmap by a given percentage. The processing takes about 110 milliseconds on a Nexus 5, using an image of 1920 x 1080 pixel resolution.

```
#pragma version(1)
#pragma rs java_package_name(com.andraskindler.sandbox)
#pragma rs_fp_relaxed

float dimmingValue = .4f;

uchar4 __attribute__((kernel)) dim(uchar4 in)
{
    uchar4 out = in;
    out.r = in.r * dimmingValue;
    out.g = in.g * dimmingValue;
    out.b = in.b * dimmingValue;
    return out;
}
```

Let's take a look at the RenderScript-specific elements!

*   The _#pragma version(1)_ defines the version of the RS kernel script (currently only a value of 1 is supported).
*   The _#pragma rs java_package_name_ declares the package name of the Java classes that use the script.
*   The _#pragma rs_fp_relaxed_ enables NEON optimizations on supported SoC-s.
*   The ___attribute__((kernel))_ between the return value and the name of the _dim_ function indicates that the method is a kernel function, invokable from Java code.
*   The parameter and the return value are both of uchar4 types, representing the color of a pixel.
*   The input parameter is automatically filled in.
*   Global variables (like _dimmingValue_ in the example) automatically generate setters.

The rest is pretty much self-explanatory. In the example the color of each pixel is dimmed by the amount; any other per-pixel based effect can be achieved by changing these calculations.

## Using the script from java code

The reflected class will be available after completing the RenderScript file and building the project. The steps include initializing the RenderScript and the script objects, creating the necessary Allocations (these serve as inputs for the kernel), setting the required options, running and destroying the script. And of course since we're talking about a potentially long-running operation, put everything into an async wrapper.

```
final RenderScript renderScript = RenderScript.create(MainActivity.this);
final ScriptC_dim script = new ScriptC_dim(renderScript);
script.set_dimmingValue(0.6f);
final Allocation alloc1 = Allocation.createFromBitmap(renderScript, source, Allocation.MipmapControl.MIPMAP_NONE, Allocation.USAGE_SCRIPT);
final Allocation alloc2 = Allocation.createTyped(renderScript, alloc1.getType());
script.forEach_dim(alloc1, alloc2);
alloc2.copyTo(source);
```

## Learning RenderScript

Creating a basic script is easy, but RenderScript is much more capable than this simple example. There are a lot of great resources available to learn more about RenderScript, here are my favourites.

*   [Introducing Renderscript](http://android-developers.blogspot.com/2011/02/introducing-renderscript.html) and [Renderscript Part 2](http://android-developers.blogspot.com/2011/03/renderscript.html) on the official Android dev blog
*   [RenderScript](http://developer.android.com/guide/topics/renderscript/index.html) on the official documentation
*   [High Performance Applications with RenderScript](https://developers.google.com/events/io/sessions/331954522) by Jason Sams and Tim Murray at Google I/O 2014