---
title: android开发之contentprovider
date: 2016-06-23 23:45:36
categories: Android基础
tags: Provider
---

## 概述

ContentProvider（内容提供者）是Android中的四大组件之一。主要用于程序间数据的共享(IPC的一种).
它提供了一套完整的机制，允许一个程序访问另一个应用程序,并且保证数据的安全性.
我们知道在Android中常见的数据存储方式有sharedpreferences,文件和数据库等,但是数据的访问方式会因为存储方式的不同而不同.
而且这些数据只能在应用内使用,而ContentProvider允许在程序间实现数据的共享,并且提供好了统一了数据的访问方式.

<!-- more -->

## ContentResolver

访问内容提供器中共享的数据,需要借助通过ContentResolver,其中context.getContentResolver()方法可以获取到ContentResolver实例,
ContentResolver提供了一系列方法来对数据进行CRUD操作.

## URI

统一资源标识符,结构和HTTP形式的URL是一样的.
代表了要操作的数据,每一个ContentProvider都拥有一个公共的URI,不能重复
```
<prefix>://<authority>/<data_type>/<id>
```

其中,`<prefix>`的值永远为`content`,不可更改

`<authority>`唯一标识符,用来定位`ContentProvider`,一般为自定义Provider的全路径名,保证唯一

`<data_type>/<id>`代表资源路径,如`/person/2`代表person表中id为2的数据

## MIME

用于指定某个扩展名的文件用一种应用程序来打开,如application/pdf,text/css等,Android中的MIME类型有两种
* 多条记录
vnd.android.cursor.dir/vnd.xxx
* 单条记录
vnd.android.cursor.item/vnd.xxx

其中包含类型和子类型,/前面的类型是Android定义好的,不可更改,子类型中vnd.后面的可以随意指定.在使用Intent时，会用到MIME这玩意，根据Mimetype打开符合条件的活动。

## 自定义Contentprovider

*  继承ContentProvider,并重写其中的一些方法,

其中OnCreate()方法在Provider初始化的时候执行

getType（URI uri）用于返回某一URI对应的MIME类型,运行在Binder线程池中

还有其他一些操作CRUD操作,运行在Binder线程池中

* UriMatcher

用于冠关联Provider中的自定义URI和URI_CODE,通过addURI（）将相应的Path和Code联系起来,通过match(uri)得到相应的CODE.

```java

//为外部程序准备好一个所有地址匹配集合
static{
      uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
      uriMatcher.addURI(PROVIDER_NAME, "students", STUDENTS);
      uriMatcher.addURI(PROVIDER_NAME, "students/#", STUDENT_ID);
   }

// 取得数据的类型,此方法会在系统进行URI的MIME过滤时被调用。
@Override
 public String getType(Uri uri) {
    switch (uriMatcher.match(uri)){
       /**
       * Get all student records
       */
       case STUDENTS:
       return "vnd.android.cursor.dir/vnd.example.students";

       /**
       * Get a particular student
       */
       case STUDENT_ID:
       return "vnd.android.cursor.item/vnd.example.students";

       default:
       throw new IllegalArgumentException("Unsupported URI: " + uri);
   }
}
```

* ContentUris
用于操作Uri路径后面的ID部分

其中withAppendedId(uri, id)用于为路径加上ID部分：

```java
Uri uri = Uri.parse("content://com.example.provider.College/students")  
Uri resultUri = ContentUris.withAppendedId(uri, 10);   
//生成后的Uri为：content://com.example.provider.College/students/10  
```

parseId(uri)方法用于从路径中获取ID部分：

```java
Uri uri = Uri.parse("content://com.example.provider.College/students/10 ")  
long personid = ContentUris.parseId(uri);//获取的结果为:10
```

*  配置清单文件

```java
<provider
        android:authorities="com.bobomee.android.contentproviderdemo"
        android:name=".BookContentProvider"/>
```

## 监听ContentProvider变化


如果ContentProvider的访问者需要知道ContentProvider中的数据发生变化，可以在ContentProvider发生数据变化时调用getContentResolver().notifyChange(uri, null)来通知注册在此URI上的访问者，

```java
Uri insertUri = Uri.withAppendedPath(uri, "/" + rowId);
      Log.i(TAG, "insertUri:"+insertUri.toString());
      //发出数据变化通知(book表的数据发生变化)
      getContext().getContentResolver().notifyChange(uri, null);
      return insertUri;
```

使用Demo:[ProviderDemo](https://github.com/BoBoMEe/AndroidDev/tree/provider)

参考文章:

[ContentProvider和Uri详解](http://blog.csdn.net/xyz_lmn/article/details/7161635)

[Android - Content Providers](http://www.tutorialspoint.com/android/android_content_providers.htm)

[我是这样理解Android开发中的contentprovider的](http://www.imooc.com/article/8001)

[Android开发教程：Android数据储存之ContentProvider](https://liuzhichao.com/p/562.html)

[android ContentProvider使用详解](http://codingnow.cn/android/1078.html)
