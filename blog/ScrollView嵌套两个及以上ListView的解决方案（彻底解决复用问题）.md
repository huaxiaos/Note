---
title: ScrollView嵌套两个及以上ListView的解决方案（彻底解决复用问题）
date: 2017-08-01 18:44:09
tags: [Android,ListView,ScrollView] 
categories: Android
---

# 前言

开发过程中总会遇到Scrollview中嵌套listview的需求，嵌套1个listview网上有很多成熟的解决方案，如listview item设置定高、listview高度自适应、重新测量listview高度等等。但上述几种方案，都有一个缺点是无法利用到listview的复用特性，因为上述方法的计算过程中，所有的item项都会执行一遍getView方法。

除此之外，如果需要在Scrollview中嵌套两个listview，三个listview呢？如何处理滑动冲突问题？本文将给出解决方案。

值得一提的是，本文只是单纯的探讨Scrollview中嵌套两个listview的解决方案。有的朋友会说，完全可以使用RecycleView，或者自定义LinearLayout来实现这个需求，但就是另外一个问题了，和本文无关。

# 效果展示

![image](http://7xuvhf.com1.z0.glb.clouddn.com/multilistviewandscrollview.gif)

# 面临的问题

Scrollview中嵌套两个listview，会面临以下几个问题

* listview1滑动到底部时，如何让listview2也跟着滑动上来？
* 复用问题如何解决，listview1的数据什么时候加载，listview2的数据什么时候加载？
* 滑动事件是否会与列表点击事件相冲突？

# 两个ListView与Scrollview间的滑动冲突问题

## 解决方案

首先需要定义一个概念：混合状态，也就是两个ListView同时出现在Scrollview中。

那么我们可以发现，总共有三种状态：混合状态、ListView1充满ScrollView（如果数据量足够大）、ListView2充满ScrollView（如果数据量足够大）。所以解决方案本身要紧密围绕这三种状态。解决滑动冲突的核心思想是，同一时间段只能使ListView和ScrollView其中的一个滑动，而屏蔽另一个的滑动。我们先假设ListView1和ListView2的高度都足够大，解决方案如下：

* ListView1需要监听其滑动到底部的事件，如果滑动到底部，则屏蔽ListView1的滑动事件，开启ScrollView的滑动事件

```
mLv1.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                switch (scrollState) {
                    case AbsListView.OnScrollListener.SCROLL_STATE_IDLE:
                        // lv1滚动到底部，屏蔽lv1的滚动，开启sv的滚动，状态设置为混合状态
                        // 这段代码不能省略，否则在lv1滚动到底部后，无法开启sv的滚动，进而无法加载lv2
                        if (view.getLastVisiblePosition() == (view.getCount() - 1)) {
                            setMixStatus(true);
                        }
                        break;
                }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                // Empty
            }
        });
```

* ScrollView的滑动过程中，始终屏蔽其他ListView的滑动

```
mSv.setScrollViewListener(new MultiScrollView.ScrollViewListener() {
            @Override
            public void onScrollChanged(ScrollView scrollView, int x, int y, int oldx, int oldy) {
                
                ...

                // lv1与lv2的混合状态下，屏蔽listview的滚动
                mSv.forbidChildScroll();
            }
        });
```

* 在ScrollView滑动过程中，如果监测到ListView2充满了屏幕，则屏蔽ScrollView的滑动，开启ListView2的滑动

```
mSv.setScrollViewListener(new MultiScrollView.ScrollViewListener() {
            @Override
            public void onScrollChanged(ScrollView scrollView, int x, int y, int oldx, int oldy) {
                int[] svLocation = new int[2];
                mSv.getLocationOnScreen(svLocation);

                ...

                int[] lv2Location = new int[2];
                mLv2.getLocationOnScreen(lv2Location);

                if (lv2Location[1] == svLocation[1]) {
                    mLv2.forbidParentScroll();
                    mSv.allowChildScroll();
                    mLv2.setMix(false);
                    return;
                }

                ...
            }
        });
```

* 反向滑动时（从ListView2滑动至ListView1），需要对ListView2滑动至顶部的事件做监听，如果ListView2滑动至顶部，则屏蔽ListView2的滑动，开启ScrollView的滑动

```
mLv2.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                switch (scrollState) {
                    case AbsListView.OnScrollListener.SCROLL_STATE_IDLE:
                        // lv2滚动到顶部，屏蔽sv的滚动，开启lv2的滚动
                        if (view.getFirstVisiblePosition() == 0) {
                            mLv2.allowParentScroll();
                            mSv.forbidChildScroll();
                        }
                        break;
                }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                if (visibleItemCount == totalItemCount) {
                    // 说明listview2的高度是小于父控件高度的
                    mLv2.setMaxHeight(setListViewHeightBasedOnChildren(mLv2, 0));
                    mLv2.setMax(true);
                }
            }
        });
```

## 其他情况的分析

上面只是在两个ListView的数据量足够大的情况下分析的，那么如果ListView1数据量小，不能充满父控件，又或者ListView2数据量小呢？又或者两个ListView的数据量都小呢？

### ListView1数据量小，ListView2数据量大

这种情况下，首先状态肯定是混合状态，那么问题来了，何时设置混合状态呢？因为从上面的解决方案中可以看出，混合状态的设置是在ListView1滑动至底部的时候设置的，但在混合状态下，ListView的滑动是被禁止的。那么，我们需要在ListView1首次加载的时候完成状态设置。

```
root.post(new Runnable() {
            @Override
            public void run() {
                int lv1Height = setListViewHeightBasedOnChildren(mLv1, root.getHeight());

                if (lv1Height < root.getHeight()) {
                    
                    ...
                    
                    // 设置混合状态
                    setMixStatus(true);
                }
            }
        });
```

### ListView1和ListView2数据量都小

和上面的情况几乎一致，不再赘述

### ListView1的数据量足够大，ListView2的数据量小

这种情况下，由于ListView2数据量小，所以在ListView2滑动至底部后，反向滑动时，一直是处于混合状态，所以在上述方案中不会存在问题，不需要特殊处理。

# ListView显示不全的问题

这个问题众所周知，解决方案也都很成熟，多数是需要指定ListView的高度，或者重新测量ListView的高度等等。那么这些方案中，都有一个确定是无法使用ListView的复用特性，所有的item项都是一次性加载出来的。如何解决复用问题呢？请看下面的分析

# ListView复用问题

如果数据量小，或者getView方法中的操作足够简单，那这个问题可以不予理睬。但如果不是上述情况，我们就需要注意ListView的复用问题了，如果一次性把ListView1和ListView2的各item全部加载出来，这显然是不能接受的。解决这个问题的核心思想是，即用即取，显示多少加载多少。详情如下：

## ListView1

对应两种情形，一种是ListView1足够大，能够撑满父控件；另一种是足够小，不能撑满父控件。可以看出ListView1的最大显示边界是父控件的高度，最小显示边界是其本身实际的大小。从这里可以看出，获取父控件的高度是需要首先完成的，否则无法进行后续的判断。获取父控件高度的代码如下：

```
root.post(new Runnable() {
            @Override
            public void run() {
                
                ...
                
                root.getHeight();
                
                ...
            }
        });
```

之后需要对ListView1重新测量，此时涉及到ListView测量高度的方法

## ListView测量方法的改进

传统的测量方法是遍历全部的item项，然后进行计算，我们这里做一个小改动，方法接收一个最大高度的值，如果达到了最大高度，则停止遍历。这样就能保证其余的item项不会被提前执行getView方法

```
public static int setListViewHeightBasedOnChildren(ListView listView, int maxHeight) {
    ListAdapter listAdapter = listView.getAdapter();
    if (listAdapter == null) {
        return 0;
    }

    int totalHeight = 0;
    int dividerHeight = listView.getDividerHeight();
    for (int i = 0; i < listAdapter.getCount(); i++) {
        View listItem = listAdapter.getView(i, null, listView);
        listItem.measure(0, 0);
        totalHeight = totalHeight + listItem.getMeasuredHeight() + dividerHeight;

        if (totalHeight > maxHeight && maxHeight > 0) {
            ViewGroup.LayoutParams params = listView.getLayoutParams();
            params.height = maxHeight;
            listView.setLayoutParams(params);
            return maxHeight;
        }
    }

    ViewGroup.LayoutParams params = listView.getLayoutParams();
    params.height = totalHeight;
    listView.setLayoutParams(params);

    return totalHeight;
}
```

## ListView2

ListView2的加载是重头戏，解决方案是根据ScrollView的滑动，动态改变ListView2的高度。这里就会涉及到两个问题，其一是如果ListView2足够小，不能撑满父控件，如何处理？我们捕获ListView2达到最大高度这一状态，代码如下：

```
        mLv2.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                ...
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                if (visibleItemCount == totalItemCount) {
                    // 说明listview2的高度是小于父控件高度的
                    mLv2.setMaxHeight(setListViewHeightBasedOnChildren(mLv2, 0));
                    mLv2.setMax(true);
                }
            }
        });
```

其二，如果ListView2的高度过大也需要做限制，不能大于父控件的高度，否则多余的item项的复用问题又会重现。其三，动态设置ListView2的高度，有时候会使最后一项显示不全，此时我们需要做一次“纠正”操作，即在ScrollView滑动时，如果发现ListView2已经达到了最大状态，但实际高度却不是最大高度时，重新设置一次高度。

ScrollView的代码如下

```
mSv.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                int action = event.getAction();

                switch (action) {
                    case MotionEvent.ACTION_DOWN:
                        mLastY = (int) event.getRawY();
                        break;

                    case MotionEvent.ACTION_MOVE:
                        int dy = (int) event.getRawY() - mLastY;

                        if (mMix) {
                            if (mLv2.isMax()) {
                                if (mLv2.getHeight() < mLv2.getMaxHeight()) {
                                    // 用于防止出现最后一项显示不全的状况
                                    ViewGroup.LayoutParams layoutParams = mLv2.getLayoutParams();
                                    layoutParams.height = mLv2.getMaxHeight();
                                    mLv2.setLayoutParams(layoutParams);
                                }
                            } else if (dy < 0) {
                                // 只有在lv2高度扩大时才执行
                                ViewGroup.LayoutParams layoutParams = mLv2.getLayoutParams();
                                int dex = mLv2.getHeight() - dy;
                                if (dex > root.getHeight()) {
                                    dex = root.getHeight();
                                }
                                layoutParams.height = dex;
                                mLv2.setLayoutParams(layoutParams);
                            }
                        }

                        mLastY = (int) event.getRawY();
                        break;

                    ...
                }

                return false;
            }
        });
```

# 混合状态下的点击事件失效问题

如果代码写到此为止，滑动冲突和复用的问题已经全部解决了。但是反复测试后会发现，此时存在一个严重的点击事件失效问题：当处于滑动状态时，由于开启了ScrollView的滑动，屏蔽了ListView的滑动，这样就导致了ListView的点击事件被ScrollView截获了，ListView的各项是无法点击的。

解决方案是：在ScrollView的ACTION_UP动作中恢复ListView的点击事件，但同时，混合状态下ListView只处理点击事件，忽略移动事件。

ScrollView关键代码

```
mSv.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                int action = event.getAction();

                switch (action) {
                    
                    ...

                    case MotionEvent.ACTION_UP:
                        mSv.allowChildScroll();
                        break;
                }

                return false;
            }
        });
```

ListView关键代码

```
@Override
public boolean dispatchTouchEvent(MotionEvent event) {
    if (mForbidParentScroll) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mScrollView.requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_UP:
                mScrollView.requestDisallowInterceptTouchEvent(false);
                break;
        }
    } else {
        mScrollView.requestDisallowInterceptTouchEvent(false);
        if (mMix) {
            if (event.getAction() == MotionEvent.ACTION_MOVE) {
                return true;
            }
        }
    }
    return super.dispatchTouchEvent(event);
}
```

# 总结

至此，ScrollView嵌套两个ListView的诸多问题已经全部解决，两个及以上的ListView的实现思路大同小异。
最后再补充一句，本文只是“就事论事”，单纯的对ScrollView嵌套ListView做分析，至于这个需求本身是否需要采用这种方式，这种方式本身是否契合需求本身，不在本文的讨论范围内。

# GitHub

[项目地址](https://github.com/huaxiaos/MultiListViewAndScrollView)

# 参考资料

* http://www.jianshu.com/p/c1fc19795244
* http://blog.csdn.net/chaihuasong/article/details/17499799
* http://blog.csdn.net/zhang_duo/article/details/20942703
* http://blog.qiji.tech/archives/12674
* http://blog.csdn.net/u010687392/article/details/47809295
* https://stackoverflow.com/questions/7999211/how-to-iterate-through-sparsearray
* http://www.eoeandroid.com/blog-1088111-13840.html
