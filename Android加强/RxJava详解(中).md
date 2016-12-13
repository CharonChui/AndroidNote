RxJava详解(中)
===


说好的简洁呢？
---

上面这一部分，又是介绍、又是`Hello World`、又是数据变换，但是你会发现然而并没有什么卵用。说好的简洁一点也木有体现出来。    

下面我们会通过一个简单的例子来进行说明。现在我们有这样一个需求:    

> 有一个服务提供了一些`API`来搜索整个网络上的符合查询关键字的所有猫的图片。 每个图片包含一个可爱程度的参数(一个整数值表示其可爱程度)。 我们的任务就是下载所有猫的列表并选择最可爱的那个，把它的图片保存到本地。

####`Model`和`API`


下面是猫的数据结构`Cat`:   

```java
public class Cat implements Comparable<Cat>{
    Bitmap image;
    int cuteness;

    @Override
    public int compareTo(Cat another) {
        return Integer.compare(cuteness, another.cuteness);
    }
}
```

我们的`API`会调用`cat-sdk.jar`中堵塞式的接口。
```java
public interface Api {
    List<Cat> queryCats(String query);
    Uri store(Cat cat);
}
```

看起来很清晰吧？当然了，我们继续写客户端的业务逻辑.
```java
public class CatsHelper {

    Api api;

    public Uri saveTheCutestCat(String query){
        List<Cat> cats = api.queryCats(query);
        Cat cutest = findCutest(cats);
        Uri savedUri = api.store(cutest);
        return savedUri;
    }

    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

通俗易懂、简单明了，非常酷的代码。主要的函数`saveTheCutestCat()`只包含了3个方法。使用这些方法然后等待方法执行完并接受返回值就可以了。

非常简单、有效。接下来我们看一下这种方式的其他优点。  

####组合


可以看到我们的`saveTheCutestCat`由其他三个函数调用所组成的。我们通过函数来把一个大功能分割为每个容易理解的小功能。通过函数调用来组合使用这些小功能。使用和理解起来都相当简单

####异常传递


另外一个使用函数的好处就是方便处理异常。每个函数都可以通过抛出异常来结束运行。该异常可以在抛出异常的函数里面处理，也可以在调用该函数的外面处理，所以我们无需每次都处理每个异常，我们可以在一个地方处理所有可能抛出的异常。


```java
try{
    List<Cat> cats = api.queryCats(query);
    Cat cutest = findCutest(cats);
    Uri savedUri = api.store(cutest);
    return savedUri;
} catch (Exception e) {
    e.printStackTrace()
    return someDefaultValue;
}
```

这样，我们就可以处理这三个函数中所抛出的任何异常了。如果没有`try catch`语句，我们也可以把异常继续传递下去。


向异步粗发
---

但是，现实世界中我们往往没法等待。有些时候你没法只使用阻塞调用。在`Android`中你需要处理各种异步操作。
就那`Android`的`OnClickListener`接口来说吧，如果你需要处理一个`View`的点击事件，你必须提供一个该`Listener` 的实现来处理用户的点击事件。下面来看看如何处理异步调用。


####异步网络调用


假设我们的`cats-sdk.jar`使用了异步调用的`API`来访问网络资源，
这样我们的新`API`接口就变为这样了：

```java
public interface Api {
    interface CatsQueryCallback {
        void onCatListReceived(List<Cat> cats);
        void onError(Exception e);
    }


    void queryCats(String query, CatsQueryCallback catsQueryCallback);

    Uri store(Cat cat);
}
```
这样我们查询猫的操作就变为异步的了， 通过`CatsQueryCallback`回调接口来结束查询的数据和处理异常情况。
我们的业务逻辑也需要跟着改变一下：

```java
public class CatsHelper {
 
    public interface CutestCatCallback {
        void onCutestCatSaved(Uri uri);
        void onQueryFailed(Exception e);
    }
 
    Api api;
 
