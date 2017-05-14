---
title: Android:Dagger2学习之由浅入深
date: 2016-06-05 23:02:14
categories: Android基础
tags: dagger2
---


##  概述

Dagger2是一款使用在Java和Android上的静态的,运行时依赖注入框架.官方地址:http://google.github.io/dagger/

记得当初刚学习Dagger2的时候看了许多博客,但是感觉上手依然困难,所谓光学不练就是这个意思吧

时至今日,用上此框架的同仁越来越多.分析文章也很多,上手相对要简单了许多.

学习Dagger2最先要明白的是其各个注解的含义及工作原理,这样才可以快速的上手和使用.

在这里简要记录一下在使用Dagger2过程中的感受和心得体会.

本文示例代码地址:[Dagger2Sample](https://github.com/BoBoMEe/AndroidDev/tree/dagger2/Dagger2Sample)

<!-- more -->

## 配置信息

首先贴出此篇博客的所有依赖配置信息,因为dagger2需要依赖 apt,所以也需要引入apt插件,

* project  下面的build.gradle文件配置

```java
buildscript {
  repositories {
    jcenter()
    //依赖maven库
    mavenCentral()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:2.1.2'

    // NOTE: Do not place your application dependencies here; they belong
    // in the individual module build.gradle files
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    classpath 'me.tatarka:gradle-retrolambda:3.2.5'
  }
}

allprojects {
  repositories {
    jcenter()
    mavenCentral()
  }
}
```
* app目录下的build.gradle文件配置

```java
apply plugin: 'me.tatarka.retrolambda' //使用lanbda表达式
apply plugin: 'com.neenbedankt.android-apt' //apt插件

android {
  //...
  // 注释冲突
  packagingOptions {
    exclude 'META-INF/services/javax.annotation.processing.Processor'
  }

  // 使用Java1.8
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  testCompile 'junit:junit:4.12'
  compile 'com.android.support:appcompat-v7:23.4.0'

  apt 'com.google.dagger:dagger-compiler:2.0.2'
  compile 'com.google.dagger:dagger:2.0.2'
  provided 'javax.annotation:jsr250-api:1.0'

  compile 'io.reactivex:rxandroid:1.1.0' // RxAndroid
  compile 'io.reactivex:rxjava:1.1.0' // RxJava

  compile 'com.squareup.retrofit2:retrofit:2.0.2' // Retrofit网络处理
  compile 'com.squareup.retrofit2:adapter-rxjava:2.0.2' // Retrofit的rx解析库
  compile 'com.squareup.retrofit2:converter-gson:2.0.2' // Retrofit的gson库

  compile 'com.jakewharton:butterknife:8.0.1' // 标注
  apt 'com.jakewharton:butterknife-compiler:8.0.1' //视图注入

}

```

## Dagger2基础注解

Inject，Component，Module，Provides是Dagger中最基础的几个注解,是整个依赖注入的核心,下面我们来看一下各个注解的作用.

### Inject and Component
`@Inject`:

用来标注`需要依赖的成员` 和 `被依赖类的构造函数`(如果依赖类同时依赖了其他类,其他类的构造函数也要有`@Inject`标注),

**注意:** 使用`@Inject`标注构造函数,不能标注一个类的多个构造函数

```java

  @Inject public UserModel() {
    this.name = "Hello Dagger2,I`m from Inject";
  }
  @Inject public UserModel(String name) {
    this.name = name;
  }

```
上面的写法会报错:`Error:(29, 18) 错误: Types may only contain one @Inject constructor.`

`@Component`:

光有`@Inject`可不行,还需要一个东西将他两联系起来才能让`依赖类`找到`被依赖对象`,其中`Component`就起到了这个作用,在`Dagger2`中是以接口形式存在,
是用来连接 `需要依赖类` 和 `被依赖类`,`Component`使用`injectXX(XX xx)`将依赖注入到需要依赖的地方,

#### 基本使用

接下来我们来看一下Dagger2最基本的用法

第一步:编写JavaBean
```java
/**
 * Created on 16/6/8.下午8:57.
 *
 * @author bobomee
 */
public class UserModel {
  private String name;

  @Inject public UserModel() {
    this.name = "Hello Dagger2,I`m from Inject";
  }

  public String getName() {
    return name;
  }
}
```
第二步:创建Component
```java
/**
 * Created on 16/6/8.下午8:59.
 *
 * @author bobomee
 */
@Component public interface UserComponent {

  void inject(MainActivity mainActivity);

  final class Initializer {
    private Initializer() {
    }

