---
layout: post
title:  Creating a fullscreen DialogFragment with a custom background
date:   2013-08-16 09:30:00
categories: Android
excerpt: A post about how to create a fullscreen DialogFragment, with a custom background.
permalink: /blog/2013/creating-a-fullscreen-dialogfragment-with-a-custom-background/
deprecated: true
---
The default Dialogs (or rather the [DialogFragments](http://developer.android.com/reference/android/app/DialogFragment.html)) look pretty good in Android since Honeycomb, but the default look-and-feel doesn't go well together with all app designs. Not to mention sometimes you need a fully different layout, a custom background color, or a semitransparent background with no grey dimming at all. We're talking about Android, where (almost) everything is possible, so there is a solution for this problem as well.

Customizing a DialogFragment is a quite easy task. Create a class extending DialogFragment, and override the _onCreateDialog_ method, which is responsible for creating the Dialog. Instantiate a Dialog, then make the following calls on the instance:

*   _requestWindowFeature(Window.FEATURE_NO_TITLE)_ - removes the dialog's title
*   _getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT)_ forces the dialog to occupy the whole screen estate
*   _setBackgroundDrawable()_ - sets the dimming drawable. This can be transparent, a solid color, a gradient or a custom drawable (even an image)
*   create the layout and set it to the dialog using _setContentView()_

These four steps give you an empty fullscreen layout to play with, with the option to add a semi-transparent background, revealing the underlying activity.

You can find more on using and customizing the DialogFragment at the [official developer site](http://developer.android.com/reference/android/app/DialogFragment.html). Here is a short sample code illustrating a DialogFragment with a yellow background and an empty layout.

```
public class CustomDialogFragment extends DialogFragment {

    public CustomDialogFragment() {}

    @Override
    public Dialog onCreateDialog(final Bundle savedInstanceState) {

        // the content
        final RelativeLayout root = new RelativeLayout(getActivity());
        root.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));

        // creating the fullscreen dialog
        final Dialog dialog = new Dialog(getActivity());
        dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
        dialog.setContentView(root);
        dialog.getWindow().setBackgroundDrawable(new ColorDrawable(Color.YELLOW));
        dialog.getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);

        return dialog;
    }

}
```