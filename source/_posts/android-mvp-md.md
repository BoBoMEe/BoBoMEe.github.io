---
title: 浅谈Android架构之MVP,MVVM
date: 2016-06-09 11:48:32
tags: architecture
---



## 概述

`MVP(Model-View-Presenter)`是传统`MVC(Model-View-Controller)`在Android开发上的一种变种、进化模式。主要用来隔离UI、UI逻辑和业务逻辑、数据,创建松散耦合并可重用的对象。

我们知道View层是容易变化且多种多样的,业务逻辑也是多种多样的,与传统的MVC相比,P充当了C的作用.
Model存储数据，View表示Model的表现，Presenter协调两者之间的通信.

而后在MVP基础上也出现了一些变种,如MVVM,MVPVM等,相比较MVP而言,MVVM使数据绑定变得更加简单.MVPVM在MVVM中加入引入Presenter层

<!-- more -->

## MVC,MVP,MVVM,MVPVM图解

这里感谢源博客和图片提供者.

### MVC:
![MVC_Android MVP模式 简单易懂的介绍方式](http://img.blog.csdn.net/20160609230144717)

MVC中View接受事件,并调用Controller来操作Model,同时,当Model实例的数据发生变化后，Controller再更新界面(当然View也可以直接更新Model)。

在传统的开发中Activity俨然既充当了Controller又充当了View的作用.既需要接受用户响应操作Model,又要更新界面.
这样做有一个好处就是数据的更新变得很简单,但是缺点也十分明显,Activity是非臃肿,后期不好维护.

### MVP:
![MVP_Android MVP模式 简单易懂的介绍方式](http://img.blog.csdn.net/20160609230312969)

MVP中将业务逻辑单独抽出Presenter,View层变成一个被动的东西,Presenter负责完成View层与Model层的交互.
View 不可以直接和Model交互(MVC中允许Model和View交互),只有Presenter告知其更新，它才会去更新.
而且Presenter和View的交互是通过接口来完成.

### MVVM:
![MVVM_MVVM_Android-CleanArchitecture](http://img.blog.csdn.net/20160609230346176)

MVVM在Android上对应data binding,MVVM最先使用在WPF中,通过ViewModel和View的映射,完成了View和Model的双向绑定.
View的事件直接传递到ViewModel，ViewModel去对Model进行操作并接受更新.进而反馈到View上.
因为ViewModel与View的耦合,MVVM有一个缺点就是View的复用问题,
因为去掉了Presenter,View层依然过重.

### MVPVM:
![MVPVM_从零开始的Android新项目3 - MVPVM in Action, 谁告诉你MVP和MVVM是互斥的](http://img.blog.csdn.net/20160609230425176)

MVPVM是MVP和MVVM的演化版本,降低了ViewModel与View的耦合,View只需要实现ViewModel的观察者接口实现更新.ViewModel不再对Model直接进行操作,而是交给了Presenter.Presenter操作Model并反馈到ViewModel上
Model,View,ViewModel之间通过Presenter联系了起来.

## MVP实践

Google官方[android-architecture ](https://github.com/googlesamples/android-architecture)无疑是学习MVP最佳资料,官方项目展示了通过不同方式来实现MVP,其中展示了通过basic,loaders,data binding,clean,dagger,content provider,rxjava等不同方式来实现相同的功能,当然我们只要掌握其精髓,触类旁通就可以

### 看官方MVP的体会

* 单独模块抽取`IContract`接口管理`IView`和 `Presenter`接口,一目了然,而且维护也方便
```java
public interface AddEditTaskContract {

    interface View extends BaseView<Presenter> {
//...
    }

    interface Presenter extends BasePresenter {
//...
    }
}
```
* 当Fragment作为`View`的时候,Activity负责创建`IView`和`Presenter`实例,并将二者关联起来,官方的这幅图即可说明

![fragment_mvp](http://img.blog.csdn.net/20160609230727130)

代码说明:

```java
//todo-mvp$TasksActivity
@Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.tasks_act);
       // ...
       TasksFragment tasksFragment =
               (TasksFragment) getSupportFragmentManager().findFragmentById(R.id.contentFrame);
       //...
       mTasksPresenter = new TasksPresenter(
               Injection.provideTasksRepository(getApplicationContext()), tasksFragment);
       //...
   }
```

* `IPresenter`由具体的`Presenter`实现,`IView`由View层(`Activity/Fragment`)实现,`IView`实现类中包含了`Presenter`,他们通过如下方式实现绑定.
```java
public interface BaseView<T> {
      // View中保持对Presenter的引用。
      void setPresenter(T presenter);
}
```

* 同时在`Presenter`构造时需要传入`IView`对象(用于更新`View`).

```java
public class TasksPresenter implements TasksContract.Presenter {
    private final TasksRepository mTasksRepository;
    private final TasksContract.View mTasksView;
//...
    public TasksPresenter(@NonNull TasksRepository tasksRepository, @NonNull TasksContract.View tasksView) {
        mTasksRepository = checkNotNull(tasksRepository, "tasksRepository cannot be null");
        mTasksView = checkNotNull(tasksView, "tasksView cannot be null!");
//setPresenter
        mTasksView.setPresenter(this);
    }
```

* `Model`不仅仅是`JavaBean`,而且拥有操作数据的业务逻辑(获取、存储、更新),同时`Model`也可以将业务抽成接口,这样就可以随意拓展数据源



## MVP变种

MVP架构的好处就不多说了,但是使用`Activity/Fragment`作为`View`层有如下问题,
参考[一种在android中实现MVP模式的新思路](http://www.devtf.cn/?p=27)

* 当内存不足,Activity被回收后,这使得状态的保存和恢复成为问题,因为涉及到了Model操作.
* 生命周期的控制问题也很麻烦,需要在Presenter中写一大堆和生命周期相关的接口规范

* Activity中包含了很多系统服务,逻辑操作方便

### 使用Activity/Fragment作为Presenter

`Activity/Fragment`中的系统服务和业务逻辑紧密相关.理想的状态是View不涉及到逻辑操作.

`Activity/Fragment`作为Presenter,需要将其UI逻辑抽取到一个单独的类中来管理.

UI逻辑接口
```java
public interface Vu {
    void inflate(LayoutInflater inflater, ViewGroup container, Bundle bundle);
    View getView();
}
```

作为Presenter的BaseActivity,覆盖了所有生命周期方法

```java
public abstract class BasePresenterActivity<V extends Vu> extends Activity {

    protected V vu;

    @Override
    protected final void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        try {
            vu = getVuClass().newInstance();
            vu.inflate(getLayoutInflater(), null,null);
            setContentView(vu.getView());
            onBindVu();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected final void onDestroy() {
        onDestroyVu();
        vu = null;
        super.onDestroy();
    }

    protected abstract Class<V> getVuClass();

    protected void onBindVu(){};

    protected void onDestroyVu() {};

}
```
### 举个例子:
具体的UI逻辑,不论Presenter变为Activity还是Fragment都不用改变.在周期方法中改变View对外的操作即可.
```java
public class HelloVu implements Vu {

View view;
TextView helloView;

@Override
public void init(LayoutInflater inflater, ViewGroup container) {
    view = inflater.inflate(R.layout.hello, container, false);
    helloView = (TextView) view.findViewById(R.id.hello);
}

@Override
public View getView() {
    return view;
}

public void setHelloMessage(String msg){
    helloView.setText(msg);
}

}
```
具体的Presenter
```java
public class HelloActivity extends BasePresenterActivity {
@Override
protected void onBindVu() {
    vu.setHelloMessage("Hello World!");
}

@Override
protected Class<MainVu> getVuClass() {
    return HelloVu.class;
}

}
```
## MVVM:数据的动态绑定

官方文档:
https://developer.android.com/topic/libraries/data-binding/index.html
使用Data Binding后,我们的xml和之前是不太一样的.抄袭自官方文档

```java
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="user" type="com.example.User"/>
    </data>
    <LinearLayout>
    ....
    </LinearLayout>
</layout>
```

相应的Activity的setContentView也会变化

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```

这里一个JavaBean对应一个xml布局文件,View的复用变得很困难.

## TheMVP:MVVM和MVP的结合

TheMVP解决了这个问题,通过在Presenter中定义ViewModel接口,实现数据的双向绑定与MVP的结合.
项目地址: [kymjs](https://github.com/kymjs)/**[TheMVP](https://github.com/kymjs/TheMVP)**

核心代码:这里的IDelegate相当与上面的Vu
```java
//ViewModel接口
public interface DataBinder<T extends IDelegate, D extends IModel> {
    /**
     * 将数据与View绑定，这样当数据改变的时候，框架就知道这个数据是和哪个View绑定在一起的，就可以自动改变ui
     * 当数据改变的时候，会回调本方法。
     *
     * @param viewDelegate 视图层代理
     * @param data         数据模型对象
     */
    void viewBindModel(T viewDelegate, D data);
}
```

Presenter:在数据改变的时候调用notifyModelChanged()更新View
```java
public abstract class DataBindActivity<T extends IDelegate> extends
        ActivityPresenter<T> {
    protected DataBinder binder;

    public abstract DataBinder getDataBinder();

    public <D extends IModel> void notifyModelChanged(D data) {
        binder.viewBindModel(viewDelegate, data);
    }
}
```
## MVPVM:View复用与瘦身

在MVPVM作者的介绍中.

* Model层,和MVP中的Model是类似的.即PO或者DO
* View层,依然是由Activity/Fragment来担当,不同的是需要实现ViewModel的观察者接口,来实现View的动态更新
* Presenter层,如上图所示,Presenter作为核心,连接着M,V,VM
* VM层,和MVVM中的VM是类似的,只是为了让VM可以重用.不再和特定的View绑定,而且不再直接对Model进行操作.

详见:[从零开始的Android新项目3 - MVPVM in Action, 谁告诉你MVP和MVVM是互斥的](http://blog.zhaiyifan.cn/2016/03/16/android-new-project-from-0-p3/)

最后附上用MVP写的一个小Demo:
[MVP](https://github.com/BoBoMEe/AndroidDev/tree/mvp/MVP)


参考:
[一种在android中实现MVP模式的新思路](http://www.devtf.cn/?p=27)

[ANDROID MVP模式 简单易懂的介绍方式](http://kaedea.com/2015/10/11/android-mvp-pattern/)

[MVVM_Android-CleanArchitecture](http://rocko.xyz/2015/11/07/MVVM_Android-CleanArchitecture/)

[从零开始的Android新项目3 - MVPVM in Action, 谁告诉你MVP和MVVM是互斥的](http://blog.zhaiyifan.cn/2016/03/16/android-new-project-from-0-p3/#rd?sukey=ecafc0a7cc4a741b87b0faa09babd83d0aaade29e75380871b478818ebd3831b8e0f7185e732c99cdeaa2efe0922b0f8)

[用MVP架构开发Android应用](http://kymjs.com/code/2015/11/09/01)