    public static UserComponent init() {
      return DaggerUserComponent.create();
    }
  }
}
```

第三步:构建依赖,先build一下生成dagger图谱
```java
public class MainActivity extends AppCompatActivity {

  @BindView(R.id.text) TextView text;

  @Inject UserModel userModel;

  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    ButterKnife.bind(this);

    //inject dependencies
    UserComponent.Initializer.init().inject(this);

    text.setText(userModel.getName());
  }
}
```
**注意:** `@Inject`成员不能是`private`的,否则会报:`Error:(35, 29) 错误: Dagger does not support injection into private fields`

### Provides and Module
`@Provides`:

自定义依赖,Dagger2中不仅提供了`@Inject`标注,他比`@Inject`更加强大,不仅可以提供本地依赖,还可以提供第三方依赖(`第三方库和Android系统类`不可以用`@Inject`注解构造函数).

**注意:** `@Provides`的优先级高于`@Inject`

`@Module`:

所有的`@Provides`都必须包含在`@Module`内部,相当于简单工厂,提供了各种依赖`@Provides`方法.之后再将Module加入到Component管理即可完成依赖注入(Component不仅可以从`@Inject`找到被依赖类,还可以从`@Module`找到被依赖类)

**注意:** 通过modules列出一个Component所有依赖的Module,如果缺失任何一个编译会报错

#### 自定义Module使用

第一步:编写AppModule,提供依赖`@Provide`方法,其中`@Singleton`是自定义注解
```java
/**
 * Created on 16/6/8.下午9:30.
 *
 * @author bobomee
 */
@Module public class AppModule {

  private App app;//App为我们自定义的Application

  public AppModule(App app) {
    this.app = app;
  }

  @Provides @Singleton public App provideApp() {
    return app;
  }
}
```

第二步:创建Component,这里使用了`@Singleton`,必须和Module中`@Provides`方法修饰相同,否则编译报错

```java
/**
 * Created on 16/6/8.下午9:31.
 *
 * @author bobomee
 */
@Singleton @Component(modules = {
    AppModule.class
}) public interface AppComponent {

//将依赖注入到自定义的Application
  void inject(App app);

  final class Initializer {
    private Initializer() {
    }

    public static AppComponent init(App app) {
      return DaggerAppComponent.builder().appModule(new AppModule(app)).build();
    }
  }
}
```

第三步:自定义Application中完成依赖

```java
/**
 * Created on 16/6/8.下午9:29.
 *
 * @author bobomee
 */
public class App extends Application {

  private AppComponent appComponent;

  @Inject static App app;

  public static App get() {
    return app;
  }

  @Override public void onCreate() {
    super.onCreate();

//当inject完成后app就不为空了,且和Application生命周期相同,因此//Singleton可以起到全局单例的作用
    buildComponent();
  }

  private void buildComponent() {
    appComponent = AppComponent.Initializer.init(this);
    appComponent.inject(this);
  }

//向外提供appComponent,方便其他依赖appComponent的component构建
  public AppComponent component() {
    return appComponent;
  }
}
```

第四步:使用依赖

```java
//MainActivity中
 @BindView(R.id.text1) TextView text1;

 text1.setText(App.get().toString());
```

## Component组织方式

1. 一个应用中,必须包含一个全局的Component(类似于上面的AppComponent),管理整个App的实例.

2. 一般AppComponent和Application生命周期相同,所以注入到Application中即可.

3. 因为Application是全局单例的,所以AppModule中创建的实例也是单例的(`@Singleton`注解就是依照此原理).

4. 根据具体功能或者单独的一个页面定义一个Component.

5. 某个单独的Component要用到全局实例的时候,可以通过继承AppComponent来实现

6. Component之间的关系有 `依赖(dependencies)`,`包含(SubComponent)`,`继承方式(extends)`

### Component依赖写法
接下来来看一下一个典型的dependencies写法

第一步: 定义注入到MainActivity的Product

```java
/**
 * Created on 16/6/8.下午9:48.
 *
 * @author bobomee
 */
public class Product {

  private String productQualifier;

  public Product(String productQualifier) {
    this.productQualifier = productQualifier;
  }

  public String getProductQualifier() {
    return productQualifier;
  }
}
```

第二步:定义Component

```java
/**
 * Created on 16/6/8.下午8:59.
 *
 * @author bobomee
 */
@Component public interface UserComponent {

  // unused
  //void inject(MainActivity mainActivity);

  final class Initializer {
    private Initializer() {
    }

