RxJava详解(上)
===

年初的时候就想学习下`RxJava`然后写一些`RxJava`的教程，无奈发现已经年底了，然而我还么有写。今天有点时间，特别是发布了`RxJava 2.0`后，我决定动笔开始。

现在`RxJava`变的越来越流行了，很多项目中都使用了它。特别是大神`JakeWharton`等的加入，以及`RxBinding、Retrofit、RxLifecycle`等众多项目的，然开发越来越方便，但是上手比较难，不过一旦你入门后你就会发现真是太棒了。


在介绍`RxJava`之前，感觉有必要说一下什么是函数响应式编程(`FRP`)？     

函数响应式编程（`FRP`）为解决现代编程问题提供了全新的视角。一旦理解它，可以极大地简化你的项目，特别是处理嵌套回调的异步事件，复杂的列表过滤和变换，或者时间相关问题，而且`RxJava`是响应式编程的一个具体实现。

这里以一个真实的例子来开始讲解函数响应式编程怎么提高我们代码的可读性。我们的任务是通过查询`GitHub`的`API`， 首先获取用户列表，然后请求每个用户的详细信息。这个过程包括两个`web`服务端点:
`https://api.github.com/users`-获取用户列表；`https://api.github.com/users/{username}`－获取特定用户的详细信息，例如`https://api.github.com/users/mutexkid`。

普通情况下是这样写的:    
```java
//The "Nested Callbacks" Way
public void fetchUserDetails() {
    //first, request the users...
    mService.requestUsers(new Callback<GithubUsersResponse>() {
        @Override
        public void success(final GithubUsersResponse githubUsersResponse,
                            final Response response) {
            Timber.i(TAG, "Request Users request completed");
            final synchronized List<GithubUserDetail> githubUserDetails = new ArrayList<GithubUserDetail>();
            //next, loop over each item in the response
            for (GithubUserDetail githubUserDetail : githubUsersResponse) {
                //request a detail object for that user
                mService.requestUserDetails(githubUserDetail.mLogin,
                                            new Callback<GithubUserDetail>() {
                    @Override
                    public void success(GithubUserDetail githubUserDetail,
                                        Response response) {
                        Log.i("User Detail request completed for user : " + githubUserDetail.mLogin);
                        githubUserDetails.add(githubUserDetail);
                        if (githubUserDetails.size() == githubUsersResponse.mGithubUsers.size()) {
                            //we've downloaded'em all - notify all who are interested!
                            mBus.post(new UserDetailsLoadedCompleteEvent(githubUserDetails));
                        }
                    }

                    @Override
                    public void failure(RetrofitError error) {
                        Log.e(TAG, "Request User Detail Failed!!!!", error);
                    }
                });
            }
        }

        @Override
        public void failure(RetrofitError error) {
            Log.e(TAG, "Request User Failed!!!!", error);
        }
    });
}
```

尽管这不是最差的代码－至少它是异步的，因此在等待每个请求完成的时候不会阻塞－但由于代码复杂（增加更多层次的回调代码复杂度将呈指数级增长）因此远非理想的代码。
当我们不可避免要修改代码时（在前面的`web service`调用中，我们依赖前一次的回调状态，因此它不适用于模块化或者修改要传递给下一个回调的数据）也远非容易的工作。
我们亲切的称这种情况为“回调地狱”。

而通过`RxJava`的方式:    
```java
public void rxFetchUserDetails() {
    //request the users
    mService.rxRequestUsers().concatMap(Observable::from)
    .concatMap((GithubUser githubUser) ->
                    //request the details for each user
                    mService.rxRequestUserDetails(githubUser.mLogin)
    )
    //accumulate them as a list
    .toList()
    //define which threads information will be passed on
    .subscribeOn(Schedulers.newThread())
    .observeOn(AndroidSchedulers.mainThread())
    //post them on an eventbus
    .subscribe(githubUserDetails -> {
        EventBus.getDefault().post(new UserDetailsLoadedCompleteEvent(githubUserDetails));
    });
}
```

如你所见，使用函数响应式编程模型我们完全摆脱了回调，并最终得到了更短小的程序。让我们从函数响应式编程的基本定义开始慢慢解释到底发生了什么，并逐渐理解上面的代码，这些代码托管在`GitHub`上面。

从根本上讲，函数响应式编程是在观察者模式的基础上，增加对`Observables`发送的数据流进行操纵和变换的功能。

`RxJava`简介
---

在介绍`RxJava`之前先说一下`Rx`。`Rx`的全称是`Reactive Extensions`，直译过来就是响应式扩展。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rx_logo.png?raw=true)

`Rx`基于观察者模式，它是一种编程模型，目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流。`ReactiveX.io`给的定义是，`Rx`是一个使用可观察数据流进行异步编程的编程接口，`ReactiveX`结合了观察者模式、迭代器模式和函数式编程的精华。`Rx`已经渗透到了各个语言中，有了`Rx`所以才有了`RxJava`、`Rx.NET`、`RxJS`、`RxSwift`、`Rx.rb`、`RxPHP`等等，

这里先列举一下相关的官网:  

