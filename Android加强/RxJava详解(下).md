RxJava详解(下)
===

变换的原理`lift()`
---

这些变换虽然功能各有不同，但实质上都是针对事件序列的处理和再发送。而在`RxJava`的内部，它们是基于同一个基础的变换方法：`lift()`。

首先看一下`lift()` 的内部实现（仅核心代码）：

```java
// 注意：这不是 lift() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
public <R> Observable<R> lift(Operator<? extends R, ? super T> operator) {
    return Observable.create(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber subscriber) {
            Subscriber newSubscriber = operator.call(subscriber);
            newSubscriber.onStart();
            onSubscribe.call(newSubscriber);
        }
    });
}
```

这段代码很有意思：它生成了一个新的`Observable`并返回，而且创建新`Observable`所用的参数`OnSubscribe的回调方法`call()`中的实现竟然看起来和前面讲过的`Observable.subscribe()`一样！然而它们并不一样哟~不一样的地方关键就在于第二行`onSubscribe.call(subscriber)`中的`onSubscribe` 所指代的对象不同（高能预警：接下来的几句话可能会导致身体的严重不适）

- `subscribe()`中这句话的`onSubscribe`指的是`Observable`中的`onSubscribe`对象，这个没有问题，但是`lift()`之后的情况就复杂了点。
- 当含有`lift()`时： 
    - `lift()`创建了一个`Observable`后，加上之前的原始`Observable`，已经有两个`Observable`了； 
    - 而同样地，新`Observable`里的新`OnSubscribe`加上之前的原始`Observable`中的原始`OnSubscribe`，也就有了两个`OnSubscribe`； 
    - 当用户调用经过`lift()`后的`Observable`的`subscribe()`的时候，使用的是`lift()`所返回的新的`Observable`，于是它所触发的`onSubscribe.call(subscriber)`，也是用的新`Observable`中的新`OnSubscribe`，即在`lift()`中生成的那个`OnSubscribe`； 
    - 而这个新`OnSubscribe`的`call()`方法中的`onSubscribe`，就是指的原始`Observable`中的原始`OnSubscribe`，在这个`call()`方法里，新`OnSubscribe`利用`operator.call(subscriber)`生成了一个新的`Subscriber`(`Operator`就是在这里，通过自己的`call()`方法将新`Subscriber`和原始`Subscriber` 进行关联，并插入自己的『变换』代码以实现变换），然后利用这个新`Subscriber`向原始`Observable`进行订阅。 
    这样就实现了`lift()`过程，有点像一种代理机制，通过事件拦截和处理实现事件序列的变换。

精简掉细节的话，也可以这么说：在`Observable`执行了`lift(Operator)`方法之后，会返回一个新的`Observable`，这个新的`Observable`会像一个代理一样，负责接收原始的`Observable` 发出的事件，并在处理后发送给`Subscriber`。

如果你更喜欢具象思维，可以看图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_lift.gif?raw=true)

举一个具体的`Operator`的实现。下面这是一个将事件中的`Integer`对象转换成`String`的例子，仅供参考：

```java
observable.lift(new Observable.Operator<String, Integer>() {
    @Override
    public Subscriber<? super Integer> call(final Subscriber<? super String> subscriber) {
        // 将事件序列中的 Integer 对象转换为 String 对象
        return new Subscriber<Integer>() {
            @Override
            public void onNext(Integer integer) {
                subscriber.onNext("" + integer);
            }

            @Override
            public void onCompleted() {
                subscriber.onCompleted();
            }

            @Override
            public void onError(Throwable e) {
                subscriber.onError(e);
            }
        };
    }
});
```
> 讲述`lift()`的原理只是为了让你更好地了解`RxJava` ，从而可以更好地使用它。然而不管你是否理解了`lift()`的原理,`RxJava`都不建议开发者自定义`Operator`来直接使用`lift()`，而是建议尽量使用已有的`lift()`包装方法（如`map()、flatMap()`等）进行组合来实现需求，因为直接使用`lift()`非常容易发生一些难以发现的错误。


线程控制`Scheduler`
---

在不指定线程的情况下，`RxJava`遵循的是线程不变的原则，即在哪个线程调用`subscribe()`方法就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。也就是说事件的发出和消费都是在同一个线程的。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于`RxJava`是至关重要的。而要实现异步，则需要用到`RxJava`的另一个概念：`Scheduler`。   


####`Scheduler`简介


在`RxJava`中,`Scheduler`相当于线程控制器，`RxJava`通过它来指定每一段代码应该运行在什么样的线程。`RxJava`已经内置了几个`Scheduler`，它们已经适合大多数的使用场景：

- `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的`Scheduler`。
- `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
- `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的`Scheduler`。行为模式和`newThread()`差不多，区别在于`io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下`io()`比`newThread()`更有效率。不要把计算工作放在`io()`中，可以避免创建不必要的线程。
- `Schedulers.computation()`: 计算所使用的`Scheduler`。这个计算指的是`CPU`密集型计算，即不会被`I/O`等操作限制性能的操作，例如图形的计算。这个`Scheduler` 使用的固定的线程池，大小为`CPU`核数。不要把`I/O`操作放在`computation()`中，否则`I/O`操作的等待时间会浪费`CPU`。
- 另外，`Android`还有一个专用的`AndroidSchedulers.mainThread()`，它指定的操作将在`Android`主线程运行。


有了这几个`Scheduler`，就可以使用`subscribeOn()`和`observeOn()`两个方法来对线程进行控制了。`subscribeOn()`指定`subscribe()`所发生的线程，即`Observable.OnSubscribe()`被激活时所处的线程或者叫做事件产生的线程。`observeOn()`指定`Subscriber`所运行在的线程或者叫做事件消费的线程。

```java
Observable.just("Hello ", "World !")
        .subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程，可以理解成数据的获取是在io线程
        .observeOn(AndroidSchedulers.mainThread())// 指定 Subscriber 的回调发生在主线程，可以理解成数据的消费时在主线程
        .subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                Log.i("@@@", s);
            }
        });
