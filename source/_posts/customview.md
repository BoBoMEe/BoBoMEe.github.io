---
title: Android自定义控件细节
date: 2016-06-25 11:48:32
tags: android
---

## 概述

在开发过程中,当系统提供的控件不能满足我们的需求的时候,通常都会采用自定义控件来完成,自定义控件的一般流程:

attrs.xml-->onMeasure()-->onLayout(ViewGroup)-->onDraw()-->onTouchEvent()-->onInterceptTouchEvent(ViewGroup);

其中带有ViewGroup的是自定义ViewGroup需要用到的方法.
<!-- more -->

## 自定义属性

自定义属性,一般定义用户可自定义的一些特性,关于自定义属性的声明和获取,网上的讲解也比较多了,之前关于自定义属性的总结[Android 自定义属性解析](http://blog.csdn.net/wbwjx/article/details/50583812),主要步骤:
在values目录下声明相关属性,在自定义控件的构造方法中获取这些属性,应用到自定义控件中.

## onMeasure

onMeasure即测量,测量自定义控件的大小.测量的值是由两部分决定的[Mode和Size],Mode和Size都封装到了一个叫做MeasureSpec的类中;

扩展阅读[View的MeasureSpec确定过程](http://ghui.me/post/2015/10/view-measure/)

```java
private int measureWidth(int widthMeasureSpec) {
   int result;
   int mode = MeasureSpec.getMode(widthMeasureSpec);
   int size = MeasureSpec.getSize(widthMeasureSpec);

   if (mode == MeasureSpec.EXACTLY) {
     // 如果是精确的,则直接返回结果,
     // 这种模式通常是我们给View设置了一个值,如:android:layout_width="300dp"
     // 或者 android:layout_width="match_parent",屏幕的宽度
     result = size;
   } else {
     // MeasureSpec.UNSPECIFIED View的大小没有限制.
     // 通常在ScrollView或者ListView中出现
     result = getNeedWidth() + getPaddingLeft() + getPaddingRight();
     if (mode == MeasureSpec.AT_MOST) {
       // 至多不能超过某个值,一般出现在我们设置android:layout_width="wrap_content" 时
       // 设置wrap_content,则View的尺寸由自身内容决定,但最大不能超过父控件的尺寸
       // 这里 即和父控件传入的值 进行比较,取小(最多不能超过size)
       result = Math.max(result, size);
     }
   }
   return result;
 }
```

在得到result 之后需要调用`setMeasuredDimension(width,height);`
`requestLayout();`会触发重新测量onMeasure()



## onLayout

onLayout即布局,决定了ViewGroup中子View的显示位置. View中提供了空实现的onLayout,但自定义View一般不需要管,ViewGroup中的onLayout是抽象的,即必须实现.

扩展阅读[Android自定义控件布局之OnLayout](http://coderrobin.com/2015/01/26/Android%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6%E5%B8%83%E5%B1%80%E4%B9%8BOnLayout/)

```java
@Override protected void onLayout(boolean changed, int l, int t, int r, int b) {

    final  int childCount = getChildCount();
    for (int i = 0; i < childCount; i++) {
      final View child = getChildAt(i);
      if (child.getVisibility() == GONE){
        continue; //如果child为GONE,则不需要布局
      }

      int left = caculateChildLeft();//计算ChildView左上角x坐标
      int top = caculateChildTop();//计算ChildView左上角y坐标

      child.layout(left,top,left+childWidth,top+childHeight);//childWidth 和 childHeight 是child的宽和高

    }
  }
```

尽可能将onMeasure中的一些与Measure无关的操作移动到onLayout中,因为onMeasure在一次自定义控件流程中,可能会执行多次,而onLayout只执行一次.
`requestLayout();`会触发重新布局onLayout()


## onDraw
onDraw决定自定义控件的样子,在onDraw()我们只负责View内容的绘制,</br>
背景系统已经帮我们绘制好了, 这里主要借助的是Canvas这个类.</br>
同时在onDraw方法中,我们一般会使用translate,rotate,scale,skew等来实现canvas的变换 </br>
如果使用了变换操作,我们需要使用save()和restore()来保存于恢复canvas的状态</br>
如果是自定义ViewGroup一般我们也不需要管onDraw这个方法,draw由子控件自己完成

扩展阅读 [自定义 View 必备－Draw 源码分析及其实践](http://gold.xitu.io/entry/57465c88c4c971005d6e4422)

[为什么自定义ViewGroup ondraw方法不会被调用](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1014/1765.html)

```java
@Override protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   // allocations per draw cycle.
   int paddingLeft = getPaddingLeft();
   int paddingTop = getPaddingTop();
   int paddingRight = getPaddingRight();
   int paddingBottom = getPaddingBottom();

   int contentWidth = getWidth() - paddingLeft - paddingRight;
   int contentHeight = getHeight() - paddingTop - paddingBottom;

   // Draw the text.
   canvas.drawText(mExampleString, paddingLeft + (contentWidth - mTextWidth) / 2,
       paddingTop + (contentHeight + mTextHeight) / 2, mTextPaint);

   // Draw the example drawable on top of the text.
   if (mExampleDrawable != null) {
     mExampleDrawable.setBounds(paddingLeft, paddingTop, paddingLeft + contentWidth,
         paddingTop + contentHeight);
     mExampleDrawable.draw(canvas);
   }
 }
```

` postInvalidate();`和`invalidate();`会触发重新绘制onDraw().

## onTouchEvent

它是在view中定义的一个方法,用于处理传递到view 的手势事件,
其中涉及到几个常量;
* MotionEvent.ACTION_DOWN: 按下的时候,做一些初始化,赋值操作.
* MotionEvent.ACTION_MOVE: 移动的事件
* MotionEvent.ACTION_UP: 手指抬起时,做一些释放资源,重置变量的操作
* MotionEvent.ACTION_CANCEL:手势释放操作,释放资源,重置变量
* MotionEvent.ACTION_POINTER_DOWN: 多点触控操作,需要借助event.getActionIndex(),getPointerId(actionIndex)等一系列方法
* MotionEvent.ACTION_POINTER_UP:多点触控非最后一个点被释放时执行,这里需要注意的是activePointer才会起作用

```java
private void onSecondaryPointerUp(MotionEvent ev) {
       //...
       final int pointerId = ev.getPointerId(pointerIndex);
       if (pointerId == mActivePointerId) {
           // This was our active pointer going up. Choose a new
           // active pointer and adjust accordingly.
           // TODO: Make this decision more intelligent.
           final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
           //...
           mActivePointerId = ev.getPointerId(newPointerIndex);
       }
   }
```

```java
// 通知父控件不要拦截事件
final ViewParent parent = getParent();
                    if (parent != null) {
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
```

## onInterceptTouchEvent

ViewGroup的方法,表示是否拦截事件,其中传递的参数也是MotionEvent,
同样包含上面那些常量,我们可以在不同时机调用不同的方法来完成逻辑

扩展阅读 [图解 Android 事件分发机制](http://www.jianshu.com/p/e99b5e8bd67b)

## 其他

Configuration: 述设备的配置信息(locale,scaling,..):[官网Configuration介绍](https://developer.android.com/reference/android/content/res/Configuration.html)

ViewConfiguration: 包含了设置UI的超时、大小和距离 de 方法和标准的常量用来,[官网ViewConfiguration介绍](https://developer.android.com/reference/android/view/ViewConfiguration.html)

扩展阅读: [Viewconfiguration和configuration类使用](http://souly.cn/%E6%8A%80%E6%9C%AF%E5%8D%9A%E6%96%87/2015/11/26/ViewConfiguration%E5%92%8CConfiguration%E7%B1%BB%E4%BD%BF%E7%94%A8/)

手势操作GestureDetector: [Android - Gestures Tutorial](http://www.tutorialspoint.com/android/android_gestures.htm)

ViewDragHelper:事件处理,[Android应用ViewDragHelper详解及部分源码浅析](http://blog.csdn.net/yanbober/article/details/50419059)

VelocityTracker: 触摸速率跟踪 [官网介绍Tracking Movement](https://developer.android.com/training/gestures/movement.html)

Scroller: 平滑滚动帮助类 [ Android应用开发Scroller详解及源码浅析](http://blog.csdn.net/yanbober/article/details/49904715)


onSaveInstanceState和onRestoreInstanceState: 存储和恢复自定义控件的状态,参考[onSaveInstanceState & onRestoreInstanceState](http://stormzhang.com/android/2014/09/22/onsaveinstancestate-and-onrestoreinstancestate/)
