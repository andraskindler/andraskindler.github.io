---
layout: post
title:  Using the JobScheduler API
date:   2015-01-31 15:15:00
categories: android
excerpt: This tutorial shows how to use the JobScheduler API, which waas introduced in Android 5.0 Lollipop, bundling tasks and defering them until certain conditions are met.
permalink: /blog/2015/jobscheduler-tutorial/
deprecated: true
---
This also means that, unless a deadline is set (yes, it is possible to do so), the developer has no control over when the system will start the job, hence this depends on factors like plugging the charger or connecting to an unmetered network. Good examples include time-insensitive and power-hungry tasks, like creating a backup of photos, syncing a music playlist or caching a movie, or downloading and uploading non user-facing data. The conditions and parameters can be provided precisely, a few examples:

* start a task when the phone is plugged in
* wait for an hour, and if the device is in idle, start a job
* sync every two hours if the device is connected to an unmetered network
* run a job if the phone is charging, but finish within twenty minutes

## Setting up the JobScheduler
The developer has to take care of two components: a service that wakes up when a job is due, and setting up the parameters of the tasks to be scheduled. The former is a class that extends [JobService](https://developer.android.com/reference/android/app/job/JobService.html), which is a subclass of the regular [Service](https://developer.android.com/reference/android/app/Service.html), with all the well-known properties, lifecycle and methods. This also means it must be declared in the manifest, however in this case the *BIND_JOB_SERVICE* permission is also required.

```
<manifest>
  // ...
  <service
    android:name=".MyJobService"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:exported="true"/>
</manifest>
```

The next step is to implement the JobService subclass. The service runs in bound mode, so there's no need to override the `onStartCommand()` function, the developer has to override only two methods, `onStartJob()` and `onStopJob()`.

Scheduled jobs will pop up in `onStartJob()`. Return true here if a separate thread is required, false otherwise. Each scheduled task is executed on a Handler running in the main thread, which blocks all future calls from the service. The solution is to only use the service to get notified when the conditions are met, and do the actual work on a separate thread. There's no need to acquire a [WakeLock](https://developer.android.com/training/scheduling/wakelock.html), JobScheduler keeps one while the service is running.

The system calls 'onStopJob()' if the current job has to be stopped before `jobFinished()` was called. This occurs when the requirements specified at schedule time are no longer met. For example, the task requires that the battery is charging, but the phone is no longer plugged in. Once this method is called, the system will release the wakelock. It is up to the developer to handle this situation besides stopping the job. Returning true notifies the JobManager that the task will be rescheduled, while returning false means the task is dropped.

If the job finished with no errors, notify the JobManager via `jobFinished(JobParameters params, boolean needsReschedule)`. Can be called from any thread. Setting the `needsReschedule` param to true notifies the JobManager that the job is rescheduled.

This example does nothing but log if one of the methods are called.

```
public class MyJobService extends JobService {

  @Override public boolean onStartJob(JobParameters params) {
    Log.i(JobService.class.getName(), "onStartJob " + params.getJobId());
    return true;
  }

  @Override public boolean onStopJob(JobParameters params) {
    Log.i(JobService.class.getName(), "onStopJob " + params.getJobId());
    return true;
  }

}
```

## Scheduling a Job
The [JobInfo](https://developer.android.com/reference/android/app/job/JobInfo.html) class specifies the parameters of a task, constructed with the [JobInfo.Builder](https://developer.android.com/reference/android/app/job/JobInfo.Builder.html). The following criteria can be set:

*   the device is plugged into an outlet;
*   the device is connected to a certain kind of network;
*   the device is in [idle mode](https://developer.android.com/reference/android/app/job/JobInfo.Builder.html#setRequiresDeviceIdle(boolean)).

A backoff-policy can be provided with `setBackoffCriteria(long initialBackoffMillis, int backoffPolicy)`, so there's no need of writing a custom failure handling mechanism; the defaults here are 30 seconds with an exponential policy, capped at 5 hours. This can be triggered by setting the `needsReschedule` parameter as true in `jobFinished()`, has to be called after a job is done. It is also possible to create recurring tasks, which run within the given interval, not more than once per period.

A deadline (maximum delay) can be provided with the `setOverrideDeadline()` method, meaning the job has to be executed until the given timeframe even if the conditions aren't met. Similarly, a minimum latency can be issued, via `setMinimumLatency()`. Note that these are only available for non-periodic tasks.

Persistence is also taken care of, scheduled jobs survive if the device is restarted, meaning there's no need to reschedule everything manually on the `BOOT_COMPLETED` broadcast anymore. Call the `setPersisted()` method with true on the Job to achieve this - without this method, the tasks are gone after a restart.

This example creates a job which should run when the device is charging, it is connected to an unmetered network, and has to start within 30 minutes.
```
private static final int MY_JOB_ID = 255;

// ...

final ComponentName componentName = new ComponentName(MainActivity.this, MyJobService.class);
final JobInfo.Builder builder = new JobInfo.Builder(MY_JOB_ID, componentName)
  .setRequiresCharging(true)
  .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
  .setOverrideDeadline(30 * 60 * 1000);
final JobInfo jobInfo = builder.build();
```

And this is how to schedule a job:
```
((JobScheduler) getSystemService(Context.JOB_SCHEDULER_SERVICE)).schedule(jobInfo);
```

## Communication between the Activity and the JobService
Due to the nature of most use cases, there's no need of communication between the app and the service, since the jobs run independently from the state of the app, often with no activities visible. Still, it might be necessary in certain occasions. The [official example](https://developer.android.com/samples/JobScheduler/src/com.example.android.jobscheduler/service/TestJobService.html) does this via keeping a reference of the Activity in the Service, but in my opinion a decoupled approach, like an eventbus, is a better fit to the problem. I would roll with GreenRobot's [EventBus](https://github.com/greenrobot/EventBus) or Square's [Otto](http://andraskindler.com/blog/2013/eventbus-in-android-an-otto-example/).

## Compatibility
The JobScheduler class is available since API level 21 only - there's no official compatibility version. However, there are a few workarounds for some cases: an app can subscribe to the `ACTION_POWER_CONNECTED` and `ACTION_BATTERY_CHANGED` broadcasts, so with some extra coding, it is possible to schedule jobs to start when the device is plugged in, or check if the phone is conencted to a WiFi network. You might want to check out the [JobSchedulerCompat](https://github.com/evant/JobSchedulerCompat) project on GitHub, which has similar capabilities as the JobScheduler itself. Also, check out Lars Vogel's great [post](http://www.vogella.com/tutorials/AndroidTaskScheduling/article.html) about other ways to schedule tasks on Android.