- [Rx](http://reactivex.io/)
- [RxJava](https://github.com/ReactiveX/RxJava)       
- [RxAndroid](https://github.com/ReactiveX/RxAndroid)

`RxJava`在`GitHub`上的介绍是：`a library for composing asynchronous and event-based programs by using observable sequences for the Java VM.`
翻译过来也就是一个基于事件和程序在`Java VM`上使用可观测的序列来组成异步的库。`RxJava`的本质就是一个实现异步操作的库，它的优势就是简洁，随着程序逻辑变得越来越复杂，它依然能够保持简洁。       

其实一句话总结一下`RxJava`的作用就是：异步   

这里可能会有人想不就是个异步吗，至于辣么矫情么?用`AsyncTask`、`Handler`甚至自定义一个`BigAsyncTask`分分钟搞定。

但是`RxJava`的好处是简洁。异步操作很关键的一点是程序的简洁性，因为在调度过程比较复杂的情况下，异步代码经常会既难写也难被读懂。 `Android`创造的`AsyncTask`和`Handler`其实都是为了让异步代码更加简洁。虽然`RxJava`的优势也是简洁，但它的简洁的与众不同之处在于，随着程序逻辑变得越来越复杂，它依然能够保持简洁。


#### 扩展的观察者模式


`RxJava`的异步实现，是通过一种扩展的观察者模式来实现的。

观察者模式面向的需求是：`A`对象（观察者）对`B`对象（被观察者）的某种变化高度敏感，需要在`B`变化的一瞬间做出反应。举个例子，新闻里喜闻乐见的警察抓小偷，警察需要在小偷伸手作案的时候实施抓捕。在这个例子里，警察是观察者，小偷是被观察者，警察需要时刻盯着小偷的一举一动，才能保证不会漏过任何瞬间。程序的观察者模式和这种真正的『观察』略有不同，观察者不需要时刻盯着被观察者（例如`A`不需要每过`2ms`就检查一次`B`的状态），而是采用注册(`Register`)或者称为订阅(`Subscribe`)的方式，告诉被观察者：我需要你的某某状态，你要在它变化的时候通知我。 `Android`开发中一个比较典型的例子是点击监听器`OnClickListener`。对设置`OnClickListener`来说，`View`是被观察者，`OnClickListener`是观察者，二者通过 `setOnClickListener()`方法达成订阅关系。订阅之后用户点击按钮的瞬间，`Android Framework`就会将点击事件发送给已经注册的`OnClickListener`。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。当然，这也得益于我们可以随意定制自己程序中的观察者和被观察者，而警察叔叔明显无法要求小偷『你在作案的时候务必通知我』。

`OnClickListener`的模式大致如下图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/btn_onclick.jpg?raw=true)

如图所示，通过`setOnClickListener()`方法，`Button`持有`OnClickListener`的引用（这一过程没有在图上画出）；当用户点击时，`Button`自动调用`OnClickListener`的`onClick()` 方法。另外，如果把这张图中的概念抽象出来（`Button` -> 被观察者、`OnClickListener` -> 观察者、`setOnClickListener()` -> 订阅，`onClick()` -> 事件），就由专用的观察者模式（例如只用于监听控件点击）转变成了通用的观察者模式。如下图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/btn_rxjava_observable.jpg?raw=true)

而`RxJava`作为一个工具库，使用的就是通用形式的观察者模式。


#### `RxJava`的观察者模式

 
`RxJava`的基本概念：

- `Observable`(可观察者，即被观察者)、 
- `Observer`(观察者)、 
- `subscribe()`(订阅)、事件。        
    `Observable`和`Observer`通过`subscribe()` 方法实现订阅关系，从而`Observable`可以在需要的时候发出事件来通知`Observer`。

与传统观察者模式不同，`RxJava`的事件回调方法除了普通事件`onNext()`(相当于`onClick()`/`onEvent()`)之外，还定义了两个特殊的事件：`onCompleted()`和`onError()`:     
但是`RxJava`与传统的观察者设计模式有一点明显不同，那就是如果一个`Observerble`没有任何的的`Subscriber`，那么这个`Observable`是不会发出任何事件的。


- `onCompleted()`: 事件队列完结。         
    `RxJava`不仅把每个事件单独处理，还会把它们看做一个队列。`RxJava`规定，当不会再有新的`onNext()`发出时，需要触发`onCompleted()` 方法作为标志。
- `onError()`: 事件队列异常。
    在事件处理过程中出异常时，`onError()`会被触发，同时队列自动终止，不允许再有事件发出。
- 在一个正确运行的事件序列中, `onCompleted()`和`onError()`有且只有一个，并且是事件序列中的最后一个。需要注意的是`onCompleted()`和`onError()` 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。


`RxJava`的观察者模式大致如下图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_observer_1.jpg?raw=true)


#### 基本实现


基于上面的概念， `RxJava`的基本实现主要有三点:    

- 创建`Observable`         
    `Observable`即被观察者，它决定什么时候触发事件以及触发怎样的事件。 `RxJava`使用`Observable.create()`方法来创建一个`Observable`，并为它定义事件触发规则。    

