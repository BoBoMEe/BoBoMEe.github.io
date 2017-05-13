---
title: Java依赖注入（控制反转）
date: 2015-12-30 21:53:46
tags: java
---

## 概述

名词释义：
依赖注入(Dependency Injection)，俗称DI。我们将互相协的关系称为依赖关系，在Java应用中，A对象调用了B对象的方法，我们可以说A依赖于B。
系统将实例创建出来，供调用着使用，就可以看作是系统将依赖注入了调用者。

控制反转(Inversion of Control),俗称IoC。控制反转就是关于一个对象如何获取他所依赖的对象的引用这个责任的反转。在Java中，就是不需要调用者new出来一个依赖对象，这个责任交给系统，在需要的时候直接使用系统提供的实例。

<!-- more -->

## 优缺点

优点：依赖注入在某种程度上实现了热插拔，降低了耦合度。
缺点：有些依赖注入框架使用的是反射，一定程度降低了效率。

## 实例

```java
classA
{
    AInterface a;

    A(){}

    AMethod()//一个方法
    {
        a = new AInterfaceImpl();
    }
}
```
这里面 Class A与AInterfaceImpl就是依赖关系,如果想使用AInterface的另外一个实现就需要更改代码了,依赖注入就是为了解决这种耦合关系的
使用new（对象创建）是一种硬编码,是代码耦合度变得很高,不方便测试.依赖注入简单的讲就是通过外界传入依赖来进行成员变量的初始化,比如可以采用如下方式

```java
classA
{
    AInterface a;

    A(){}

    AMethod(AInterface a)//一个方法
    {
        this.a = a;
    }
}
```

## 依赖注入方式

依赖注入一般有如下几种方式

- Contructor Injection(构造函数注入)

``` java
public interface IFather {
 //method
}
```

``` java
public class Human {
   IFather father;
   public Human(IFather father) {
       this.father = father;
   }
}
```

- 基于setter,通过JavaBean的属性(setter方法)为可服务对象指定服务

``` java
public class Human  {
IFather father;
   public void setIFather(IFather father) {
       this.father = father;
   }
}
```

- 接口注入

``` java
// 注入功能的interface
public interface InjectFinder {
   void injectFinder(IFather father);
}
```

``` java
// 让我们的Human实现接口
public class Human implements InjectFinder   {
IFather father;
public void injectFinder(IFather father) {
      this.father = father;
   }
}
```

参考:
[dependency-injection](https://github.com/android-cn/blog/tree/master/java/dependency-injection)
[dependency-injection-theory](http://codethink.me/2015/08/01/dependency-injection-theory/)
[依赖注入（Dependency Injection）模式](http://blog.csdn.net/yqj2065/article/details/8510074)
