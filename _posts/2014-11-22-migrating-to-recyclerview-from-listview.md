---
layout: post
title:  Migrating to RecyclerView from ListView
date:   2014-11-22 12:00:00
categories: Android development
excerpt: A step-by-step guide about how to change a ListView, a GridView or ExpandableListView to a RecyclerView.
permalink: /blog/2014/migrating-to-recyclerview-from-listview/
deprecated: true
---
Google introduced [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) as a part of the support library as the next-gen ListView. It's new, efficient and highly customisable, no wonder everyone wants to make the switch. This step-by-step guide will show you how to migrate from an existing ListView (or GridView / StaggeredGridView / ExpandableListView), replacing the implementation with a RecyclerView.

### RecyclerView vs. LayoutManager

The official documentation states that the RecyclerView is _a flexible view for providing a limited window into a large data set_. So the RecyclerView itself is only a container made to render items from an adapter, specifying the orientation and other properties of the children is up to the developer. This is where the [LayoutManager](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) comes to play, this class is responsible for positioning and measuring the child views, and also for recycling them. Choosing the LayoutManager affects the behaviour of the RecyclerView - for example if you want to mimic a ListView, roll with a LinearLayoutManager.

Built-in LayoutManagers are provided for the basic use cases, but of course it is possible to implement a custom one. Each have two orientations, for vertical or horizontal scrolling. This means a horizontally scrolling adapter-based view (like a _TwoWayView_) can also be replaced effortlessly. After adding an _android.support.v7.widget.RecyclerView_ to the layout, the next step is to figure out which LayoutManager to use:

*   ListView or ExpandableListView - [LinearLayoutManager](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html)
*   GridView - [GridLayoutManager](https://developer.android.com/reference/android/support/v7/widget/GridLayoutManager.html)
*   StaggeredGridView - [StaggeredGridLayoutManager](https://developer.android.com/reference/android/support/v7/widget/StaggeredGridLayoutManager.html)
Tip: the setHasFixedSize(true) method on the RecyclerView instance improves performance if the adapter content doesn't change the size of the RecyclerView itself.

### BaseAdapter vs RecyclerView.Adapter

Instead of a BaseAdapter or a BaseExpandableAdapter, you'll have to subclass the RecyclerView.Adapter subclass. Both have a similar approach, but the former enforces the use of ViewHolders, and the mandatory functions are also a bit different. Since we're moving from a Base(Expandable)Adapter to a [RecyclerView.Adapter](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html), let's take a look at the required methods and how to move them!

The method _getCount()_ is equivalent to _getItemCount()_, required in both adapters.

The function getItemID(int position) mandatory in a BaseAdapter is not required in a RecyclerView.Adapter, but it's possible to override it optionally. No modification required here.

There's no equivalent to BaseAdapter's getItem(int position) in RecyclerView.Adapter. If it is necessary, a public method with the same parameters and body works like a charm.

In a Base(Expandable)Adapter, the getView() method is responsible for recycling, setting up the ViewHolder if necessary, and setting data for the current element. This is broken into two functions in a RecyclerView.Adapter: onCreateViewHolder() creates the ViewHolder, while onBindViewHolder() attaches data to the view. The good news is there's no need to manually take care of recycling anymore.

**The ViewHolder pattern**  
There's a good chance that everyone concerned about performance already uses the ViewHolder pattern, but with RecyclerView, you don't have a choice not to. It's a mystery why did Google wait so long to enforce the use of the VH pattern, since it has been around for ages.

### Handling clicks

There's no way of attaching an OnItemClickListener or an OnLongClickListener to a RecyclerView, the adapter has to take care of clicks. This can be done by for example setting an OnClickListener to each item, or by setting one for each ViewHolder, with a properly set position indicator, or other kind of data as class variable.

```
public static class ViewHolder extends RecyclerView.ViewHolder {
    public View view;
    public Item currentItem;

    public ViewHolder(View v) {
        super(v);
        view = v;
        view.setOnClickListener(new View.OnClickListener() {
            @Override public void onClick(View v) {
                //...
            }
        });
    }
}

@Override public void onBindViewHolder(ViewHolder viewHolder, int i) {
    // bind data to the VH
    //...
    // set the current item
    viewHolder.currentItem = items.get(i);
}
```

It's also possible to set an onTouchListener on the RecyclerView itself, and figure out which item is selected with findChildViewUnder(float x, float y).

The code should be working now. In the following sections, we'll review the most common _extras_ used on adapter-based views.

### ViewType handling

Using item types with a RecyclerView is almost the same as before. The _getItemViewType(int position_) function takes care of mapping positions to types, but there's no need of implementing a _getViewTypeCount()_ function. The adapter will pass the proper ViewHolder in _onBindViewHolder()_, however you'll have to take care of creating all used types in _onCreateViewHolder()_, where the second parameter indicates the ViewType.

### Dividers and padding

These require a bit more work than calling the _setDivider()_ and _setDividerHeight()_ functions, you'll have to use a RecyclerView.ItemDecoration() subclass. If you only want padding between the items, just override the getItemOffsets() method by increasing the proper coordinates of the outRect param. For example for a vertical ListView:

```
public class DividerItemDecoration extends RecyclerView.ItemDecoration {
    private int padding;

    public DividerItemDecoration(int padding) {
        this.padding = padding;
    }

    @Override public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        outRect.left += padding;
        outRect.right += padding;
    }
}
```

A horizontal would mean the left and right parameters, while using a grid would means all four sides.

Using drawables as dividers is also possible.

### Headers

Doing headers is easy, they can be substituted with an extra ViewType.

**Sticky headers**  
This is available as a [third-party library](https://github.com/timehop/sticky-headers-recyclerview) by Timehop.

### SwipeRefreshLayout

The [SwipeRefreshLayout](https://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html) is a good thing to have if the adapter can be refreshed. The good news is that it can be used as usual, check out [this tutorial](http://andraskindler.com/2013/playing-around-with-the-swiperefreshlayout/) for more.

### Item animations

ListViews don't support item animations out of the box, but they can be achieved with quite a bit of hacking. The good news is that this became much easier with the [RecyclerView.ItemAnimator](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemAnimator.html), with methods for adding, removing, changing and moving items.

### Other properties

Managing the scrollbars, the overscrollmode and fading the edges have the same methods as before. Highlighting items on click can be done the same way as before, with a custom background drawable.

### Wrapping up

The transition to a RecyclerView requires some effort, but it's worth it - the result is much more flexible and powerful.