- 创建`Observer`即观察者，它决定事件触发的时候将有怎样的行为。       
    `RxJava`中的`Observer`接口的实现方式:    
   ```java
   Observer<String> observer = new Observer<String>() {
       @Override
       public void onNext(String s) {
           Log.d(tag, "Item: " + s);
       }

       @Override
       public void onCompleted() {
           Log.d(tag, "Completed!");
       }

       @Override
       public void onError(Throwable e) {
           Log.d(tag, "Error!");
       }
   };
   ```
    除了`Observer`接口之外，`RxJava`还内置了一个实现了`Observer`的抽象类:`Subscriber`。`Subscriber`对`Observer`接口进行了一些扩展，但他们的基本使用方式是完全一样的。

   ```java
   Subscriber<String> subscriber = new Subscriber<String>() {
       @Override
       public void onNext(String s) {
           Log.d(tag, "Item: " + s);
       }

       @Override
       public void onCompleted() {
           Log.d(tag, "Completed!");
       }

       @Override
       public void onError(Throwable e) {
           Log.d(tag, "Error!");
       }
   };
   ```
    不仅基本使用方式一样，实质上，在`RxJava`的`subscribe()`过程中，`Observer`也总是会先被转换成一个`Subscriber`再使用。所以如果你只想使用基本功能，选择`Observer`和 `Subscriber`是完全一样的。它们的区别对于使用者来说主要有两点：

    - `onStart()`: 这是`Subscriber`增加的方法。它会在`subscribe()`刚开始而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， `onStart()`就不适用了，因为它总是在`subscribe()` 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用`doOnSubscribe()`方法，具体可以在后面的文中看到。
    - `unsubscribe`(): 这是`Subscriber`所实现的另一个接口`Subscription`的方法，用于取消订阅。在这个方法被调用后，`Subscriber` 将不再接收事件。一般在这个方法调用前，可以使用`isUnsubscribed()`先判断一下状态。 `unsubscribe()`这个方法很重要，因为在`subscribe()`之后,`Observable`会持有 `Subscriber`的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如`onPause()、onStop()`等方法中）调用`unsubscribe()`来解除引用关系，以避免内存泄露的发生。
    所以后续讲解时我们有时会用`Subscriber`来代替`Observer`。

- 调用`subscribe()`方法(订阅)

    创建了一个`Observable`和`Observer`之后，再用`subscribe()`方法将它们联结起来，整条链子就可以工作了。代码形式很简单：

    ```java
    observable.subscribe(observer);  
    // 或者：
    observable.subscribe(subscriber);
    ```

    > 有人可能会注意到，`subscribe()`这个方法有点怪：它看起来是`observalbe`订阅了`observer/subscriber`而不是`observer/subscriber`订阅了`observalbe`，这看起来就像『杂志订阅了读者』一样颠倒了对象关系。这让人读起来有点别扭，不过如果把`API`设计成`observer.subscribe(observable)/subscriber.subscribe(observable)` ，虽然更加符合思维逻辑，但对流式`API`的设计就造成影响了，比较起来明显是得不偿失的。

  
`RxJava`入门示例
---

一个`Observable`可以发出零个或者多个事件，知道结束或者出错。每发出一个事件，就会调用它的`Subscriber`的`onNext`方法，最后调用`Subscriber.onComplete()`或者`Subscriber.onError()`结束。

#### `Hello World`


```
compile 'io.reactivex:rxandroid:1.2.1'
// Because RxAndroid releases are few and far between, it is recommended you also
// explicitly depend on RxJava's latest version for bug fixes and new features.
compile 'io.reactivex:rxjava:1.2.3'
```

```java
// 创建被观察者、数据源
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        // 可以看到，这里传入了一个 OnSubscribe 对象作为参数。OnSubscribe 会被存储在返回的 Observable 对象中，它的作用相当于一个计划表，当 Observable      
        //被订阅的时候，OnSubscribe 的 call() 方法会自动被调用，事件序列就会依照设定依次触发（对于上面的代码，就是观察者Subscriber 将会被调用三次 onNext() 和一次 
        // onCompleted()）。这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。
        Log.i("@@@", "call");
        subscriber.onNext("Hello ");
        subscriber.onNext("World !");
        subscriber.onCompleted();
    }
});
// 创建观察者
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onCompleted() {
        Log.i("@@@", "onCompleted");
    }

    @Override
    public void onError(Throwable e) {
        Log.i("@@@", "onError");
    }

    @Override
    public void onNext(String s) {
        Log.i("@@@", "onNext : " + s);
    }
};
// 关联或者叫订阅更合适。
observable.subscribe(subscriber);
```

一旦`subscriber`订阅了`observable`，`observable`就会调用`subscriber`对象的`onNext`和`onComplete`方法，`subscriber`就会打印出`Hello World`.        

 `Observable.subscribe(Subscriber)`的内部实现是这样的（仅核心代码）：

```java
// 注意：这不是`subscribe()`的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
public Subscription subscribe(Subscriber subscriber) {
   subscriber.onStart();
   onSubscribe.call(subscriber);
   return subscriber;
}
```

可以看到`subscriber()`做了3件事：

- 调用`Subscriber.onStart()`。这个方法在前面已经介绍过，是一个可选的准备方法。
- 调用`Observable`中的`onSubscribe.call(Subscriber)`。在这里，事件发送的逻辑开始运行。从这也可以看出，在`RxJava`中，`Observable` 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当`subscribe()`方法执行的时候。
- 将传入的`Subscriber`作为`Subscription`返回。这是为了方便`unsubscribe()`.

整个过程中对象间的关系如下图：
    
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_observable_list.gif?raw=true)


讲到这里很多人肯定会骂傻`X`,这TM简洁你妹啊...，这里只是个入门`Hello World`，真正的简洁等你看完全部介绍后就明白了。

`RxJava`内置了很多简化创建`Observable`对象的函数，比如`Observable.just()`就是用来创建只发出一个事件就结束的`Observable`对象，上面创建`Observable`对象的代码可以简化为一行

```java
Observable<String> observable = Observable.just("Hello ", "World !");
```

