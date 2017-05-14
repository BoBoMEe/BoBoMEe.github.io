---
title: RxJava+Retrofit错误预处理
date: 2016-08-01 11:37:49
categories: Android基础
tags:
   - rxjava
   - retrofit
---

## 概述

在使用 `RxJava+Retrofit` 结合的网络框架时,为了避免打破流式调用 和 过于繁杂的 `Subscribe` 代码
我们做了很多的尝试,比如 `自定义操作符`,`自定义Transformer`,`泛型处理`,和 自定义 `Subscriber`等


## 错误和异常举例

比如,在服务器返回数据中,假设服务器遵循规范,请求体 一般类似下面这种,

```java
{
    "success": false, // 是否成功
    "code": "500",   // 响应码
    "data": {
        // 内容,错误的时候返回""
    }      
}
```
<!-- more -->

这时候,我们我们的 `JavaBean`是这样的,

```java
public class Response<T> {
    public boolean success;
    public String code;
    public T data;
}
```
这种错误 `code`是我们和服务器约定好的,称为异常,即网络访问成功了.

但是还有一种错误 即: http 错误,比如典型的
Retrofit中的 `retrofit cache 504 Unsatisfiable Request (only-if-cached) error`错误,即网络连接错误.这两种我们都必须处理.

## 错误处理

接下来 根据[Retrofit+RxJava 优雅的处理服务器返回异常、错误](http://blog.csdn.net/jdsjlzx/article/details/51882661)的介绍来进行了一系列的处理,

首先定义如下`Transformer`转换器

```java
public static <T> Observable.Transformer<Response<T>, T> sTransformer() {

   return responseObservable -> responseObservable.map(tResponse -> {
     if (!tResponse.success) throw new RuntimeException(tResponse.code);
     return tResponse.data;
   }).onErrorResumeNext(new HttpResponseFunc<>());
 }

 public static <T> Observable.Transformer<T, T> switchSchedulers() {
    return observable -> observable.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
  }

 private static class HttpResponseFunc<T> implements Func1<Throwable, Observable<T>> {
   @Override public Observable<T> call(Throwable throwable) {
     //ExceptionEngine为处理异常的驱动器
     return Observable.error(new Throwable(throwable));
   }
 }
```

在真正调用的地方就可以这样写了

```java
public void login(View v){

  apiservice.login(name,pwd)
      .compose(Transformers.sTransformer())
      .compose(Transformers.switchSchedulers())
      .subscribe(subscriber);

}

private Subscriber<UserModel> subscriber = new Subscriber<UserModel>() {
    @Override public void onCompleted() {
      // do onCompleted
    }

    @Override public void onError(Throwable e) {
      // do on success != true;
      // do on http error
      // do on other error
    }

    @Override public void onNext(UserModel model) {
      // parse data
    }
  };
```

我们的接口是这样的

```java
public interface Api {

@FormUrlEncoded @POST("interface?login")
Observable<Response<T>> login(@Field("name") String name,@Field("pwd") String pwd);

}
```

## Transformer 和 Func 处理的区别

如上的处理,我们定义了 一个 `sTransformer` 和一个 `HttpResponseFunc`,
从中我们可以明显感觉的到`sTransformer`其实也是可以用`Func1`来定义的,

```java
public void login(View v){

  apiservice.login(name,pwd)
      .compose(Transformers.switchSchedulers())
      .map(new TransFuc<UserModel>())
      .onErrorReturn(new HttpResponseFunc<>())
      .subscribe(subscriber);

}
```

```java
public static class TransFuc<T> implements Func1<Response<T>, T> {
    @Override public T call(Response<T> tResponse) {
      if (!tResponse.success) throw new RuntimeException(tResponse.code);
      return tResponse.data;
    }
  }
```

`Transformer`作用于整个流,`Func1`是一个操作符,作用于数据项

## 不规范数据的处理

有时候服务器返回的数据并不是十分规范的,比如
正常返回是这样的
```
{
    "success": true, // 是否成功
    "status": "1",   // 状态码
    "data": {
        // 内容
    }      
}
```

错误时时这样的

```
{
    "success": false, // 是否成功
    "status": "0",   // 状态码
    "data": "371"   //错误码  
}
```
这时候如果我么用泛型处理

```java
public class Response<T> {
    public boolean success;
    public String status;
    public T data;
}
```
针对这种数据,我们的泛型该怎么写成了问题,错误的时候是`String`,正确的时候是`Bean`?
如果我们直接写成`JavaBean`,那么我们会得到一个错误,`LinkedTreeMap cannot be cast to xxx`
类型转换错误.这时候只能将泛型写成`String`来处理了,使用如下`Transformer`

```java
public static Observable.Transformer<String, UserModel> trans() {
    return stringObservable -> stringObservable.map(s -> {

      Response parseJson = GsonUtil.parseJson(s, Response.class);

      if(null != parseJson && !parseJson.success && !TextUtils.isEmpty(parseJson.status)){
        throw new RuntimeException(parseJson.status);
      }

      if (null != parseJson && PatternsUtil.isNum(parseJson.data)) {
        throw new RuntimeException(parseJson.data);
      }

      return GsonUtil.parseJson(s, UserModel.class);
    }).onErrorResumeNext(new HttpResponseFunc<>());
  }
```
使用就变成了如下这样

```java
public void login(View v){

  apiservice.login(name,pwd)
      .compose(Transformers.switchSchedulers())
       .compose(Transformers.trans())
      .subscribe(subscriber);

}
```
