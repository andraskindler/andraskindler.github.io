---
layout: post
title:  "Multitasking on steroids: managing documents in the Overview"
date:   2014-11-29 19:22:27
categories: android
excerpt: A tutorial about the new multitasking features of Android 5.0. This guide will show you how to customize your overview entry.
permalink: /blog/2014/managing-documents-in-the-overview/
deprecated: true
---
**Opening multiple Documents and Tasks**

Starting an activity with a new document can be specified in the launching intent, by adding the _FLAG_ACTIVITY_NEW_DOCUMENT_ flag. Also, by setting the _FLAG_ACTIVITY_MULTIPLE_TASK_ flag the system creates a new task as well, with the Activity as root. Check out the snippet below:
```
final Intent intent = new Intent(this, ThirdActivity.class)
    .setFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT)
    .setFlags(Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
startActivity(intent);
```

This can also be done in the manifest, by setting the _documentLaunchMode_ attribute of the activity to either _intoExisting_ (opens a new document, but reuses the current task), or _always_ (opens a new document and starts a new task). The number of documents in the overview can be limited manually in the manifest with the _maxRecents_ attribute. However, this cannot exceed 50, or 25 for devices with low RAM. Also, this number is for the whole application, not for individual activities.

```
<activity android:maxRecents="8" .../>
```

**Customizing the appearance of Documents**

The looks of each card can be specified programmatically, which is a good way of expressing the branding of the app. The title, the icon and the color of the header background can be defining an _ActivityManager.TaskDescription_; this should be done in the _onCreate()_ function in the Activity.

```
protected void onCreate(Bundle savedInstanceState) {
    //...
    final ActivityManager.TaskDescription taskDescription = new ActivityManager.TaskDescription("1st Activity", BitmapFactory.decodeResource(getResources(), R.drawable.ic_icon), Color.GREEN); setTaskDescription(taskDescription);
    //...
}
```

This is available for sites as well - while the document belonging to Chrome has a grey header by default, webpages can override this setting to have a color of their own. The color setting also effects the status bar. A few sites have already adopted this feature, check out for example [AndroidPolice](http://androidpolice.com) in Chrome. Just add the following tag to the _head_ of the page (_content_ specifies the color).

```
<meta name="theme-color" content="#ababab"/>
```

**The defaults**

The document header is customized by default even when nothing is set manually. The icon comes from the _android:icon_ attribute, the title is the _android:label_ attribute, and the color of the header is equal to the _colorPrimary_ value of the theme definition.

**Persistence across reboots**

The documents in the overview can be reopened across reboots. This can be specified by setting the _persistableMode_ attribute of the _activity_ tag. Three options are available (more info [here](https://developer.android.com/reference/android/R.attr.html#persistableMode)):

  * _persistRootOnly_ \- the default setting. The task persists after a restart, the launch intent is used.
  * _persistNever_ \- the task will not persist after a reboot.
  * _persistAcrossReboots_ \- the task and the activity persists after a reboot.

**Managing the tasks of an application**
The [AppTask](https://developer.android.com/reference/android/app/ActivityManager.AppTask.html) class was also introduced in API level 21. This class can be used to list and manage the current tasks of the application.

```
ActivityManager activityManager = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
for (ActivityManager.AppTask appTask : activityManager.getAppTasks()) {
    // ...
}
```

The new multitasking is more than just a gimmick. A proper setup can result in a better user experience with multiple documents, and modifying the appearance of the header can mean a more recognisable brand.