接下来看看如何简化`Subscriber`，上面的例子中，我们其实并不关心`onComplete()`和`onError`，我们只需要在`onNext`的时候做一些处理，这时候就可以使用`Action1`类。

```java
Action1<String> action1 = new Action1<String>() {
    @Override
    public void call(String s) {
        Log.i("@@@", "call : " + s);
    }
};
```

`Observable.subscribe()`方法有一个重载版本，接受三个`Action1`类型的参数      

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_subscribe_params.png?raw=true)

所以上面的代码最终可以写成这样:   

```java
Observable.just("Hello ", "World !").subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.i("@@@", "call : " + s);
    }
});
```

这里顺便多提一些`subscribe()`的多个`Action`参数：   
```java
Action1<String> onNextAction = new Action1<String>() {
    // onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    // onError()
    @Override
    public void call(Throwable throwable) {
        // Error handling
    }
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {
        Log.d(tag, "completed");
    }
};

observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```

简单解释一下这段代码中出现的`Action1`和`Action0`。`Action0`是`RxJava`的一个接口，它只有一个方法`call()`，这个方法是无参无返回值的；由于`onCompleted()` 方法也是无参无返回值的，因此`Action0`可以被当成一个包装对象，将`onCompleted()`的内容打包起来将自己作为一个参数传入`subscribe()`以实现不完整定义的回调。这样其实也可以看做将 `onCompleted()`方法作为参数传进了`subscribe()`，相当于其他某些语言中的『闭包』。`Action1`也是一个接口，它同样只有一个方法`call(T param)`，这个方法也无返回值，但有一个参数；与`Action0`同理，由于`onNext(T obj)`和`onError(Throwable error)`也是单参数无返回值的，因此`Action1`可以将`onNext(obj)`和`onError(error)`打包起来传入`subscribe()`以实现不完整定义的回调。事实上，虽然`Action0`和`Action1`在`API`中使用最广泛，但`RxJava`是提供了多个`ActionX`形式的接口(例如`Action2`, `Action3`)的，它们可以被用以包装不同的无返回值的方法。


假设我们的`Observable`是第三方提供的，它提供了大量的用户数据给我们，而我们需要从用户数据中筛选部分有用的信息，那我们该怎么办呢？    
从`Observable`中去修改肯定是不现实的？那从`Subscriber`中进行修改呢？ 这样好像是可以完成的。但是这种方式并不好，因为我们希望`Subscriber`越轻量越好，因为很有可能我们需要
在主线程中去执行`Subscriber`。另外，根据响应式函数编程的概念，`Subscribers`更应该做的事情是`响应`，响应`Observable`发出的事件，而不是去修改。
那该怎么办呢？ 这就要用到下面的部分要讲的操作符。  



接口变化
---

`RxJava 2.x`拥有了新的特性，其依赖于4个基础接口，它们分别是:     
- `Publisher`
- `Subscriber`
- `Subscription`
- `Processor`
其中最核心的莫过于`Publisher`和`Subscriber`。`Publisher`可以发出一系列的事件，而`Subscriber`负责和处理这些事件。

其中用的比较多的自然是`Publisher`的`Flowable`，它支持背压(`backpressure`)。关于背压给个简洁的定义就是: 

> 背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。

简而言之，背压是流速控制的一种策略。
其实`RxJava 2.x`最大的改动就是对于`backpressure`的处理，为此将原来的`Observable`拆分成了新的`Observable`和`Flowable`，同时其他相关部分也同时进行了拆分。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava1vs2.png?raw=true)


在`RxJava 2.x`中，`Observable`用于订阅`Observer`，不再支持背压（`1.x`中可以使用背压策略），而`Flowable`用于订阅`Subscriber`，是支持背压的。

操作符(`Operators`)
---

`RxJava`提供了对事件序列进行变换的支持，这是它的核心功能之一.所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。       
操作符就是为了解决对`Observable`对象的变换的问题，操作符用于在`Observable`和最终的`Subscriber`之间修改`Observable`发出的事件。`RxJava`提供了很多很有用的操作符。
比如`map`操作符，就是用来把把一个事件转换为另一个事件的。

#### `map`


`Returns an Observable that applies a specified function to each item emitted by the source Observable and emits the results of these function applications.`

```java
Observable<String> just = Observable.just("Hello ", "World !");
Observable<String> map = just.map(new Func1<String, String>() {
    @Override
    public String call(String s) {
        return s + "@@@";
    }
});
map.subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.i("@@@", s);
    }
});
```
上面的代码打印出的结果是:   
```
12-12 15:51:22.184 472-472/com.charon.rxjavastudydemo I/@@@: Hello @@@
12-12 15:51:22.184 472-472/com.charon.rxjavastudydemo I/@@@: World !@@@
```

`map()`操作符就是用于变换`Observable`对象的，`map`操作符返回一个`Observable`对象，这样就可以实现链式调用，在一个`Observable`对象上多次使用map操作符，最终将最简洁的数据传递给`Subscriber`对象。

`map`操作符更有趣的一点是它不必返回Observable对象返回的类型，你可以使用`map`操作符返回一个发出新的数据类型的`Observable`对象。
比如上面的例子中，`Subscriber`并不关心返回的字符串，而是想要字符串的`hash`值。


```java
Observable<String> just = Observable.just("Hello ", "World !");
Observable<Integer> map = just.map(new Func1<String, Integer>() {
    @Override
    public Integer call(String s) {
        return s.hashCode();
    }
});
map.subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.i("@@@", "" + integer);
    }
});
```