    public static UserComponent init() {
      return DaggerUserComponent.create();
    }
  }
}
```

```java
/**
 * Created on 16/6/8.下午10:00.
 *
 * @author bobomee
 */
@ActivityScope @Component(dependencies = UserComponent.class,
    modules = ProductModule.class) public interface ProductComponent extends UserComponent {

  void inject(MainActivity mainActivity);

  final class Initializer {
    private Initializer() {
    }

    public static ProductComponent init(UserComponent userComponent) {
      //dependency usercomponent
      return DaggerProductComponent.builder().productModule(new ProductModule()).userComponent(userComponent).build();
    }
  }
}
```

**注意:** 此处依赖UserComponent,需要将UserComponent中的注入删除.

```java
//MainActivity
//inject dependencies
ProductComponent.Initializer.init(UserComponent.Initializer.init()).inject(this);

```

`@Scope`:

注解作用域.用于更好的组织Component,和定义Component的粒度.
`@Singleton`是Dagger2默认实现的,用于管理全局单例(AppComponent中),
`@Scope`用在Component 和 Module 头上,
如果要定义一个实例的生命周期在Activity内,则可以定义`@ActivityScope`(当然名称随便起,只要对应即可)

```java
/**
 * Created on 16/6/8.下午9:55.
 *
 * @author bobomee
 */
@Scope @Retention(RetentionPolicy.RUNTIME) public @interface ActivityScope {
}
```

```java
/**
 * Created on 16/6/8.下午10:00.
 *
 * @author bobomee
 */
@ActivityScope @Component(dependencies = UserComponent.class,
    modules = ProductModule.class) public interface ProductComponent extends UserComponent {
//...
}
```

```java
/**
 * Created on 16/6/8.下午9:49.
 *
 * @author bobomee
 */
@Module public class ProductModule {

  @ActivityScope @Provides Product provideProduct() {
    return new Product("this is a product");
  }
//...
}
```

## 依赖迷失之Qualifier

`@Qualifier`:

限定符,当依赖类中需要被依赖类的两个不同对象的时候,帮助我们去为相同接口的依赖创建“tags”.
如我们需要两个不同的level的product,Qualifier会帮助你区分对应的一个,

```java
/**
 * Created on 16/6/8.下午9:57.
 *
 * @author bobomee
 */
@Qualifier @Retention(RetentionPolicy.RUNTIME) public @interface ProductLevel {
  String value() default "";
}
```
* 声明Module

```java
/**
 * Created on 16/6/8.下午9:49.
 *
 * @author bobomee
 */
@Module public class ProductModule {

//....
  @ActivityScope @ProductLevel("good") @Provides Product provideGoodProduct() {
    return new Product("good product");
  }

  @ActivityScope @ProductLevel("bad") @Provides Product provideBadProduct() {
    return new Product("bad product");
  }
}
```
* 注入Inject

```java
@Inject @ProductLevel("good") Product product1;
@Inject @ProductLevel("bad") Product product2;
```

## 总结

* Inject用来标注`依赖`和`被依赖的构造函数`
* Provides提供依赖的方法上添加的注解,provide方法需要包含在Module中
* Module专门提供依赖,类似工厂模式,包含Provides方法
* Component`依赖`和`被依赖`的桥梁,(先从Module中找依赖,再从Inject构造函数找)
* Scope自定义注解,用于标示作用域,命名随意,对应即可,其中@Singleton是一个系统的模式实现.(管理Module与Component的匹配)
* Qualifier自定义注解,用于解决一个实例可以被多种方式构建的依赖迷失问题
* Component有三种组织关系,分为依赖,包含和继承,用于解决依赖复用与共享问题

>依赖注入的过程:
>步骤1：查找Module中是否存在创建该类的方法。

>步骤2：若存在创建类方法，查看该方法是否存在参数

>...........步骤2.1：若存在参数，则按从**步骤1**开始依次初始化每个参数

>...........步骤2.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束

>步骤3：若不存在创建类方法，则查找Inject注解的构造函数，看构造函数是否存在参数

>...........步骤3.1：若存在参数，则从**步骤1**开始依次初始化每个参数

>...........步骤3.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束


参考:

这里推荐牛晓伟的三篇博客:

[Android：dagger2让你爱不释手-基础依赖注入框架篇](http://www.jianshu.com/p/cd2c1c9f68d4)

[Android：dagger2让你爱不释手-重点概念讲解、融合篇](http://www.jianshu.com/p/1d42d2e6f4a5)

[Android：dagger2让你爱不释手-终结篇](http://www.jianshu.com/p/65737ac39c44)