```

上面这段代码中，`subscribeOn(Schedulers.io())`的指定会让创建的事件的内容`Hello `、`World !`将会在`IO`线程发出；而由于`observeOn(AndroidScheculers.mainThread())` 的指定，因此`subscriber()`方法设置后的回调中内容的打印将发生在主线程中。事实上，这种在`subscribe()`之前写上两句`subscribeOn(Scheduler.io())`和`observeOn(AndroidSchedulers.mainThread())`的使用方式非常常见，它适用于多数的***后台线程取数据，主线程显示***的程序策略。

####`Scheduler`的原理

`RxJava`的`Scheduler API`很方便，也很神奇（加了一句话就把线程切换了，怎么做到的？而且 subscribe() 不是最外层直接调用的方法吗，它竟然也能被指定线程？）。然而 Scheduler 的原理需要放在后面讲，因为它的原理是以下一节《变换》的原理作为基础的。

好吧这一节其实我屁也没说，只是为了让你安心，让你知道我不是忘了讲原理，而是把它放在了更合适的地方。


能不能多切换几次线程？答案是：能。因为`observeOn()`指定的是`Subscriber`的线程，而这个`Subscriber`并不是（严格说应该为『不一定是』，但这里不妨理解为『不是』）`subscribe()` 参数中的`Subscriber`，而是`observeOn()`执行时的当前`Observable`所对应的`Subscriber`，即它的直接下级`Subscriber`。换句话说`observeOn()` 指定的是它之后的操作所在的线程。因此如果有多次切换线程的需求，只要在每个想要切换线程的位置调用一次`observeOn()`即可。

上代码：

```java
Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.newThread())
    .map(mapOperator) // 新线程，由 observeOn() 指定
    .observeOn(Schedulers.io())
    .map(mapOperator2) // IO 线程，由 observeOn() 指定
    .observeOn(AndroidSchedulers.mainThread) 
    .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定