上面部分的打印结果是:    
```
12-12 15:54:35.515 8521-8521/com.charon.rxjavastudydemo I/@@@: -2137068114
12-12 15:54:35.516 8521-8521/com.charon.rxjavastudydemo I/@@@: -1105126669
```
或者也可以通过`map`执行网络请求:    

- 通过`Observable.create()`方法，调用`OkHttp`网络请求；
- 通过`map`操作符集合`gson`，将`Response`转换为`bean`类；
- 通过`doOnNext()`方法，解析`bean`中的数据，并进行数据库存储等操作；
- 调度线程，在子线程中进行耗时操作任务，在主线程中更新`UI`；
- 通过`subscribe()`，根据请求成功或者失败来更新`UI`。

```java

×
这可能是最好的RxJava 2.x 教程（完结版）
96  nanchen2251 
2017.07.03 15:17* 字数 3856 阅读 108192评论 71喜欢 414赞赏 4
这可能是最好的 RxJava 2.x 入门教程系列专栏
文章链接：
这可能是最好的RxJava 2.x 入门教程（一）
这可能是最好的RxJava 2.x 入门教程（二）
这可能是最好的RxJava 2.x 入门教程（三）
这可能是最好的RxJava 2.x 入门教程（四）
这可能是最好的RxJava 2.x 入门教程（五）
GitHub 代码同步更新：https://github.com/nanchen2251/RxJava2Examples
为了满足大家的饥渴难耐，GitHub将同步更新代码，主要包含基本的代码封装，RxJava 2.x所有操作符应用场景介绍和实际应用场景，后期除了RxJava可能还会增添其他东西，总之，GitHub上的Demo专为大家倾心打造。传送门：https://github.com/nanchen2251/RxJava2Examples

为什么要学 RxJava？
提升开发效率，降低维护成本一直是开发团队永恒不变的宗旨。近两年来国内的技术圈子中越来越多的开始提及 RxJava ，越来越多的应用和面试中都会有 RxJava ，而就目前的情况，Android 的网络库基本被 Retrofit + OkHttp 一统天下了，而配合上响应式编程 RxJava 可谓如鱼得水。想必大家肯定被近期的 Kotlin 炸开了锅，笔者也在闲暇之时去了解了一番（作为一个与时俱进的有理想的青年怎么可能不与时俱进？），发现其中有个非常好的优点就是简洁，支持函数式编程。是的， RxJava 最大的优点也是简洁，但它不止是简洁，而且是** 随着程序逻辑变得越来越复杂，它依然能够保持简洁 **（这货洁身自好呀有木有）。

咳咳，要例子，猛戳这里：给 Android 开发者的 RxJava 详解
什么是响应式编程
上面我们提及了响应式编程，不少新司机对它可谓一脸懵逼，那什么是响应式编程呢？响应式编程是一种基于异步数据流概念的编程模式。数据流就像一条河：它可以被观测，被过滤，被操作，或者为新的消费者与另外一条流合并为一条新的流。

响应式编程的一个关键概念是事件。事件可以被等待，可以触发过程，也可以触发其它事件。事件是唯一的以合适的方式将我们的现实世界映射到我们的软件中：如果屋里太热了我们就打开一扇窗户。同样的，当我们的天气app从服务端获取到新的天气数据后，我们需要更新app上展示天气信息的UI；汽车上的车道偏移系统探测到车辆偏移了正常路线就会提醒驾驶者纠正，就是是响应事件。

今天，响应式编程最通用的一个场景是UI：我们的移动App必须做出对网络调用、用户触摸输入和系统弹框的响应。在这个世界上，软件之所以是事件驱动并响应的是因为现实生活也是如此。

为什么出了一个系列后还有完结版？
RxJava 这些年可谓越来越流行，而在去年的晚些时候发布了2.0正式版。大半年已过，虽然网上已经出现了大部分的 RxJava 教程（其实细心的你还是会发现 1.x 的超级多），前些日子，笔者花了大约两周的闲暇之时写了 RxJava 2.x 系列教程，也得到了不少反馈，其中就有不少读者觉得每一篇的教程太短，抑或是希望更多的侧重适用场景的介绍，在这样的大前提下，这篇完结版教程就此诞生，仅供各位新司机采纳。

开始
RxJava 2.x 已经按照 Reactive-Streams specification 规范完全的重写了，maven也被放在了io.reactivex.rxjava2:rxjava:2.x.y 下，所以 RxJava 2.x 独立于 RxJava 1.x 而存在，而随后官方宣布的将在一段时间后终止对 RxJava 1.x 的维护，所以对于熟悉 RxJava 1.x 的老司机自然可以直接看一下 2.x 的文档和异同就能轻松上手了，而对于不熟悉的年轻司机，不要慌，本酱带你装逼带你飞，马上就发车，坐稳了：https://github.com/nanchen2251/RxJava2Examples

你只需要在 build.gradle 中加上：compile 'io.reactivex.rxjava2:rxjava:2.1.1'（2.1.1为写此文章时的最新版本）

接口变化
RxJava 2.x 拥有了新的特性，其依赖于4个基础接口，它们分别是

Publisher
Subscriber
Subscription
Processor
其中最核心的莫过于 Publisher 和 Subscriber。Publisher 可以发出一系列的事件，而 Subscriber 负责和处理这些事件。

其中用的比较多的自然是 Publisher 的 Flowable，它支持背压。关于背压给个简洁的定义就是：

背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。

简而言之，背压是流速控制的一种策略。有兴趣的可以看一下官方对于背压的讲解。

可以明显地发现，RxJava 2.x 最大的改动就是对于 backpressure 的处理，为此将原来的  Observable 拆分成了新的 Observable 和 Flowable，同时其他相关部分也同时进行了拆分，但令人庆幸的是，是它，是它，还是它，还是我们最熟悉和最喜欢的 RxJava。

观察者模式
大家可能都知道， RxJava 以观察者模式为骨架，在 2.0 中依旧如此。

不过此次更新中，出现了两种观察者模式：

Observable ( 被观察者 ) / Observer ( 观察者 )
Flowable （被观察者）/ Subscriber （观察者）



在 RxJava 2.x 中，Observable 用于订阅 Observer，不再支持背压（1.x 中可以使用背压策略），而 Flowable 用于订阅 Subscriber ， 是支持背压（Backpressure）的。

Observable
在 RxJava 1.x 中，我们最熟悉的莫过于 Observable 这个类了，笔者在刚刚使用 RxJava 2.x 的时候，创建了 一个 Observable，瞬间一脸懵逼有木有，居然连我们最最熟悉的 Subscriber 都没了，取而代之的是 ObservableEmmiter，俗称发射器。此外，由于没有了Subscriber的踪影，我们创建观察者时需使用 Observer。而 Observer 也不是我们熟悉的那个 Observer，又出现了一个 Disposable 参数带你装逼带你飞。

废话不多说，从会用开始，还记得 RxJava 的三部曲吗？



** 第一步：初始化 Observable **
** 第二步：初始化 Observer **
** 第三步：建立订阅关系 **



Observable.create(new ObservableOnSubscribe<Integer>() { // 第一步：初始化Observable
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                Log.e(TAG, "Observable emit 1" + "\n");
                e.onNext(1);
                Log.e(TAG, "Observable emit 2" + "\n");
                e.onNext(2);
                Log.e(TAG, "Observable emit 3" + "\n");
                e.onNext(3);
                e.onComplete();
                Log.e(TAG, "Observable emit 4" + "\n" );
                e.onNext(4);
            }
        }).subscribe(new Observer<Integer>() { // 第三步：订阅

            // 第二步：初始化Observer
            private int i;
            private Disposable mDisposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {      
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                i++;
                if (i == 2) {
                    // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
                    mDisposable.dispose();
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete" + "\n" );
            }
        });
不难看出，RxJava 2.x 与 1.x 还是存在着一些区别的。首先，创建 Observable 时，回调的是 ObservableEmitter ，字面意思即发射器，并且直接 throws Exception。其次，在创建的 Observer 中，也多了一个回调方法：onSubscribe，传递参数为Disposable，Disposable 相当于 RxJava 1.x 中的 Subscription， 用于解除订阅。可以看到示例代码中，在 i 自增到 2 的时候，订阅关系被切断。

07-03 14:24:11.663 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onSubscribe : false
07-03 14:24:11.664 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 1
07-03 14:24:11.665 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : value : 1
07-03 14:24:11.666 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 2
07-03 14:24:11.667 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : value : 2
07-03 14:24:11.668 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : isDisposable : true
07-03 14:24:11.669 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 3
07-03 14:24:11.670 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 4
当然，我们的 RxJava 2.x 也为我们保留了简化订阅方法，我们可以根据需求，进行相应的简化订阅，只不过传入对象改为了 Consumer。

Consumer 即消费者，用于接收单个值，BiConsumer 则是接收两个值，Function 用于变换对象，Predicate 用于判断。这些接口命名大多参照了 Java 8 ，熟悉 Java 8 新特性的应该都知道意思，这里也不再赘述。

线程调度
关于线程切换这点，RxJava 1.x 和 RxJava 2.x 的实现思路是一样的。这里简单的说一下，以便于我们的新司机入手。

subScribeOn
同 RxJava 1.x 一样，subscribeOn 用于指定 subscribe() 时所发生的线程，从源码角度可以看出，内部线程调度是通过 ObservableSubscribeOn来实现的。

    @SchedulerSupport(SchedulerSupport.CUSTOM)
    public final Observable<T> subscribeOn(Scheduler scheduler) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
    }
ObservableSubscribeOn 的核心源码在 subscribeActual 方法中，通过代理的方式使用 SubscribeOnObserver 包装 Observer 后，设置 Disposable 来将 subscribe 切换到 Scheduler 线程中。

observeOn
observeOn 方法用于指定下游 Observer 回调发生的线程。

   @SchedulerSupport(SchedulerSupport.CUSTOM)
    public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        ObjectHelper.verifyPositive(bufferSize, "bufferSize");
        return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
    }
线程切换需要注意的
RxJava 内置的线程调度器的确可以让我们的线程切换得心应手，但其中也有些需要注意的地方。

简单地说，subscribeOn() 指定的就是发射事件的线程，observerOn 指定的就是订阅者接收事件的线程。
多次指定发射事件的线程只有第一次指定的有效，也就是说多次调用 subscribeOn() 只有第一次的有效，其余的会被忽略。
但多次指定订阅者接收线程是可以的，也就是说每调用一次 observerOn()，下游的线程就会切换一次。

Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                Log.e(TAG, "Observable thread is : " + Thread.currentThread().getName());
                e.onNext(1);
                e.onComplete();
            }
        }).subscribeOn(Schedulers.newThread())
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "After observeOn(mainThread)，Current thread is " + Thread.currentThread().getName());
                    }
                })
                .observeOn(Schedulers.io())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "After observeOn(io)，Current thread is " + Thread.currentThread().getName());
                    }
                });
输出：

07-03 14:54:01.177 15121-15438/com.nanchen.rxjava2examples E/RxThreadActivity: Observable thread is : RxNewThreadScheduler-1
07-03 14:54:01.178 15121-15121/com.nanchen.rxjava2examples E/RxThreadActivity: After observeOn(mainThread)，Current thread is main
07-03 14:54:01.179 15121-15439/com.nanchen.rxjava2examples E/RxThreadActivity: After observeOn(io)，Current thread is RxCachedThreadScheduler-2
实例代码中，分别用 Schedulers.newThread() 和 Schedulers.io() 对发射线程进行切换，并采用 observeOn(AndroidSchedulers.mainThread() 和 Schedulers.io() 进行了接收线程的切换。可以看到输出中发射线程仅仅响应了第一个 newThread，但每调用一次 observeOn() ，线程便会切换一次，因此如果我们有类似的需求时，便知道如何处理了。

RxJava 中，已经内置了很多线程选项供我们选择，例如有：

Schedulers.io() 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作；
Schedulers.computation() 代表CPU计算密集型的操作, 例如需要大量计算的操作；
Schedulers.newThread() 代表一个常规的新线程；
AndroidSchedulers.mainThread() 代表Android的主线程
这些内置的 Scheduler 已经足够满足我们开发的需求，因此我们应该使用内置的这些选项，而 RxJava 内部使用的是线程池来维护这些线程，所以效率也比较高。

操作符
关于操作符，在官方文档中已经做了非常完善的讲解，并且笔者前面的系列教程中也着重讲解了绝大多数的操作符作用，这里受于篇幅限制，就不多做赘述，只挑选几个进行实际情景的讲解。

map

map 操作符可以将一个 Observable 对象通过某种关系转换为另一个Observable 对象。在 2.x 中和 1.x 中作用几乎一致，不同点在于：2.x 将 1.x 中的 Func1 和 Func2 改为了 Function 和 BiFunction。

采用 map 操作符进行网络数据解析
想必大家都知道，很多时候我们在使用 RxJava 的时候总是和 Retrofit 进行结合使用，而为了方便演示，这里我们就暂且采用 OkHttp3 进行演示，配合 map，doOnNext ，线程切换进行简单的网络请求：
1）通过 Observable.create() 方法，调用 OkHttp 网络请求；
2）通过 map 操作符集合 gson，将 Response 转换为 bean 类；
3）通过 doOnNext() 方法，解析 bean 中的数据，并进行数据库存储等操作；
4）调度线程，在子线程中进行耗时操作任务，在主线程中更新 UI ；
5）通过 subscribe()，根据请求成功或者失败来更新 UI 。
```java
Observable.create(new ObservableOnSubscribe<Response>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Response> e) throws Exception {
        Builder builder = new Builder()
                .url(mUrl)
                .get();
        Request request = builder.build();
        Call call = new OkHttpClient().newCall(request);
        Response response = call.execute();
        e.onNext(response);
    }
}).map(new Function<Response, MobileAddress>() {
    @Override
    public MobileAddress apply(@NonNull Response response) throws Exception {
        if (response.isSuccessful()) {
            ResponseBody body = response.body();
            if (body != null) {
                Log.e(TAG, "map:转换前:" + response.body());
                return new Gson().fromJson(body.string(), MobileAddress.class);
            }
        }
        return null;
    }
}).observeOn(AndroidSchedulers.mainThread())
.doOnNext(new Consumer<MobileAddress>() {
    @Override
    public void accept(@NonNull MobileAddress s) throws Exception {
        Log.e(TAG, "doOnNext: 保存成功：" + s.toString() + "\n");
    }
}).subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(new Consumer<MobileAddress>() {
    @Override
    public void accept(@NonNull MobileAddress data) throws Exception {
        Log.e(TAG, "成功:" + data.toString() + "\n");
}, new Consumer<Throwable>() {
    @Override
    public void accept(@NonNull Throwable throwable) throws Exception {
        Log.e(TAG, "失败：" + throwable.getMessage() + "\n");
    }
});
```

`map()`的示意图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_map.jpg?raw=true)

通过上面的部分我们可以得知:   

- `Observable`和`Subscriber`可以做任何事情
    `Observable`可以是一个数据库查询，`Subscriber`用来显示查询结果；`Observable`可以是屏幕上的点击事件，`Subscriber`用来响应点击事件；`Observable`可以是一个网络请求，`Subscriber`用来显示请求结果。

- `Observable`和`Subscriber`是独立于中间的变换过程的。
    在`Observable`和`Subscriber`中间 可以增减任何数量的`map`。整个系统是高度可组合的，操作数据是一个很简单的过程。


#### `flatmap`


`Returns an Observable that emits items based on applying a function that you supply to each item emitted 
 by the source Observable, where that function returns an Observable, and then merging those resulting 
 Observables and emitting the results of this merger.`

`flatMap()`是一个很有用但非常难理解的变换，首先假设这么一种需求：假设有一个数据结构『学生』，现在需要打印出一组学生的名字。实现方式很简单：

```java
Student[] students = ...;
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String name) {
        Log.d(tag, name);
    }
    ...
};
Observable.from(students)
    .map(new Func1<Student, String>() {
        @Override
        public String call(Student student) {
            return student.getName();
        }
    })
    .subscribe(subscriber);
