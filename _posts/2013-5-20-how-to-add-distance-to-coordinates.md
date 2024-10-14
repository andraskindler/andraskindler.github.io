---
layout: post
title:  "How to add distance to coordinates"
date:   2013-5-20 09:30:00
categories: Android
excerpt: Modifying latitude and longitude with a distance.
permalink: /blog/2013/how-to-add-distance-to-coordinates/
deprecated: true
---
A few weeks ago I had to translate geocoordinates (latitude and longitude) having a given directon (aka an angle) and a distance in an Android app. Searching the web I found a few good solutions, this is my take on the problem shaped to my needs, heavily based on one of the suggestions of [this thread](http://gis.stackexchange.com/questions/2951/algorithm-for-offsetting-a-latitude-longitude-by-some-amount-of-meters).

The algorithm detailed below is a bit inaccurate. The Earth is a geoid, not a perfect globe, so the radius varies throughout the surface. In most cases, the Earth is approximated with a perfect globe (with a radius of 6,371 kilometers, accoring to [Wikipedia](http://en.wikipedia.org/wiki/Earth_radius)), but for obvious reasons this is not the most accurate solution. If precision is very important, you should consider a [more complex solution](http://williams.best.vwh.net/avform.htm#LL).

So we have an origin point (with Android I use [LatLng](http://developer.android.com/reference/com/google/android/gms/maps/model/LatLng.html)) with a given latitude and longitude, a distance (in meters) and an angle. Because the coordinates have two values in two dimensions, we have to brake the distance and the angle into two values; this can be easily done with a sinus and a cosinus.

Next, we have to calculate the latitude portion, because the latitude circles have a fix radius (the radius of the Earth). The north pole is 90º N, the south pole is 90º E, on an absolute scale latitude spans from 90º to -90º, 0º being parallel to the Equator. So we have to divide the distance with the radius of the Earth, then normalize it into the [-90, 90] interval by multiplying with 180/PI. After this, you can increase the latitude of the original point with the new value.

Moving away from 0º vertically means that a degree of longitude will get smaller, so you have to take account of the latitude value. So first we calculate the radius of circle at the given latitude, the rest is the same as above.

Knowing how latitude and longitude works help understanding the algorithm; more about that [here](http://en.wikipedia.org/wiki/Geographic_coordinate_system).

Here's the code doing all the work, offsetting the _origpoint_ with _distance_ at the given _angle_:

```
public LatLng translateCoordinates(final double distance, final LatLng origpoint, final double angle) {
        final double distanceNorth = Math.sin(angle) * distance;
        final double distanceEast = Math.cos(angle) * distance;

        final double earthRadius = 6371000;

        final double newLat = origpoint.latitude + (distanceNorth / earthRadius) * 180 / Math.PI;
        final double newLon = origpoint.longitude + (distanceEast / (earthRadius * Math.cos(newLat * 180 / Math.PI))) * 180 / Math.PI;

        return new LatLng(newLat, newLon);
}
```