```
如上，通过`observeOn()`的多次调用，程序实现了线程的多次切换。
不过，不同于`observeOn()`,`subscribeOn()`的位置放在哪里都可以，但它是只能调用一次的。

其实，`subscribeOn()`和`observeOn()`的内部实现，也是用的`lift()`。

具体看图（不同颜色的箭头表示不同的线程,`subscribeOn()`原理图：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_subscribeon.jpg?raw=true)

`observeOn()`原理图: 

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_observeon.jpg?raw=true)

从图中可以看出,`subscribeOn()`和`observeOn()`都做了线程切换的工作（图中的`schedule...`部位）。不同的是,`subscribeOn()`的线程切换发生在`OnSubscribe`中，即在它通知上一级 `OnSubscribe`时，这时事件还没有开始发送，因此`subscribeOn()`的线程控制可以从事件发出的开端就造成影响；而`observeOn()`的线程切换则发生在它内建的`Subscriber`中，即发生在它即将给下一级`Subscriber`发送事件时，因此`observeOn()`控制的是它后面的线程。

最后，我用一张图来解释当多个`subscribeOn()`和`observeOn()`混合使用时，线程调度是怎么发生的（由于图中对象较多，相对于上面的图对结构做了一些简化调整）：

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/rxjava_observable_list.jpg?raw=true)

图中共有5处含有对事件的操作。由图中可以看出，①和②两处受第一个`subscribeOn()`影响，运行在红色线程；③和④处受第一个`observeOn()`的影响，运行在绿色线程；⑤处受第二个 `onserveOn()`影响，运行在紫色线程；而第二个`subscribeOn()`，由于在通知过程中线程就被第一个`subscribeOn()` 截断，因此对整个流程并没有任何影响。这里也就回答了前面的问题：当使用了多个`subscribeOn()`的时候，只有第一个`subscribeOn()`起作用。

在前面讲`Subscriber`的时候，提到过`Subscriber`的`onStart()`可以用作流程开始前的初始化。然而`onStart()`由于在`subscribe()`发生时就被调用了，因此不能指定线程，而是只能执行在`subscribe()`被调用时的线程。这就导致如果`onStart()`中含有对线程有要求的代码（例如在界面上显示一个`ProgressBar`，这必须在主线程执行），将会有线程非法的风险，因为有时你无法预测`subscribe()`将会在什么线程执行。

而与`Subscriber.onStart()`相对应的，有一个方法`Observable.doOnSubscribe()`。它和`Subscriber.onStart()`同样是在`subscribe()`调用后而且在事件发送前执行，但区别在于它可以指定线程。默认情况下,`doOnSubscribe()`执行在`subscribe()`发生的线程；而如果在`doOnSubscribe()`之后有`subscribeOn()`的话，它将执行在离它最近的`subscribeOn()`所指定的线程。

示例代码：

```java
Observable.create(onSubscribe)
    .subscribeOn(Schedulers.io())
    .doOnSubscribe(new Action0() {
        @Override
        public void call() {
            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行
        }
    })
    .subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(subscriber);
```

如上，在`doOnSubscribe()`的后面跟一个`subscribeOn()`，就能指定准备工作的线程了。

####总结


`RxJava`辣么好，难道他就没有缺点吗？当然有那就是使用越来越多的订阅，内存开销也会变得很大，稍不留神就会出现内存溢出的情况。我们可以用`RxJava`实现基本任何功能，但是你并不能这么做，你要明白什么时候需要用它，而什么时候没必要用它，不要一味的把功能都有`RxJava`来实现。    

至于上面提到的`2.0`版本，有关它的区别请见:    
[What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)


Agera
---

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/agera.png?raw=true)


之前`Google`发布`agera`，它在`Github`上的介绍是:`Reactive Programming for Android`

[agera](https://github.com/google/agera)

既然是响应式编程框架，其核心思想肯定和`RxJava`也是类似的。
`Agera`中核心也是关于事件流（数据流）的处理，可以转换数据、可以指定数据操作函数在那个线程执行。
而`Agera`和`RxJava`不一样的地方在于，`Agera`提供了一个新的事件响应和数据请求的模型，被称之为`Push event, pull data`。也就是一个事件发生了，会通过回调来主动告诉你，你关心的事件发生了。然后你需要主动的去获取数据，根据获取到的数据做一些操作。而`RxJava`在事件发生的时候，已经带有数据了。为了支持 `Push event pull data`模型，`Agera`中有个新的概念 — `Repository`。`Repository`翻译过来也就是数据仓库,是用来提供数据的,当你收到事件的时候,通过`Repository`来获取需要的数据。 这样做的好处是，把事件分发和数据处理的逻辑给分开了，事件分发做的事情比较少，事件分发就比较高效，当你收到事件后，根据当前的状态如果发现这个时候，数据已经无效了，则你根本不用请求数据，这样数据也就不用去处理和转换了。这样可以避免无用的数据计算，而有用的数据计算也可以在你需要的时候才去计算。
同样，在多线程环境下，使用`push event pull data`模型，可能当你收到事件通知的时候，你只能获取到最新的数据，无法去获取历史数据了。设计就是这样考虑的，因为在`Android`开发中大部分的数据都是用来更新`UI`的，这个时候你根本不需要旧的数据了，只要最新的就够了。

`Agera`相对比较简单，并且是一个专业为`Android`平台打造的响应式编程框架。但是如果你熟悉了`RxJava`你会发现其用起来会有点不舒服，特别是还要主动的获取数据略感不爽（虽然提高了事件分发的效率）。 

总之，现在看来`RxJava`支持的比较全面，但是会略显笨重，而`Agera`是一个专门为`Android`平台设计的响应式编程框架，比较轻巧，当然支持的并不是很全面。可以根据自己的喜好来选择.

但是从个人观点来看`Agera`比起来`RxJava`还是相差几个版本。当然这是我的片面之词，有关更多信息请看[Question: what's the relation to RxJava?](https://github.com/google/agera/issues/20)


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