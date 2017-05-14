---
title:  Rxjava+Retrofit结合开发的封装技巧
date: 2016-07-31 00:13:46
categories: Android基础
tags: 
   - rxjava
   - retrofit
---


## 概述

在开发中使用`RxJava+Retrofit`的网络框架,是时下的趋势,使用起来也非常的方便.
如果能够在一定程度上进一步封装,能够大大提高我们的开发效率.接下来我们看一下比较常用的简洁处理场景.

<!-- more -->

## CreateObservable

我们都知道创建一个`Observable`可以使用`RxJava`的创建操作符,比如Create,..等等,
在[effective-rxjava](https://github.com/mgp/effective-rxjava/blob/master/items/convert-functions-to-observable.md)中作者介绍了一种将`functions`转化为`observable`的方式,感觉非常新颖,果断使用了.

通过 `defer` 和 `just` 操作符方便的将`Func0`转化为 `Observable`

```java
/**
 * @return an {@link Observable} that emits invokes {@code function} upon subscription and emits
 *         its value
 */
public static <O> Observable<O> makeObservable(final Func0<O> function) {
    checkNotNull(function);

    return Observable.defer(() -> Observable.just(function.call()));
}
```

我们都知道`defer`还有一个好处就是只有订阅时才会生效,而`just`中的参数采用了泛型化,
即我们可以将任意一个有返回值的方法都使用 `defer`,产生即时的数据流.

```java
private Object slowBlockingMethod() { ... }

public Observable<Object> newMethod() {
    return Observable.defer(() -> Observable.just(slowBlockingMethod()));
}
```

使用方式

```java
public Observable<Optional<ContentItem>> fetchContentItem(
            final ContentItemIdentifier contentItemId) {
        return subscribeOnScheduler(() -> mContentDatabase.fetchContentItem(contentItemId));
    }
    // more delegating methods follow here ...
private <T> Observable<T> subscribeOnScheduler(final Func0<T> function) {
      return ObservableUtils.makeObservable(function)
                .subscribeOn(mScheduler);
}
```


## Transformer

在[Don't break the chain: use RxJava's compose() operator](http://blog.danlew.net/2015/03/02/dont-break-the-chain/)中作者建议使用`compose`来避免打破`RxJava`的链式调用,`compose`中需要传入一个`Transformer`,我们先来看一下`Transformer`的源码.

```java
   /**
     * Transformer function used by {@link #compose}.
     * @warn more complete description needed
     */
    public interface Transformer<T, R> extends Func1<Observable<T>, Observable<R>> {
        // cover for generics insanity
    }
```

从中我们可以看到`Transformer`是一个`Func1`,其将一种类型的`Observable`转换成另一种类型的`Observable`

### 常见的`Transformer`用法

- 线程切换

在`RxJava`中最常用的某过于 `线程切换` 了,我们定义一个线程切换的`RxSchedulerTransformer`

```java
public class RxSchedulerTransformer<T> implements Observable.Transformer<T, T> {

  private static final RxSchedulerTransformer<Object> INSTANCE = new RxSchedulerTransformer<>();

  public static <T> RxSchedulerTransformer<T> instance() {
    return (RxSchedulerTransformer<T>) INSTANCE;
  }

  private RxSchedulerTransformer() {
  }

  @Override public Observable<T> call(Observable<T> tObservable) {
    return tObservable.subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
  }
}
```

- 错误处理

一般和服务器交互取回的`json`数据结构类似下面这种,当`success == true`的时候才是正确的返回值,
`success == false` 的时候,虽然不会走 `RxJava`的`OnError`,但是也是异常.我们希望的是 所有的 异常都在
`OnError`中处理.`OnNext`只关心正确的返回值即可.

```json
{
    "success": false, // 是否成功
    "code": "500",   // 响应码
    "data": ""      // 内容
}
```

根据如上`json`,我们的`Transformers`可定义为如下格式,对结果进行一些预处理,只有正常值才返回`JavaBean`

```java
public class RxHandleResultTransformer<T>
    implements Observable.Transformer<Transformers.Result<T>, T> {

  private static final RxHandleResultTransformer<Object> INSTANCE =
      new RxHandleResultTransformer<>();

  public static <T> RxHandleResultTransformer<T> instance() {
    return (RxHandleResultTransformer<T>) INSTANCE;
  }

  private RxHandleResultTransformer() {
  }

  @Override public Observable<T> call(Observable<Transformers.Result<T>> resultObservable) {
    return resultObservable.flatMap(tResult -> {
      if (tResult.success) {
        return Observable.just(tResult.data);
      } else {
        return Observable.error(new RuntimeException(tResult.code));
      }
    });
  }

  final public class Result<T> {
    boolean success;
    String code;
    T data;
  }
}
```

## 自定义操作符

和`compose`不同的是,自定义`Operator` 作用于Observable发射的单独的数据项,`compose`作用于整个流, 自定义操作符是和`lift`一起使用的,自定义操作符需要实现`Operator`

更多关于自定义操作符的介绍;
[实现自己的操作符](https://mcxiaoke.gitbooks.io/rxdocs/content/topics/Implementing-Your-Own-Operators.html)
[RxJava操作符（十）自定义操作符](http://mushuichuan.com/2016/02/05/rxjava-operator-10/#more)


## 三级缓存

其实`Retrofit`是可以处理缓存的,相关介绍:[Retrofit2.0使用总结及注意事项](http://blog.csdn.net/wbwjx/article/details/51379506),
这里需要注意的是<font color = red>Retrofit缓存需要使用@GET才生效</font>,而且是使用的文件存储.

关于`RxJava`缓存参考:[RxJava使用场景小结](http://blog.csdn.net/lzyzsd/article/details/50120801),用`RxJava`的方式来处理缓存问题


```java
Observable<String> memory = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        if (memoryCache != null) {
            subscriber.onNext(memoryCache);
        } else {
            subscriber.onCompleted();
        }
    }
});
Observable<String> disk = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
      String cachePref = rxPreferences.getString("cache").get();
      if (!TextUtils.isEmpty(cachePref)) {
        subscriber.onNext(cachePref);
      } else {
        subscriber.onCompleted();
      }
    }
  }).onErrorReturn(throwable->null);

Observable<String> network = Observable.just("network").onErrorReturn(throwable -> null);

//主要就是靠concat operator来实现
Observable.concat(memory, disk, network)
.first().subscriber(observer);
```

这里用到了`concat`与`onErrorReturn`两个操作符,关于`concat(连接操作符)`,[官方解释](http://reactivex.io/documentation/operators/concat.html)

关于`onErrorReturn(错误处理操作符)`,可以看这里:[RxJava错误处理](http://blog.csdn.net/wbwjx/article/details/51292219)

## 屏幕切换

存在如下两种问题:

1. 我们知道 屏幕切换等配置发生改变的时候,会导致`Activity`的重建,当我们订阅了某一个`Subscribtion`后,屏幕发生了改变,
及调用了`unsubscribe`方法,如何才能保证`Subscribtion`的延续呢?

2. 在`Android`中`Context`是导致很多 内存泄漏的罪魁祸首.如果我们创建的`subscribtion`持有了`Context`将会变得十分的危险,
如果`Observable`没有准时完成,就很容易导致内存泄漏.

第一种问题,可以用`RxJava`的缓存机制解决,就是`cache`(或是`replay()`),

```java
Observable<Photo> request = service.getUserPhoto(id).cache();
Subscription sub = request.subscribe(photo -> handleUserPhoto(photo));

// ...When the Activity is being recreated...
sub.unsubscribe();

// ...Once the Activity is recreated...
request.subscribe(photo -> handleUserPhoto(photo));
```

第二个问题的解决方案就是在生命周期的某个时刻取消订阅。采用`CompositeSubscription`来管理

```java
  CompositeSubscription mCompositeSubscription;

  public void addSubscription(Subscription s) {
   if (this.mCompositeSubscription == null) {
     this.mCompositeSubscription = new CompositeSubscription();
   }
   if (null != s)
   {
     this.mCompositeSubscription.add(s);
   }
 }

 public void unsubscribe() {
   if (this.mCompositeSubscription != null) {
     this.mCompositeSubscription.clear();
   }
 }

 @Override protected void onDestroy() {
    super.onDestroy();
    unsubscribe();
  }
```

## RxAndroid 中好用的方法

`RxAndroid`为我们提供了很多好用的API,如`HandlerThreadScheduler`,`AndroidObservable`,`ViewObservable`
其中`HandlerThreadScheduler`是一个可以绑定到`Handler`上的`scheduler`,
`AndroidObservable`可以绑定到 `Activity` 或 `Fragment`,方便生命周期的管理.同时可以方便的创建一个`BroadCastReceiver`的`Observable`,
`ViewObservable`用于给`View`添加绑定,如 `ViewObservable.clicks()`(监听`View`的点击事件)或者 `ViewObservable.text()`(监听`TextView`的内容变化)


```java
AndroidObservable.bindActivity(this, retrofitService.getImage(url))
    .subscribeOn(Schedulers.io())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

IntentFilter filter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
AndroidObservable.fromBroadcast(context, filter)
    .subscribe(intent -> handleConnectivityChange(intent));

ViewObservable.clicks(mCardNameEditText, false)
    .subscribe(view -> handleClick(view));
```

参考:[RxJava使用场景小结](http://blog.csdn.net/lzyzsd/article/details/50120801)
[RxJava + Retrofit 的实际应用场景](http://imxie.cc/2016/05/24/RxJava-Retrofit-%E7%9A%84%E5%AE%9E%E9%99%85%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF/?utm_source=tuicool&utm_medium=referral)
[Don't break the chain: use RxJava's compose() operator](http://blog.danlew.net/2015/03/02/dont-break-the-chain/)
[effective-rxjava](https://github.com/mgp/effective-rxjava/blob/master/items/convert-functions-to-observable.md)
[实现自己的操作符](https://mcxiaoke.gitbooks.io/rxdocs/content/topics/Implementing-Your-Own-Operators.html)
[RxJava操作符（十）自定义操作符](http://mushuichuan.com/2016/02/05/rxjava-operator-10/#more)
