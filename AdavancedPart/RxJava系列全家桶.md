`RxJava Android`开发全系列
===

有关`RxJava`的介绍请看[RxJava详解系列](https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%8A).md)

要说16年`android`开发中要说那个应用最流行，那就是`RxJava`了，现在越来越多的`android`项目都会用到`RxJava`，下面就介绍一些`RxJava`必备的扩张库。 

`RxAndroid`
---

[RxAndroid](https://github.com/ReactiveX/RxAndroid)

> Android specific bindings for RxJava.

> This module adds the minimum classes to RxJava that make writing reactive components in Android applications easy and hassle-free. More specifically, it provides a Scheduler that schedules on the main thread or any given Looper.

`Android`中使用`RxJava`的必备类库，虽然里面提供的内容并不多只有`AndroidSchedulers`、`HandlerScheduler`、`LooperScheduler`，但是这些确是`Android`开发中的精髓。 

`RxLifecycle`
---

[RxLifecycle](https://github.com/trello/RxLifecycle)

> The utilities provided here allow for automatic completion of sequences based on Activity or Fragment lifecycle events. This capability is useful in Android, where incomplete subscriptions can cause memory leaks.

`RxLifecycle`提供了一些配合`Activity`、`Fragment`生命周期使用的订阅管理的相关功能。例如使用`RxJava`执行一些耗时的操作，但是在执行过程中，用户退出了当前`Activity`，这时如果`Observable`未取消订阅就会导致内存泄漏，而`RxLifecycle`就是为了接着这些问题的。在`Activity`销毁的时候`RxLifecycle`会自动取消订阅。   
`RxBinding`
---

[RxBinding](https://github.com/JakeWharton/RxBinding)

> RxJava binding APIs for Android UI widgets from the platform and support libraries.

`RxBinding`是把`Android`中`UI`事件转换为`RxJava`的方式，例如点击时间，每次点击后`Observable`的订阅者`Observer`都会通过`onNext()`回调得知。  

`Retrofit`
---

[Retrofit](https://github.com/square/retrofit)

> Type-safe HTTP client for Android and Java by Square, Inc.

良心企业`Square`出品的一个基于`OkHttp`的网络请求类库，完美支持`RxJava`。 

`SqlBrite`
---

[SQLBrite](https://github.com/square/sqlbrite)

> A lightweight wrapper around SQLiteOpenHelper and ContentResolver which introduces reactive stream semantics to queries.

良心企业`Square`出品的一个支持`RxJava`的`Sqlite`数据库的操作库。 

`RxPermissions`

[RxPermissions)](https://github.com/tbruyelle/RxPermissions)

> This library allows the usage of RxJava with the new Android M permission model.

`Android M`上动态权限申请的类库。



- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
