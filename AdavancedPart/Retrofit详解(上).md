Retrofit详解(上)
===

之前写过一篇文章[volley-retrofit-okhttp之我们该如何选择网路框架](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/volley-retrofit-okhttp%E4%B9%8B%E6%88%91%E4%BB%AC%E8%AF%A5%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9%E7%BD%91%E8%B7%AF%E6%A1%86%E6%9E%B6.md)来分析`Volley`与`Retrofit`之间的区别。之前一直用`Volley`比较多。但是随着`Rx`系列的走红，目前越来越多的项目使用`RxJava+Retrofit`这一黄金组合。而且`Retrofit`使用注解的方式比较方便以及`2.x`版本的提示让`Retrofit`更加完善，今天简单的来学习记录下。

- 有关更多`Volley`的知识请查看[Volley源码分析](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/Volley%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)      
- 有关注解更多的知识请查看[注解使用](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/%E6%B3%A8%E8%A7%A3%E4%BD%BF%E7%94%A8.md)
- 有关更多[RxJava]的介绍请查看[Rx详解系列](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%8A).md)

简介
---

[Retrofit](http://square.github.io/retrofit/)

> A type-safe HTTP client for Android and Java

`type-safe`是什么鬼？    
类型安全代码指访问被授权可以访问的内存位置。例如，类型安全代码不能从其他对象的私有字段读取值。它只从定义完善的允许方式访问类型才能读取。

使用
---

`Gradle`中集成:   

```java
compile 'com.squareup.retrofit2:retrofit:2.1.0'
```  
`Retrofit 2`底层默认使用自家兄弟`OKHttp`作为网络层,并且在它上面进行构建。所以不需要在想`1.x`版本那样在
项目中显式的定义`OkHttp`依赖。所以`Retrofit`与`OkHttp`的关系是后者专注与网络请求的高效优化，而前者专注于接口的封装和调用管理等。    

![image](https://github.com/CharonChui/Pictures/blob/master/retrofit_okhttp_relation.jpg?raw=true)

当然你还需要在清单文件中添加网络请求的权限:   

```
<uses-permission android:name="android.permission.INTERNET"/>
```

我们就以`Github`获取个人仓库的`api`来进行举例测试:   
```java
https://api.github.com/users/{user}/repos
```

- `Retrofit`会将你的`api`封装成`Java`接口   
  ```java
  public interface GitHubService {
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
  }
  ```

- `Retrofit`类会生成一个`GitHubService`接口的实现类:  

  ```java
  Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("https://api.github.com/")
      .build();
  
  GitHubService service = retrofit.create(GitHubService.class);
  ```

- 从创建的`GithubService`类返回的每个`Call`对象调用后都可以创建一个同步或异步的网络请求:   

  ```java
  Call<List<Repo>> repos = service.listRepos("CharonChui");
  ```

- 上面返回的`Call`其实并不是真正的数据结果，它更像一条指令，你需要执行它:   

  ```java
  // 同步调用
  List<Repo> data = repos.execute(); 
   
  // 异步调用
  repos.enqueue(new Callback<List<Repo>>() {
    @Override
    public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
        List<Repo> data = response.body();
        Log.i("@@@", "data size : " + (data == null ? "null" : data.size() + ""));
    }
  
    @Override
    public void onFailure(Call<List<Repo>> call, Throwable t) {
  
    }
  });
  ```

  那如何取消请求呢？ 

  ```java
  repos.cancel();
  ```

上面这一部分代码，你要是拷贝运行后是运行不了的。
当然了，因为木有`Repo`对象。但是添加`Repo`对象也是运行不了的。会报错。 

```java
Process: com.charon.retrofitdemo, PID: 7229
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.charon.retrofitdemo/com.charon.retrofitdemo.MainActivity}: java.lang.IllegalArgumentException: Unable to create converter for java.util.List<com.charon.retrofitdemo.Repo>
   for method GitHubService.listRepos
   at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2281)
   at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2331)
   at android.app.ActivityThread.handleRelaunchActivity(ActivityThread.java:3974)
   at android.app.ActivityThread.access$1100(ActivityThread.java:143)
   at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1250)
   at android.os.Handler.dispatchMessage(Handler.java:102)
   at android.os.Looper.loop(Looper.java:136)
   at android.app.ActivityThread.main(ActivityThread.java:5291)
   at java.lang.reflect.Method.invokeNative(Native Method)
   at java.lang.reflect.Method.invoke(Method.java:515)
   at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:849)
   at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:665)
   at dalvik.system.NativeStart.main(Native Method)
Caused by: java.lang.IllegalArgumentException: Unable to create converter for java.util.List<com.charon.retrofitdemo.Repo>
   for method GitHubService.listRepos
   at retrofit2.ServiceMethod$Builder.methodError(ServiceMethod.java:720)
   at retrofit2.ServiceMethod$Builder.createResponseConverter(ServiceMethod.java:706)
   at retrofit2.ServiceMethod$Builder.build(ServiceMethod.java:167)
   at retrofit2.Retrofit.loadServiceMethod(Retrofit.java:166)
   at retrofit2.Retrofit$1.invoke(Retrofit.java:145)
   at $Proxy0.listRepos(Native Method)
   at com.charon.retrofitdemo.MainActivity$override.onCreate(MainActivity.java:29)
   at com.charon.retrofitdemo.MainActivity$override.access$dispatch(MainActivity.java)
   at com.charon.retrofitdemo.MainActivity.onCreate(MainActivity.java:0)
   at android.app.Activity.performCreate(Activity.java:5304)
   at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1090)
   at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2245)
    ... 12 more
Caused by: java.lang.IllegalArgumentException: Could not locate ResponseBody converter for java.util.List<com.charon.retrofitdemo.Repo>.
 Tried:
```

这是什么鬼?难道官方给的示例代码有问题？ 从`log`上面我们能看出来是返回的数据结构不匹配导致的，返回的是`ResponseBody`的转换器无法转换为`List<Repo>`。    


`Retrofit`是一个将`API`接口转换成回调对象的类，默认情况下`Retrofit`会根绝平台提供一些默认的配置，但是它是支持配置的。   

Converters(解析数据)
---

默认情况下，`Retrofit`只能将`HTTP`响应体反序列化为`OkHttp`的`ResponseBody`类型。并且通过`@Body`也只能接受`RequestBody`类型。    

可以通过添加转换器来支持其他类型。它提供了6中类似的序列化类库来方便进行使用:    

- Gson: com.squareup.retrofit2:converter-gson
- Jackson: com.squareup.retrofit2:converter-jackson
- Moshi: com.squareup.retrofit2:converter-moshi
- Protobuf: com.squareup.retrofit2:converter-protobuf
- Wire: com.squareup.retrofit2:converter-wire
- Simple XML: com.squareup.retrofit2:converter-simplexml
- Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars


我们这里通过`Gson`为例，讲解一下如何使用，首先需要在`gradle`文件中配置对应的支持模块。 

```java
compile 'com.squareup.retrofit2:retrofit:2.1.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
```

下面是一个通过使用`GsonConverterFactory`类来指定`GitHubService`接口使用`Gson`来解析结果的配置。 

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

经过这些改造，就可以了，贴一下完整的代码:  
```java
public interface GitHubService {
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
}

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.github.com/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        GitHubService service = retrofit.create(GitHubService.class);

        Call<List<Repo>> call= service.listRepos("CharonChui");

        call.enqueue(new Callback<List<Repo>>() {
            @Override
            public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
                List<Repo> body = response.body();
                Log.i("@@@", "call " + body.size());
            }

            @Override
            public void onFailure(Call<List<Repo>> call, Throwable t) {

            }
        });
    }
}
```

运行上面的代码,`log`会打印出`12-14 14:43:38.900 21509-21509/com.charon.retrofitdemo I/@@@: call 26`


到这里，我们入门的`Hello World`就完成了。 

`Retrofit`支持的协议包括`GET/POST/PUT/DELETE/HEAD/PATCH`，当然你也可以直接用`HTTP` 来自定义请求。这些协议均以注解的形式进行配置，比如我们已经见过`GET`的用法.

我们发现在`Retrofit`创建的时候需要传入一个`baseUrl(https://api.github.com/)`，在`GitHubService`中会通过注解设置`@GET("users/{user}/repos")`，请求的完整`Url`就是通过`baseUrl`与注解的`value`也就是`path`路径结合起来组成。虽然提供了多种规则，但是统一使用下面这一种是最好的。 
```java
通过注解的value指定的是相对路径，baseUrl是目录形式：
path = "users/CharonChui/repos"，baseUrl = "https://api.github.com/"
Url = "https://api.github.com/users/CharonChui/repos"
```

上面介绍的例子中使用的是`@Path`。 

`@Query`及`@QueryMap`
---

假设我们有一个分页查询的功能:   
```java
GET("/list")
Call<ResponseBody> list(@Query("page") int page);
```

这就相当于`https://api.github.com/list?page=1`这种。`Query`其实就是`Url`中`?`后面的`k-v`。而`QueryMap`就是多个查询条件。     


`@Field`及`@FieldMap`
---

用`Post`的场景相对比较多，绝大多数的服务端接口要做加密、校验等。所以使用`Post`提交表单的场景就会很多。

```java
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```

当然`FieldMap`就是多个的版本了。  

`@Part`及`@PartMap`
---

上传文件时使用。
```java
public interface FileUploadService {  
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
}
```

`@Headers`
---

可以通过`@Headers`来设置静态的请求头

```java
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```

请求头信息可以通过`@Header`注解来动态更新

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
如果传的值是`null`，该请求头将会被忽略，否则将会使用该值得`toString`。 


`RxJava`+`Retrofit`
---

`Retrofit`如何与`RxJava`结合使用呢？ 

- 添加依赖

  ```java
  compile 'com.squareup.retrofit2:retrofit:2.1.0'
  compile 'com.squareup.retrofit2:converter-gson:2.1.0'// 支持gson
  compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'// 支持rxjava
  
  // rxjava part
  compile 'io.reactivex:rxandroid:1.2.1'
  compile 'io.reactivex:rxjava:1.2.3'
  ```

- 修改`Retrofit`的配置，让其支持`RxJava`   

  ```java
  Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("https://api.github.com/")
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava
      .build();
  ```

- 修改`GitHubService`，将返回值改为`Observable`，而不是`Call`。     

  ```java
  public interface GitHubService {
      @GET("users/{user}/repos")
      Observable<List<Repo>> listRepos(@Path("user") String user);
  }
  ```

- 执行部分

  ```java
  GitHubService service = retrofit.create(GitHubService.class);
  
  service.listRepos("CharonChui")
      .subscribeOn(Schedulers.newThread())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe(new Subscriber<List<Repo>>() {
          @Override
          public void onCompleted() {
              Log.i("@@@", "onCompleted");
          }
  
          @Override
          public void onError(Throwable e) {
              Log.i("@@@", "onError : " + e.toString());
          }
  
          @Override
          public void onNext(List<Repo> repos) {
              Log.i("@@@", "onNext : " + repos.size());
              Toast.makeText(MainActivity.this, "size : " + repos.size(), Toast.LENGTH_SHORT).show();
          }
      });
  ```
  

  `RxJava`+`Retrofit`形式的时候，`Retrofit`把请求封装进`Observable`在请求结束后调用 `onNext()`或在请求失败后调用`onError()`。


`Proguard`配置
---

如果项目中使用了`Proguard`，你需要添加如下配置:   
```java
# Platform calls Class.forName on types which do not exist on Android to determine platform.
-dontnote retrofit2.Platform
# Platform used when running on RoboVM on iOS. Will not be used at runtime.
-dontnote retrofit2.Platform$IOS$MainThreadExecutor
# Platform used when running on Java 8 VMs. Will not be used at runtime.
-dontwarn retrofit2.Platform$Java8
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# Retain declared checked exceptions for use by a Proxy instance.
-keepattributes Exceptions
```



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

