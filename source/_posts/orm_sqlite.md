---
title: Android 之数据库的使用
date: 2016-04-20 22:02:00
tags: orm
---

##概述

在android平台上，常用的数据保存方式有Preference，文件和数据库，针对于数据库操作，
市面上常用的几款orm，主要有OrmLite，SugarORM，GreenDAO，Active Android，Realm

<!-- more -->

## OrmLite
OrmLite是java平台的Orm框架,用到了很多注解 (Annotation)


## SugarORM
SugarORM是android 专用orm，提供save(), delete() 和 find()方法，启用方式：在AndroidManifest中配置即可。
```java
<meta-data android:name="DATABASE" android:value="my_database.db" />
<meta-data android:name="VERSION" android:value="1" />
<meta-data android:name="QUERY_LOG" android:value="true" />
<meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="com.my-domain" />
```

## GreenDAO
GreenDAO 号称Android上最快的orm框架，每秒数千条记录。


## Active Android
使用Active Android，需要导入jar包，并在AndroidManifest中添加meta信息,用注解生成模型
```java
<meta-data android:name="AA_DB_NAME" android:value="my_database.db" />
<meta-data android:name="AA_DB_VERSION" android:value="1" />
```
然后需要在activity中使用 ActiveAndroid.initialize(this);来开启
```java
public class MyActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActiveAndroid.initialize(this);

        //rest of the app
    }
}
```

## Realm
一个不依赖于SqlLite的ORM库,低层使用c++ 实现的