```

很简单。那么再假设：如果要打印出每个学生所需要修的所有课程的名称呢?(需求的区别在于，每个学生只有一个名字，但却有多个课程)首先可以这样实现：

```java
Student[] students = ...;
Subscriber<Student> subscriber = new Subscriber<Student>() {
    @Override
    public void onNext(Student student) {
        List<Course> courses = student.getCourses();
        for (int i = 0; i < courses.size(); i++) {
            Course course = courses.get(i);
            Log.d(tag, course.getName());
        }
    }
    ...
};
Observable.from(students)
    .subscribe(subscriber);
```

依然很简单。那么如果我不想在`Subscriber`中使用`for`循环，而是希望`Subscriber`中直接传入单个的`Course`对象呢(这对于代码复用很重要),用`map()`显然是不行的，因为`map()` 是一对一的转化，而我现在的要求是一对多的转化。那怎么才能把一个`Student`转化成多个`Course`呢？

这个时候，就需要用`flatMap()`了：
```java
Student[] students = ...;
Subscriber<Course> subscriber = new Subscriber<Course>() {
    @Override
    public void onNext(Course course) {
        Log.d(tag, course.getName());
    }
    ...
};
Observable.from(students)
    .flatMap(new Func1<Student, Observable<Course>>() {
        @Override
        public Observable<Course> call(Student student) {
            return Observable.from(student.getCourses());
        }
    })
    .subscribe(subscriber);
