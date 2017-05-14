---
title: 快速开发之Android Orm总结
date: 2016-04-20 22:02:00
categories: Android基础
tags: orm
---

## 概述

在android平台上，常用的数据保存方式有Preference，文件和数据库，针对于数据库操作，
一般都会采用orm框架来解决问题，Android中使用比较广泛的Orm有LiteOrm,OrmLite，SugarORM，GreenDAO，ActiveAndroid，Realm等

AndroidOrm使用的小demo:[AndroidOrm](https://github.com/BoBoMEe/AndroidDev/tree/orm)

<!-- more -->
## LiteOrm

这款和`OrmLite`不是一个东西,项目地址:https://github.com/litesuits/android-lite-orm,是一个专注于数据库操作的Orm

提供了非常方便的 `级联操作` 和 `单独操作` 切换,`LiteOrm对象` 需要保持全局单例, 其提供了非常丰富的注解(Annotation)

CRUD操作也十分方便,应对数据库的升级也不需要做任何操作.只需修改`Model`即可,因为`LiteOrm`自动探测技术

详细使用:[Android 快速开发系列之数据库篇（LiteOrm](http://www.jianshu.com/p/0d72226ef434)

和其他Orm的对比:[Android数据库框架：greenDAO vs LiteOrm](http://www.jianshu.com/p/330bbd3b0e68)


## OrmLite
`OrmLite`是java平台的Orm框架,可以在任何使用Java的地方,支持JDBC，Spring等，用到了很多注解 (Annotation),
官网地址http://ormlite.com/

启用方式: 第一步,编写JavaBean

```java
@DatabaseTable(tableName = "users")
public class User {
    @DatabaseField(id = true)
    private String username;
    @DatabaseField
    private String password;

    public User() {
        // ORMLite needs a no-arg constructor
    }
    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    // Implementing getter and setter methods
}
```

第二步: 编写`DatabaseHelper`,继承`OrmLiteSqliteOpenHelper`,具体查看Sample,也十分简单

第三步: 通过`helper.getUserDao()`来进行CRUD操作

create(user),deleteById(id),update(user),queryForAll()等.

详细使用:[Android 数据库框架ormlite 使用精要](http://www.jianshu.com/p/05782b598cf0)


## SugarORM
SugarORM是android 专用orm，提供save(), delete() 和 find()/findById()方法，官网地址:https://github.com/satyan/sugar


启用方式：第一步:配置meta-data
```java
<meta-data android:name="DATABASE" android:value="my_database.db" />
<meta-data android:name="VERSION" android:value="1" />
<meta-data android:name="QUERY_LOG" android:value="true" />
<meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="com.my-domain" />
```

第二步: JavaBean,需要继承SugarRecord,或者使用@Table注解
```java
public class User extends SugarRecord<User> {
    String username;
    String password;
    int age;
    @Ignore
    String bio; //this will be ignored by SugarORM

    public User() { }

    public User(String username, String password,int age){
        this.username = username;
        this.password = password;
        this.age = age;
    }
}
```

CRUD: 通过save(), delete() 和 find()方法完成
其中update操作.

```java
Book book = Book.findById(Book.class, 1);
book.title = "updated title here"; // modify the values
book.edition = "3rd edition";
book.save(); // updates the previous entry with new values.
```
上手很简单,更多用法参考官方文档:http://satyan.github.io/sugar/getting-started.html

## GreenDAO
GreenDAO 号称Android上最快的orm框架，每秒数千条记录。官网地址:http://greenrobot.org/greendao/

作为一款老字号的Orm框架,网上的介绍也比较多了,需要建立Java工程来生成DAO、Master、Session 等对象
如果数据库升级,需要重新创建DaoGenerator,来生成代码.

其后的CRUD操作都是依赖于Dao来完成.和其他Orm没有本质的区别.

详细使用见官方文档:http://greenrobot.org/greendao/documentation/


## ActiveAndroid
ActiveAndroid项目地址:https://github.com/pardom/ActiveAndroid

使用ActiveAndroid，需要在AndroidManifest中添加meta信息,用注解生成模型
```java
<meta-data android:name="AA_DB_NAME" android:value="my_database.db" />
<meta-data android:name="AA_DB_VERSION" android:value="1" />
```
然后需要在Application中使用 ActiveAndroid.initialize(this);初始化
```java
public class MyApplication extends SomeLibraryApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        ActiveAndroid.initialize(this);
    }
}
```
CRUD:
通过类Select(查询),Delete(删),bean.save()增改,其wiki介绍也十分详细

## Realm
一个不依赖于SqlLite的ORM库,底层使用c++ 实现的,是最新流行的Orm框架,而且是跨平台的.官网地址:https://realm.io

启用步骤:第一步,使用gradle来引入realm-android
第二步,新建JavaBean实现RealmModel接口.
第三步,CRUD操作,官网介绍也十分详细.

普通操作:

```java
// Obtain a Realm instance
Realm realm = Realm.getDefaultInstance();

realm.beginTransaction();

//... add or update objects here ...
User user = realm.createObject(User.class);

realm.commitTransaction();
//or cancel
//realm.cancelTransaction();
```
relam支持链式查询和Auto-Updating Results

```java
RealmResults<User> teenagers = realm.where(User.class).between("age", 13, 20).findAll();
User firstJohn = teenagers.where().equalTo("name", "John").findFirst();
```
需要注意的是,在使用完毕后需要关闭 Realm 实例。

```java
public class MyThread extends Thread {

    private Realm realm;

    public void run() {
        Looper.prepare();
        try {
            realm = Realm.getDefaultInstance();
            //... Setup the handlers using the Realm instance ...
            Lopper.loop();
        } finally {
            if (realm != null) {
                realm.close();
            }
        }
    }
}
```
使用Relam还有一个好处就是可以结合当下比较流行的Rxjava和Retrofit来使用,
其原生提供了很好的支持,截止目前已经出了1.0.0版本,但是仍有诸多限制,及JavaBean的不自由.