    public void saveTheCutestCat(String query, CutestCatCallback cutestCatCallback){
        api.queryCats(query, new Api.CatsQueryCallback() {
            @Override
            public void onCatListReceived(List<Cat> cats) {
                Cat cutest = findCutest(cats);
                Uri savedUri = api.store(cutest);
                cutestCatCallback.onCutestCatSaved(savedUri);
            }
 
            @Override
            public void onError(Exception e) {
                cutestCatCallback.onQueryFailed(e);
            }
        });
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

我们没法让`saveTheCutestCat`函数返回一个值了， 我们需要一个回调接口来异步的处理结果。
这里我们再进一步，使用两个异步操作来实现我们的功能,例如我们需要使用异步`IO`来写文件。

```java
public interface Api {
    interface CatsQueryCallback {
        void onCatListReceived(List<Cat> cats);
        void onQueryFailed(Exception e);
    }
 
    interface StoreCallback{
        void onCatStored(Uri uri);
        void onStoreFailed(Exception e);
    }
 
 
    void queryCats(String query, CatsQueryCallback catsQueryCallback);
 
    void store(Cat cat, StoreCallback storeCallback);
}
```

我们的`helper`会变成:    

```java
public class CatsHelper {

    public interface CutestCatCallback {
        void onCutestCatSaved(Uri uri);
        void onError(Exception e);
    }

    Api api;

    public void saveTheCutestCat(String query, CutestCatCallback cutestCatCallback){
        api.queryCats(query, new Api.CatsQueryCallback() {
            @Override
            public void onCatListReceived(List<Cat> cats) {
                Cat cutest = findCutest(cats);
                api.store(cutest, new Api.StoreCallback() {
                    @Override
                    public void onCatStored(Uri uri) {
                        cutestCatCallback.onCutestCatSaved(uri);
                    }

                    @Override
                    public void onStoreFailed(Exception e) {
                        cutestCatCallback.onError(e);
                    }
                });
            }

            @Override
            public void onQueryFailed(Exception e) {
                cutestCatCallback.onError(e);
            }
        });
    }

    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

现在我们再来看看这部分代码？还是之前那样简单暴力？现在有太多的干扰代码、匿名类，这简直是太恐怖了，但是他们的业务逻辑其实是一样的，都是查询猫的列表数据，然后找出最可爱的猫并保存它的图片。    

上面说好的组合功能没有了，现在你没法像阻塞操作一样来组合调用每个功能了，异步操作中，每次你都必须通过回调接口来手工的处理结果。 

上面说好的异常处理也没有了，异步代码中的异常不会自动传递，我们需要手动的去重新传递。(`onStoreFailed()`和`onQueryFailed()`就是干这事的)


####结果？


然后呢？我们可以怎么做？我们能不能使用无回调的模式？我们试着修复一下。 


####奔向更好的世界     
####通用的回调


如果我们仔细的观察下回调接口，我们会发现它们的共性:   

- 它们都有一个分发结果的方法(`onCutestCatSaved()`,`onCatListReceived()`,`onCatStored()`)
- 它们中的绝大部分都有一个处理操作过程中异常的方法(`onError()`, `onQueryFailed()`, `onStoreFailed()`)


所以我们可以创建一个通用的回调来取代它们。但是我们无法修改`api`的调用结构，我们只能创建一个包裹层的调用。   

我们通用的回调如下:  

```java
public interface Callback<T> {
    void onResult(T result);
    void onError(Exception e);
}
```

我们创建一个`ApiWrapper`类来改变我们调用的结构:   

```java
public class ApiWrapper {
    Api api;

    public void queryCats(String query, Callback<List<Cat>> catsCallback){
        api.queryCats(query, new Api.CatsQueryCallback() {
            @Override
            public void onCatListReceived(List<Cat> cats) {
                catsCallback.onResult(cats);
            }

            @Override
            public void onQueryFailed(Exception e) {
                catsCallback.onError(e);
            }
        });
    }

    public void store(Cat cat, Callback<Uri> uriCallback){
        api.store(cat, new Api.StoreCallback() {
            @Override
            public void onCatStored(Uri uri) {
                uriCallback.onResult(uri);
            }

            @Override
            public void onStoreFailed(Exception e) {
                uriCallback.onError(e);
            }
        });
    }
}
```
这样通过新的回调我们可以减少一次处理结果和异常的逻辑。    
最终，我们的`CatsHelper`如下:    
```java
public class CatsHelper{

    ApiWrapper apiWrapper;

    public void saveTheCutestCat(String query, Callback<Uri> cutestCatCallback){
        apiWrapper.queryCats(query, new Callback<List<Cat>>() {
            @Override
            public void onResult(List<Cat> cats) {
                Cat cutest = findCutest(cats);
                apiWrapper.store(cutest, cutestCatCallback);
            }

            @Override
            public void onError(Exception e) {
                cutestCatCallback.onError(e);
            }
        });
    }

    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

好了，现在比之前的代码稍微简单点了。但是我们能不能做的更好？ 当然可以！

####保持参数和回调的分离性



看看这些新的异步操作(`queryCats`,` store`和`saveTheCutestCat`)。这些函数都有同样的模式。使用一些参数来调用这些函数`(query,cat)`，同时还有一个回调接口作为参数。甚至，所有的异步操作都带有一些常规参数和一个额外的回调接口参数。如果我们把他们分离开会如何，让每个异步操作只有一些常规参数而把返回一个临时的对象来操作回调接口。
下面来试试看看这种方式能否有效。
如果我们返回一个临时的对象作为异步操作的回调接口处理方式，我们需要先定义这个对象。由于对象遵守通用的行为(有一个回调接口参数)，我们定义一个能用于所有操作的对象。我们称之为`AsyncJob`。

> 注意： 我非常想把这个名字称之为`AsyncTask`。但是由于`Android`系统已经有个`AsyncTask`类了， 为了避免混淆，所以就用`AsyncJob`了。

该对象如下:   
```java
public abstract class AsyncJob<T> {
    public abstract void start(Callback<T> callback);
}
```

`start()`函数有个`Callback`回调接口参数，并开始执行该操作。
`ApiWrapper`修改为：
```java
public class ApiWrapper {
    Api api;
 
    public AsyncJob<List<Cat>> queryCats(String query) {
        return new AsyncJob<List<Cat>>() {
            @Override
            public void start(Callback<List<Cat>> catsCallback) {
                api.queryCats(query, new Api.CatsQueryCallback() {
                    @Override
                    public void onCatListReceived(List<Cat> cats) {
                        catsCallback.onResult(cats);
                    }
 
                    @Override
                    public void onQueryFailed(Exception e) {
                        catsCallback.onError(e);
                    }
                });
            }
        };
    }
 
    public AsyncJob<Uri> store(Cat cat) {
        return new AsyncJob<Uri>() {
            @Override
            public void start(Callback<Uri> uriCallback) {
                api.store(cat, new Api.StoreCallback() {
                    @Override
                    public void onCatStored(Uri uri) {
                        uriCallback.onResult(uri);
                    }
 
                    @Override
                    public void onStoreFailed(Exception e) {
                        uriCallback.onError(e);
                    }
                });
            }
        };
    }
}
```
目前看起来还不错。现在可以使用`AsyncJob.start()`来启动每个操作了。接下来我们修改`CatsHelper`类：

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        return new AsyncJob<Uri>() {
            @Override
            public void start(Callback<Uri> cutestCatCallback) {
                apiWrapper.queryCats(query)
                        .start(new Callback<List<Cat>>() {
                            @Override
                            public void onResult(List<Cat> cats) {
                                Cat cutest = findCutest(cats);
                                apiWrapper.store(cutest)
                                        .start(new Callback<Uri>() {
                                            @Override
                                            public void onResult(Uri result) {
                                                cutestCatCallback.onResult(result);
                                            }
 
                                            @Override
                                            public void onError(Exception e) {
                                                cutestCatCallback.onError(e);
                                            }
                                        });
                            }
 
                            @Override
                            public void onError(Exception e) {
                                cutestCatCallback.onError(e);
                            }
                        });
            }
        };
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

看起来比前面一个版本更加复杂啊，这样有啥好处啊？
这里其实我们返回的是一个`AsyncJob`对象，该对象和客户端代码组合使用，这样在`Activity`或者`Fragment`客户端代码中就可以操作这个返回的对象了。
代码虽然目前看起来比较复杂，下面我们就来改进一下。

####分解


下面是流程图:   
```java
         (async)                 (sync)           (async)
query ===========> List<Cat> -------------> Cat ==========> Uri
       queryCats              findCutest           store
```

为了让代码具有可读性，我们把这个流程分解为每个操作。同时我们再进一步假设，如果一个操作是异步的，则每个调用该异步操作的函数也是异步的。例如：如果查询猫是个异步操作，则找到最可爱的猫操作也是异步的。

因此，我们可以使用`AsyncJob`来把这些操作分解为一些小的操作中。

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = new AsyncJob<Cat>() {
            @Override
            public void start(Callback<Cat> callback) {
                catsListAsyncJob.start(new Callback<List<Cat>>() {
                    @Override
                    public void onResult(List<Cat> result) {
                        callback.onResult(findCutest(result));
                    }
 
                    @Override
                    public void onError(Exception e) {
                        callback.onError(e);
                    }
                });
            }
        };
 
        AsyncJob<Uri> storedUriAsyncJob = new AsyncJob<Uri>() {
            @Override
            public void start(Callback<Uri> cutestCatCallback) {
                cutestCatAsyncJob.start(new Callback<Cat>() {
                    @Override
                    public void onResult(Cat cutest) {
                        apiWrapper.store(cutest)
                                .start(new Callback<Uri>() {
                                    @Override
                                    public void onResult(Uri result) {
                                        cutestCatCallback.onResult(result);
                                    }
 
                                    @Override
                                    public void onError(Exception e) {
                                        cutestCatCallback.onError(e);
                                    }
                                });
                    }
 
                    @Override
                    public void onError(Exception e) {
                        cutestCatCallback.onError(e);
                    }
                });
            }
        };
        return storedUriAsyncJob;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

虽然代码量多了，但是看起来更加清晰了。 嵌套的回调函数没那么多层级了，异步操作的名字也更容易理解了(`catsListAsyncJob`,`cutestCatAsyncJob`, `storedUriAsyncJob`)。
看起来还不错，但是还可以更好。

####简单的映射


先来看看我们创建 AsyncJob cutestCatAsyncJob 的代码：

```java
AsyncJob<Cat> cutestCatAsyncJob = new AsyncJob<Cat>() {
            @Override
            public void start(Callback<Cat> callback) {
                catsListAsyncJob.start(new Callback<List<Cat>>() {
                    @Override
                    public void onResult(List<Cat> result) {
                        callback.onResult(findCutest(result));
                    }
 
                    @Override
                    public void onError(Exception e) {
                        callback.onError(e);
                    }
                });
            }
        };
```

这 16 行代码中，只有一行代码是我们的业务逻辑代码：
```java
findCutest(result)
```
其他的代码只是为了启动`AsyncJob`并接收结果和处理异常的干扰代码。 但是这些代码是通用的，我们可以把他们放到其他地方来让我们更加专注业务逻辑代码。
那么如何实现呢？需要做两件事情：
- 通过`AsyncJob`获取需要转换的结果
- 转换的函数

但是由于`Java`的限制，无法把函数作为参数，所以需要用一个接口（或者类）并在里面定义一个转换函数：

```java
public interface Func<T, R> {
    R call(T t);
}
```
灰常简单。 有两个泛型类型定义，`T`代表参数的类型；`R`代表返回值的类型。

当我们把`AsyncJob`的结果转换为其他类型的时候，我们需要把一个结果值映射为另外一种类型，这个操作我们称之为`map`。 把该函数定义到`AsyncJob`类中比较方便，这样就可以通过`this`来访问`AsyncJob`对象了。

```java
public abstract class AsyncJob<T> {
    public abstract void start(Callback<T> callback);
 
    public <R> AsyncJob<R> map(Func<T, R> func){
        final AsyncJob<T> source = this;
        return new AsyncJob<R>() {
            @Override
            public void start(Callback<R> callback) {
                source.start(new Callback<T>() {
                    @Override
                    public void onResult(T result) {
                        R mapped = func.call(result);
                        callback.onResult(mapped);
                    }
 
                    @Override
                    public void onError(Exception e) {
                        callback.onError(e);
                    }
                });
            }
        };
    }
}
```

看起来不错， 现在的`CatsHelper`如下：

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = catsListAsyncJob.map(new Func<List<Cat>, Cat>() {
            @Override
            public Cat call(List<Cat> cats) {
                return findCutest(cats);
            }
        });
 
        AsyncJob<Uri> storedUriAsyncJob = new AsyncJob<Uri>() {
            @Override
            public void start(Callback<Uri> cutestCatCallback) {
                cutestCatAsyncJob.start(new Callback<Cat>() {
                    @Override
                    public void onResult(Cat cutest) {
                        apiWrapper.store(cutest)
                                .start(new Callback<Uri>() {
                                    @Override
                                    public void onResult(Uri result) {
                                        cutestCatCallback.onResult(result);
                                    }
 
                                    @Override
                                    public void onError(Exception e) {
                                        cutestCatCallback.onError(e);
                                    }
                                });
                    }
 
                    @Override
                    public void onError(Exception e) {
                        cutestCatCallback.onError(e);
                    }
                });
            }
        };
        return storedUriAsyncJob;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

新创建的`AsyncJob cutestCatAsyncJob()`的代码只有6行，并且只有一层嵌套。


####高级映射


但是`AsyncJob storedUriAsyncJob()`看起来还是非常糟糕。 这里也能使用映射吗？ 下面就来试试吧！

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = catsListAsyncJob.map(new Func<List<Cat>, Cat>() {
            @Override
            public Cat call(List<Cat> cats) {
                return findCutest(cats);
            }
        });
 
        AsyncJob<Uri> storedUriAsyncJob = cutestCatAsyncJob.map(new Func<Cat, Uri>() {
            @Override
            public Uri call(Cat cat) {
                return apiWrapper.store(cat);
        //      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 将会导致无法编译
        //      Incompatible types:
        //      Required: Uri
        //      Found: AsyncJob<Uri>                
            }
        });
        return storedUriAsyncJob;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```
哎。。。 看起来没这么简单啊， 下面修复返回的类型再试一次：

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = catsListAsyncJob.map(new Func<List<Cat>, Cat>() {
            @Override
            public Cat call(List<Cat> cats) {
                return findCutest(cats);
            }
        });
 
        AsyncJob<AsyncJob<Uri>> storedUriAsyncJob = cutestCatAsyncJob.map(new Func<Cat, AsyncJob<Uri>>() {
            @Override
            public AsyncJob<Uri> call(Cat cat) {
                return apiWrapper.store(cat);
            }
        });
        return storedUriAsyncJob;
        //^^^^^^^^^^^^^^^^^^^^^^^ 将会导致无法编译
        //      Incompatible types:
        //      Required: AsyncJob<Uri>
        //      Found: AsyncJob<AsyncJob<Uri>>
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

这里我们只能拿到`AsyncJob<AsyncJob>` 。看来还需要更进一步。我们需要压缩一层`AsyncJob`，把两个异步操作当做一个单一的异步操作来对待。
现在我们需要一个参数为`AsyncJob`的`map`转换操作而不是`R`。该操作类似于`map`，但是该操作会把嵌套的`AsyncJob`压缩为`flatten`一层`AsyncJob`. 我们称之为`flatMap`，实现如下：

```java
public abstract class AsyncJob<T> {
    public abstract void start(Callback<T> callback);
 
    public <R> AsyncJob<R> map(Func<T, R> func){
        final AsyncJob<T> source = this;
        return new AsyncJob<R>() {
            @Override
            public void start(Callback<R> callback) {
                source.start(new Callback<T>() {
                    @Override
                    public void onResult(T result) {
                        R mapped = func.call(result);
                        callback.onResult(mapped);
                    }
 
                    @Override
                    public void onError(Exception e) {
                        callback.onError(e);
                    }
                });
            }
        };
    }
 
    public <R> AsyncJob<R> flatMap(Func<T, AsyncJob<R>> func){
        final AsyncJob<T> source = this;
        return new AsyncJob<R>() {
            @Override
            public void start(Callback<R> callback) {
                source.start(new Callback<T>() {
                    @Override
                    public void onResult(T result) {
                        AsyncJob<R> mapped = func.call(result);
                        mapped.start(new Callback<R>() {
                            @Override
                            public void onResult(R result) {
                                callback.onResult(result);
                            }
 
                            @Override
                            public void onError(Exception e) {
                                callback.onError(e);
                            }
                        });
                    }
 
                    @Override
                    public void onError(Exception e) {
                        callback.onError(e);
                    }
                });
            }
        };
    }
}
```

看起来有很多干扰代码，但是还好这些代码在客户端代码中并不会出现。 现在我们的`CatsHelper`如下：

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = catsListAsyncJob.map(new Func<List<Cat>, Cat>() {
            @Override
            public Cat call(List<Cat> cats) {
                return findCutest(cats);
            }
        });
 
        AsyncJob<Uri> storedUriAsyncJob = cutestCatAsyncJob.flatMap(new Func<Cat, AsyncJob<Uri>>() {
            @Override
            public AsyncJob<Uri> call(Cat cat) {
                return apiWrapper.store(cat);
            }
        });
        return storedUriAsyncJob;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

如果把匿名类修改为`Java 8`的`lambdas`表达式（逻辑是一样的，只是让代码看起来更清晰点）就很容易发现了。

```java
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public AsyncJob<Uri> saveTheCutestCat(String query) {
        AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
        AsyncJob<Cat> cutestCatAsyncJob = catsListAsyncJob.map(cats -> findCutest(cats));
        AsyncJob<Uri> storedUriAsyncJob = cutestCatAsyncJob.flatMap(cat -> apiWrapper.store(cat));
        return storedUriAsyncJob;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

这样看起来是不是就很清晰了。 这个代码和刚刚开头的阻塞式代码是不是非常相似：

```java
public class CatsHelper {
 
    Api api;
 
    public Uri saveTheCutestCat(String query){
        List<Cat> cats = api.queryCats(query);
        Cat cutest = findCutest(cats);
        Uri savedUri = api.store(cutest);
        return savedUri;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

现在他们不仅逻辑是一样的，语义上也是一样的。 太棒了！
同时我们还可以使用组合操作，现在把两个异步操作组合一起并返还另外一个异步操作。
异常处理也会传递到最终的回调接口中。
下面来看看`RxJava`吧。
你没必要把上面代码应用到您的项目中去， 这些简单的、线程不安全的代码只是 `RxJava`的一部分。
只有一些名字上的不同：

- `AsyncJob`等同于`Observable`，不仅仅可以返回一个结果，还可以返回一系列的结果，当然也可能没有结果返回。
- `Callback`等同于`Observer`，除了`onNext(T t)`, `onError(Throwable t)`以外，还有一个`onCompleted()`函数，该函数在结束继续返回结果的时候通知`Observable`。
- `abstract void start(Callback callback)`和`Subscription subscribe(final Observer observer)`类似，返回一个`Subscription`，如果你不再需要后面的结果了，可以取消该任务。

下面是`RxJava`版本的代码:    

```java
public class ApiWrapper {
    Api api;
 
    public Observable<List<Cat>> queryCats(final String query) {
        return Observable.create(new Observable.OnSubscribe<List<Cat>>() {
            @Override
            public void call(final Subscriber<? super List<Cat>> subscriber) {
                api.queryCats(query, new Api.CatsQueryCallback() {
                    @Override
                    public void onCatListReceived(List<Cat> cats) {
                        subscriber.onNext(cats);
                    }
 
                    @Override
                    public void onQueryFailed(Exception e) {
                        subscriber.onError(e);
                    }
                });
            }
        });
    }
 
    public Observable<Uri> store(final Cat cat) {
        return Observable.create(new Observable.OnSubscribe<Uri>() {
            @Override
            public void call(final Subscriber<? super Uri> subscriber) {
                api.store(cat, new Api.StoreCallback() {
                    @Override
                    public void onCatStored(Uri uri) {
                        subscriber.onNext(uri);
                    }
 
                    @Override
                    public void onStoreFailed(Exception e) {
                        subscriber.onError(e);
                    }
                });
            }
        });
    }
}
 
public class CatsHelper {
 
    ApiWrapper apiWrapper;
 
    public Observable<Uri> saveTheCutestCat(String query) {
        Observable<List<Cat>> catsListObservable = apiWrapper.queryCats(query);
        Observable<Cat> cutestCatObservable = catsListObservable.map(new Func1<List<Cat>, Cat>() {
            @Override
            public Cat call(List<Cat> cats) {
                return CatsHelper.this.findCutest(cats);
            }
        });
        Observable<Uri> storedUriObservable = cutestCatObservable.flatMap(new Func1<Cat, Observable<? extends Uri>>() {
            @Override
            public Observable<? extends Uri> call(Cat cat) {
                return apiWrapper.store(cat);
            }
        });
        return storedUriObservable;
    }
 
    private Cat findCutest(List<Cat> cats) {
        return Collections.max(cats);
    }
}
```

把 Observable 替换为 AsyncJob 后 他们的代码是一样的。

####结论


通过简单的转换操作，我们可以把异步操作抽象出来。这种抽象的结果可以像操作简单的阻塞函数一样来操作异步操作并组合异步操作。这样我们就可以摆脱层层嵌套的回调接口了，并且不用手工的去处理每次异步操作的异常。


上面这个例子非常好，建议多看几遍，加深理解，可能把这个例子放在这里并不太好，把它放到开始讲之前可能更容易理解，但是我觉得，介绍完概念、使用方法和基本的操作符后，我们可能并不能理解操作符的原理和作用。之前看完操作符原理后迷迷糊糊的状态再来看这个例子会豁然开朗。    

这里也感谢牛逼的作者[Yaroslav](https://github.com/yarikx)(也是`RxAndroid`项目的一个重要参与者)能用这么牛逼的例子，讲解的如此透彻。   


如果嫌上面的代码麻烦，可以通过下面的例子看:   

假设有这样一个需求：界面上有一个自定义的视图`imageCollectorView`，它的作用是显示多张图片，并能使用`addImage(Bitmap)` 方法来任意增加显示的图片。现在需要程序将一个给出的目录数组`File[] folders`中每个目录下的`png`图片都加载出来并显示在`imageCollectorView`中。需要注意的是，由于读取图片的这一过程较为耗时，需要放在后台执行，而图片的显示则必须在`UI`线程执行。常用的实现方式有多种，我这里贴出其中一种：


```java
new Thread() {
    @Override
    public void run() {
        super.run();
        for (File folder : folders) {
            File[] files = folder.listFiles();
            for (File file : files) {
                if (file.getName().endsWith(".png")) {
                    final Bitmap bitmap = getBitmapFromFile(file);
                    getActivity().runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            imageCollectorView.addImage(bitmap);
                        }
                    });
                }
            }
        }
    }
}.start();
```

而如果使用 RxJava ，实现方式是这样的：

```java
Observable.from(folders)
    .flatMap(new Func1<File, Observable<File>>() {
        @Override
        public Observable<File> call(File file) {
            return Observable.from(file.listFiles());
        }
    })
    .filter(new Func1<File, Boolean>() {
        @Override
        public Boolean call(File file) {
            return file.getName().endsWith(".png");
        }
    })
    .map(new Func1<File, Bitmap>() {
        @Override
        public Bitmap call(File file) {
            return getBitmapFromFile(file);
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) {
            imageCollectorView.addImage(bitmap);
        }
    });
```

那位说话了：『你这代码明明变多了啊！简洁个毛啊！』大兄弟你消消气，我说的是逻辑的简洁，不是单纯的代码量少（逻辑简洁才是提升读写代码速度的必杀技对不？）。观察一下你会发现， `RxJava`的这个实现，是一条从上到下的链式调用，没有任何嵌套，这在逻辑的简洁性上是具有优势的。当需求变得复杂时，这种优势将更加明显（试想如果还要求只选取前`10`张图片，常规方式要怎么办？如果有更多这样那样的要求呢？再试想，在这一大堆需求实现完两个月之后需要改功能，当你翻回这里看到自己当初写下的那一片迷之缩进，你能保证自己将迅速看懂，而不是对着代码重新捋一遍思路？）。

更多内容请看下一篇文章[RxJava详解(下)](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%8B).md)

参考:   

- [RxJava Wiki](https://github.com/ReactiveX/RxJava/wiki)
- [Grokking RxJava, Part 1: The Basics](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)
- [NotRxJava](https://yarikx.github.io/NotRxJava/)
- [When Not to Use RxJava](http://tomstechnicalblog.blogspot.hk/2016/07/when-not-to-use-rxjava.html)
- [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
- [Google Agera 从入门到放弃](http://blog.chengyunfeng.com/?p=984)

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 