```

`map`与`flatmap`在功能上是一致的,它也是把传入的参数转化之后返回另一个对象。区别在于`flatmap`是通过中间`Observable`来进行，而`map`是直接执行.`flatMap()`中返回的是个 `Observable`对象，并且这个`Observable`对象并不是被直接发送到了`Subscriber`的回调方法中。 

`flatMap()`的原理是这样的:  

- 使用传入的事件对象创建一个`Observable`对象
- 并不发送这个`Observable`而是将它激活，于是它开始发送事件
- 每一个创建出来的`Observable`发送的事件，都被汇入同一个`Observable`,而这个`Observable`负责将这些事件统一交给`Subscriber` 的回调方法。

这三个步骤，把事件拆成了两级，通过一组新创建的`Observable`将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是`flatMap()`所谓的`flat`。

`flatMap()`就是根据你的规则，将`Observable`转换之后再发射出去，注意最后的顺序很可能是错乱的，如果要保证顺序的一致性，要使用`concatMap()`     
由于可以在嵌套的`Observable`中添加异步代码`flatMap()`也常用于嵌套的异步操作，例如嵌套的网络请求`(Retrofit + RxJava)`

`flatMap()`示意图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_flatmap.jpg?raw=true)


#### `throttleFirst()`


在每次事件触发后的一定时间间隔内丢弃新的事件。常用作去抖动过滤。 
例如按钮的点击监听器:    
```java
RxView.clickEvents(button); // RxBinding 代码`
    .throttleFirst(500, TimeUnit.MILLISECONDS) // 设置防抖间隔为 500ms 
    .subscribe(subscriber); // 妈妈再也不怕我的用户手抖点开两个重复的界面啦。
