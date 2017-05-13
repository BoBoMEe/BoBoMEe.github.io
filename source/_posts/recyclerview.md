---
title: Android RecyclerView 使用全解析
date: 2016-06-26 14:13:36
tags: recyclerview
---

## 概述

`RecyclerView`出来已经很长时间了,关于其的介绍也非常的多.作为`ListView`的升级版,它更加强大和灵活.
可以轻松的实现各种布局和动画,见其名,知其意.`RecyclerView`用于在有限窗口中展示大量数据集合的可复用的视图.
这里主要梳理一下`Recyclerview`的常用方法,示例Demo:[BoBoMEe/AndroidDev](https://github.com/BoBoMEe/AndroidDev/tree/blog/blogcodes/app/src/main/java/com/bobomee/blogdemos/recycler)

<!-- more -->

## 相关类

`RecyclerView`的灵活性,主要体现在各个类的职责分离上,
`RecyclerView`不负责子View的展示与布局,只负责子view的回收与复用.其他的比如,子View的布局,装饰及动画都交由其他类来处理.

`RecyclerView`相关类及其作用

| 类       | 用途           |
| ------------- |:-------------:|
|RecyclerView.ViewHolder|装载子view数据的容器|
|RecyclerView.LayoutManager|布局管理器,负责子View的布局|
|RecyclerView.Adapter|适配器,负责处理数据与绑定视图|
|RecyclerView.ItemDecoration|装饰子view,如分割线、偏移等|
|RecyclerView.ItemAnimator|子view改变时的动画|


## 基本使用

`RecyclerView`最简单的使用需要设置`LayoutManager`和`Adapter`,其他的都是可选项.简单使用流程可分为以下几步.

* 引入

```java
 compile 'com.android.support:recyclerview-v7:23.4.0'
```

```xml
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="@dimen/item_margin"
    android:clipToPadding="false"/>
```

* 设置`Adapter`和`LayoutManager`

```java
RecyclerView mRecyclerView = (RecyclerView)findViewById(R.id.recycler_view);

mLinearLayoutManager = new LinearLayoutManager(this);
mRecyclerView.setLayoutManager(mLinearLayoutManager);

mRecyclerStringAdapter = new RecyclerStringAdapter(Datas.getDatas());
mRecyclerView.setAdapter(mRecyclerStringAdapter);
```
其中`LinearLayoutManager`代表线性显示.默认是方向为`VERTICAL`,`LayoutManager`抽象类有三个官方实现版本,分别是`LinearLayoutManager`(线性布局),`GridLayoutManager`(网格布局),`StaggeredGridLayoutManager`(瀑布流布局).

* `Adapter`

不同于`ListView`的`Adapter`,`Recyclerview`的适配器强制使用`ViewHolder`模式.如下一个简单的`Adapter`实现

```java
public class RecyclerStringAdapter extends RecyclerView.Adapter<RecyclerViewholder> {

  List<String> datas;
  public RecyclerStringAdapter(List<String> datas) {
    this.datas = datas;
  }

  @Override public void onBindViewHolder(RecyclerViewholder holder, int position) {
    holder.textView.setText(datas.get(position));
  }

  @Override public RecyclerViewholder onCreateViewHolder(ViewGroup parent, int viewType) {

   View view =
       LayoutInflater.from(parent.getContext()).inflate(R.layout.recycler_item, parent, false);

   return new RecyclerViewholder(view);
 }

 @Override public int getItemCount() {
    return datas.size();
  }
}

# RecyclerViewholder
public class RecyclerViewholder extends RecyclerView.ViewHolder {

  public TextView textView;

  public RecyclerViewholder(View itemView) {
    super(itemView);
    textView = (TextView) itemView.findViewById(R.id.text);
  }
}
```

## 其他用法

* `LayoutManager`布局切换

我们知道`LayoutManager`主要负责子View的布局.通过设置不同的`LayoutManager`即可实现展现方式的改变.
如果不设置`LayoutManager`,数据将不会展现出来.

```java
//gridview的列数为3,orientation = VERTICAL
mGridLayoutManager = new GridLayoutManager(this, 3);
mRecyclerView.setLayoutManager(mGridLayoutManager);
```

```java
// 瀑布流的列数为3,orientation = VERTICAL
mStaggeredGridLayoutManager =
        new StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL);
mRecyclerView.setLayoutManager(mStaggeredGridLayoutManager);
```

```java
int orientation = mLinearLayoutManager.getOrientation();
mLinearLayoutManager.setOrientation(orientation == LinearLayoutManager.VERTICAL ? LinearLayoutManager.HORIZONTAL: LinearLayoutManager.VERTICAL);
```
* `LayoutManager`常用方法

```java
//返回当前第一个可见 Item 的 position
findFirstVisibleItemPosition()
//返回当前第一个完全可见 Item 的 position
findFirstCompletelyVisibleItemPosition()
//返回当前最后一个可见 Item 的 position
findLastVisibleItemPosition()
//返回当前最后一个完全可见 Item 的 position.
findLastCompletelyVisibleItemPosition()  
```

* `ItemAnimator`Item动画

用于设置子view的添加、移动、删除的相关动画效果.官方默认提供了一个`DefaultItemAnimator`,如果我们没有设置`ItemAnimator`,则默认就是`DefaultItemAnimator`,`Recyclerview.Adapter`提供了`notifyDataSetChanged()`、`notifyItemInserted(index)`、 `notifyItemRemoved(position)`、 `notifyItemChanged(position)`方法来刷新`Recyclerview`.这里可以看一下[wasabeef/recyclerview-animators](https://github.com/wasabeef/recyclerview-animators):实现了多种`Recyclerview`的Item动画

* `ItemDecoration`Item装饰

通过`mRecyclerView.addItemDecoration(new MarginDecoration(this));`来给Item添加装饰,

`ItemDecoration`可以叠加,`Recyclerview`展示的时候会遍历所有的`ItemDecoration`并调用其绘制方法来对Item进行装饰.

`ItemDecoration`提供了三个方法用于定制装饰器：

```java
// 在Item条目绘制之前调用,如果不是指offset则被Item的内容所遮挡
public void onDraw(Canvas c, RecyclerView parent);
// 在Item条目绘制之后调用,浮于Item之上
public void onDrawOver(Canvas c, RecyclerView parent);
// 计算并设置Item偏移量
public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent);
```
## Recyclerview多种布局

在`ListView`中为了实现多种布局,我们需要重写`Adapter`中的 `getItemViewType`和`getViewTypeCount`方法
而在`Recyclerview`中保留了`getItemViewType`方法,同时我们可以在`onCreateViewHolder`中根据不同的`ViewType`来创建不同的`Holder`
这里可以看一下[hongyangAndroid/baseAdapter](https://github.com/hongyangAndroid/baseAdapter):万能的Adapter for ListView,RecyclerView,GridView
其中关于`HeaderView`及`FooterwView`的处理,同样也是通过`ViewType`来实现的,
在网格布局中,依靠了`GridLayoutManager#setSpanSizeLookup`方法来设置每个Item占多少个span,
在瀑布流不居中,通过给`StaggeredGridLayoutManager.LayoutParams`设置`setFullSpan(true);`来占满一行.

## ItemTouchHelper

用于`Recyclerview`的触摸辅助操作,比如选中,移动,删除等事件的监听.
这里可以看一下: [可拖拽的RecyclerView](http://www.devtf.cn/?p=795),通过跟踪代码我们可以看到,`ItemTouchHelper`内部
主要是通过`GestureDetectorCompat`来处理触摸事件,并通过给`Recyclerview`设置`OnItemTouchListener`来实现效果
但是遗憾的是其中只 实现了`onLongPress`的监听.但是为我们设置`Recyclerview`的Item监听提供了一种新的思路

```java
//ItemTouchHelper#ItemTouchHelperGestureListener
private class ItemTouchHelperGestureListener extends GestureDetector.SimpleOnGestureListener {

    @Override
    public boolean onDown(MotionEvent e) {
        return true;
    }

    @Override
    public void onLongPress(MotionEvent e) {
      //...
    }
}
```
通过`ItemTouchHelper`实现`Recyclerview`的`Click`,`LongClick`,`Select`,`Swipe`,`Drag`等效果,
主要通过一个叫做`ItemTouchHelper.Callback`的类,示例:[RvItemTouchHelperCallback](https://github.com/BoBoMEe/AndroidDev/blob/blog/blogcodes/app/src/main/java/com/bobomee/blogdemos/recycler/helper/RvItemTouchHelperCallback.java)

## Recyclerview 选中模式

遗憾的是`Recyclerview`并不像`Listview`那样提供`selectMode`,但是在我们最初不适用`selectMode`的时候,我们是否采用过一种通过刷新`Adapter`来实现的方式呢
在`Recyclerview`中要实现选中模式,我们当然也要从`Adapter`下手,这里可以看一下[Multi-Selection Mode for RecyclerView](http://shawn-duan.com/android/recyclerview/2016/06/14/Multi-Selection-Mode-for-RecyclerView/)
其中的核心代码

```java
 private SparseBooleanArray mSelectedItems;

 public void switchSelectedState(int position) {
       if (mSelectedItems.get(position)) {       // item has been selected, de-select it.
           mSelectedItems.delete(position);
       } else {
           mSelectedItems.put(position, true);
       }
       notifyItemChanged(position);
   }

   public void clearSelectedState() {
       List<Integer> selection = getSelectedItems();
       mSelectedItems.clear();
       for (Integer i : selection) {
           notifyItemChanged(position);
       }
   }
```
可以看到作者也是通过在`Adapter`中维护一个选中的集合来实现`Recyclerview`的选中模式的,

## 总结

`Recyclerview`不仅在灵活度和扩展性上都较`AbsListView`好,在项目中也得到了大量的使用,也还有大量的未知方法等着去实践,
比如`Recyclerview`的`Scroll`操作,`RecycledViewPool`.`Recyclerview`的上下拉刷新,
自定义`LayoutManager`、`itemDecoration`、`itemAnimator`等待

参考:
[ Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587)
[可拖拽的RecyclerView](http://www.devtf.cn/?p=795)
[Multi-Selection Mode for RecyclerView](http://shawn-duan.com/android/recyclerview/2016/06/14/Multi-Selection-Mode-for-RecyclerView/)
[Using the RecyclerView](https://guides.codepath.com/android/using-the-recyclerview)
