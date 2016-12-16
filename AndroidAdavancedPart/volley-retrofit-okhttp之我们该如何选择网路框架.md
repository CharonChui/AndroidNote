volley-retrofit-okhttp之我们该如何选择网路框架
===

说起`Volley`、`Retrofit`、`OkHttp`相信基本没有人不知道。当然这里把`OkHttp`放进来可能有些不恰当。
因为`OkHttp`的官方介绍是`An HTTP+HTTP/2 client for Android and Java applications`。
也就是说`OkHttp`是基于`http`协议封装的一套请求客户端。它是真正的网络请求部分，
与`HttpClient`、`HttpUrlConnection`是一样的，
但是显然它的效率非常高(说到这里顺便提一嘴，从`Android 4.4`开始`HttpUrlConnection`内部默认使用的也是`OkHttp`，
具体请参考之前的文章[HttpUrlConnection详解](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/HttpURLConnection%E8%AF%A6%E8%A7%A3.md))。
而`Volley`、`Retrofit`是控制请求的队列、切换、解析、缓存等逻辑。所以`Volley`和`Retrofit`都可以结合`OkHttp`来使用。 


在`Android`开发中有很多网络请求框架，但是比较过来比较过去，最后最倾向的就是这两个:    

- `Volley`:`Google`发布的网络请求框架，专门为移动设备定制，小而美。     
- `Retrofit`:良心企业    `Square`由大神`JakeWharton`主导的开源项目,是基于`OkHttp`封装的一套`Resetful`网络请求框架。`Type-safe HTTP client for Android and Java by Square, Inc.`


有关`Volley`的介绍请看之前发布的文章[Volley源码分析](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/Volley%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)


这里就不分别介绍他俩了，直接说各自的优缺点:    

- `Retrofit`使用起来更简单。而`Volley`配置起来会稍微麻烦，因为`Volley`可以使用`HttpClient`、`HttpUrlConnection`、`OkHttp`我们需要根据自己的需求去配置。而`Retrofit`只能结合`OkHttp`使用。   

- `Retrofit`依赖于`OkHttp`，从而会导致它的包大小会比`Volley`的大。 

- `Volley`有很好的内存缓存管理，它在解析之前会将整个相应部分都加载到内存中，所以它对于小的网络请求非常合适，但是不支持`post`大数据，所以不适合上传文件。而`Retrofit`使用的是硬盘缓存，所以相比起从缓存这块来讲`Retrofit`可能会更慢一些。   

- `Retrofit`依赖于`OkHttp`，而`OkHttp`自身会避免同时两次请求同一个请求。所以`Retrofit`同样会和`Volley`一样去避免重复的请求，只不过它是在网络层来处理的。 

- `Volley`在网络请求部分默认依赖于`Apache HttpClient`。而`Apache HttpClient`从`API 23`开始已经在`Android`中被移除并废弃了。这就是为什么很多开发者会认为`Volley`已经过时了，因为`Volley`并没有迁移到新的未废弃的代码。    

- 默认情况下`Volley`会在`DefaultRetryPolicy`中会将读取和连接的超时时间设置为`2.5s`，并且对每次请求失败或者超时都有一次自动重试。 所以对于一些服务器响应可能会超过`2s`的请求，开发者需要格外的小心下。`Retrofit`的默认超时时间是`10s`，而且它对失败或者超时的操作不会自动重试。      
- 很多开发者都会说`Retrofit`会比`Volley`更快。因为有人专门去测试过，其实这里是不严谨的。因为`Volley`可以结合使用`HttpUrlConnection`、`HttpClient`、`OkHttp`等来使用，而`Retrofit`是用`OkHttp`一起，所以如果你让`Volley`结合`OkHttp`之后再来测试你就会发现总体来说其实他们不相上下。    


- `Volley`实现了很完善的`Activity`声明周期管理。

虽然`Volley`之前也有一些问题，但是它们也都被各个大神修复。


所以综合起来说使用`Volley+OKHttp`的组合是非常不错的，既可以保证速度又可以满足对缓存、重试等的处理。但是如果你是`RxJava`的使用者那你可能会更偏向于使用`Retrofit`，因为`Retrofit`可以无缝结合`RxJava`使用。目前主流的一套框架就是`Retrofit + OkHttp + RxJava + Dagger2 `，但是对使用者的要求也相对要高些。

		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 