```

#### `from`


`convert various other objects and data types into Observables`

`from()`接收一个集合作为输入，然后每次输出一个元素给`subscriber`.

```java
List<String> s = Arrays.asList("Java", "Android", "Ruby", "Ios", "Swift");
Observable.from(s).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.i("@@@", s);
    }
});
```

#### `filter`


返回满足过滤条件的数据。  

```java
Observable.from(new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9})
                .filter(new Func1<Integer, Boolean>() {
                    @Override
                    public Boolean call(Integer integer) {
                        return integer < 5;
                    }
                })
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        Log.i("@@@", "integer=" + integer); //1,2,3,4
                    }
                });
```

#### `timer`


`Timer`会在指定时间后发射一个数字0，该操作符运行在`Computation Scheduler`。

```java
Observable.timer(3, TimeUnit.SECONDS).observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long aLong) {
                        Log.i("@@@", "aLong=" + aLong); // 延时3s
                    }
                });
```

#### `interval`


创建一个按固定时间间隔发射整数序列的`Observable`.
`interval`默认在`computation`调度器上执行。

```java
Observable.interval(1, TimeUnit.SECONDS, AndroidSchedulers.mainThread())
        .subscribe(new Action1<Long>() {
            @Override
            public void call(Long aLong) {
                Log.i("@@@", "aLong=" + aLong); //从0递增，间隔1s 0,1,2,3....
            }
        });
```

#### `Repeat`


重复执行 

等等...就不继续介绍了，到时候查下文档就好了。

是不是感觉没什么乱用，那就继续看下一部分吧。

更多内容请看下一篇文章[RxJava详解(中)][1]

参考:   

- [RxJava Wiki](https://github.com/ReactiveX/RxJava/wiki)
- [Grokking RxJava, Part 1: The Basics](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)
- [NotRxJava](https://yarikx.github.io/NotRxJava/)
- [When Not to Use RxJava](http://tomstechnicalblog.blogspot.hk/2016/07/when-not-to-use-rxjava.html)
- [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
- [Google Agera 从入门到放弃](http://blog.chengyunfeng.com/?p=984)


[1]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%AD).md "RxJava详解(中)"

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
