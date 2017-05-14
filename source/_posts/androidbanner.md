---
title: Android无限轮播Banner的实现
date: 2016-05-12 20:02:00
categories: 自定义控件
tags: banner
---



## 概述
--------

应用首页的广告轮播`Banner`，一般都会使用`ViewPager`来实现，但是`ViewPager` 没有轮播效果。
现成有这么几种实现方案:

1.使用`Integer.MAX_VALUE` ,理论上很难达到边界。<br/>
2.装饰`adapter`方式[imbryk/LoopingViewPager@Github](https://github.com/imbryk/LoopingViewPager)。<br/>
3.扩展`ViewPager`方式[yanzm/LoopViewPager](https://github.com/yanzm/LoopViewPager)。

<!-- more -->

## 相关库
------
轮播效果其实就是一个定时任务，可以用定时器，`Handler`等。

[Trinea/Android Auto Scroll ViewPager@Github](https://github.com/Trinea/android-auto-scroll-view-pager)

此库通过`handler`定时发送消息实现，用的比较多，使用中感觉切换并非无缝。主要代码如下：

```java
    /**
     * scroll only once
     */
    public void scrollOnce() {
        PagerAdapter adapter = getAdapter();
        int currentItem = getCurrentItem();
        int totalCount;
        if (adapter == null || (totalCount = adapter.getCount()) <= 1) {
            return;
        }

        int nextItem = (direction == LEFT) ? --currentItem : ++currentItem;
        if (nextItem < 0) {
            if (isCycle) {
                setCurrentItem(totalCount - 1, isBorderAnimation);
            }
        } else if (nextItem == totalCount) {
            if (isCycle) {
                setCurrentItem(0, isBorderAnimation);
            }
        } else {
            setCurrentItem(nextItem, true);
        }
    }
```
这里是在最后或第一页直接`setCurrentItem()`,因此，并非无缝轮播效果。

和本文开头的几个控件整合一下，即可方便 的实现我们想要无缝 的效果。


## 基本实现
----

### 1.修改[Trinea/Android Auto Scroll ViewPager@Github](https://github.com/Trinea/android-auto-scroll-view-pager)
改变首页和最后一页的跳转设置。

```java
    /**
     * scroll only once
     */
    public void scrollOnce() {
        PagerAdapter adapter = getAdapter();
        int currentItem = getCurrentItem();
        if (adapter == null || adapter.getCount() <= 1) {
            return;
        }

        int nextItem = (direction == LEFT) ? --currentItem : ++currentItem;
        setCurrentItem(nextItem, true);
    }
```

### 2.实现方式选择
第一种方式其实主要改变了`adapter`中的`index`，大体如下：

```java
@Override  
public int getCount() {  
  return Integer.MAX_VALUE;  
}
@Override  
public Object instantiateItem(ViewGroup container, int position) {  
  position = position % listviews.size()  ;
  //....       
}  

```
只是理论上的无限，需要自己处理`position`。

第二种装饰了`adapter`，在前后各添加一个`Item`，当到了我们添加的临界`item`的时候，立即设置到正确的 `position` ，主要方法如下：

`LoopPagerAdapterWrapper`：
```java
int toRealPosition(int position) {
        int realCount = getRealCount();
        if (realCount == 0)
            return 0;
        int realPosition = (position-1) % realCount;
        if (realPosition < 0)
            realPosition += realCount;

        return realPosition;
    }
```
这里，内部封装`position`映射，不需要自己处理`position`，和正常`ViewPager`使用方式一样。

第三种，重写扩展`ViewPager` 继承 `ViewGroup`，使其`item`无限。需要自己处理`position`，而且 和 [JakeWharton/ViewPagerIndicator@Github](https://github.com/JakeWharton/ViewPagerIndicator)搭配使用起来不是很方便，`setViewPager`方法需要额外处理。

综上，使用第二种方式 扩展性和 易用性都比较好。

## Indicator
----

[JakeWharton/ViewPagerIndicator@Github](https://github.com/JakeWharton/ViewPagerIndicator)
这是一个比较全面的`indicator`，如果我们只需要简单的`CircleIndicator`，可以抽取来使用。

优秀相关开源库：

[THEONE10211024/CircleIndicator@Github](https://github.com/THEONE10211024/CircleIndicator):一个轻量的圆形指示器

[ongakuer/CircleIndicator@Github](https://github.com/ongakuer/CircleIndicator)：才用Drawable写的圆形指示器

也可以看看本人站在巨人肩膀上完成的[DrawableIndicator@Github](https://github.com/BoBoMEe/DrawableIndicator)：可以实时偏移，切换动画，drawable资源。

效果图：

![DrawableIndicator](https://github.com/BoBoMEe/DrawableIndicator/raw/master/gif.gif)


## 测试
---
my_banner.xml：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.bobomee.android.scrollloopviewpager.autoscrollviewpager.AutoScrollViewPager
        android:id="@+id/picslooper1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <com.bobomee.android.drawableindicator.widget.DrawableIndicator
        android:id="@+id/pageIndexor1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        app:indicator_height="15dp"
        app:indicator_margin="15dp"
        app:indicator_select_src="@drawable/select_drawable"
        app:indicator_unselect_src="@drawable/unselect_drawable"
        app:indicator_width="15dp" />
</RelativeLayout>
```

 MainActivity:

``` java
  AutoScrollViewPager viewPager = (AutoScrollViewPager) findViewById(R.id.picslooper1);
        viewPager.setFocusable(true);
        viewPager.setAdapter(new FragmentStateAdapter(getSupportFragmentManager()));

        BaseIndicator pageIndex = (BaseIndicator) findViewById(R.id.pageIndexor1);
        pageIndex.setViewPager(viewPager);

        viewPager.startAutoScroll();
```

最终效果图
![AutoScrollLoopViewPager](https://github.com/BoBoMEe/AutoScrollLoopViewPager/raw/master/screenshot/shot.gif)

源码:
[AutoScrollLoopViewPager@Github](https://github.com/BoBoMEe/AutoScrollLoopViewPager)
