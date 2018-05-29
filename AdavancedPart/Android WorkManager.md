Android WorkManager
===

谷歌在今年的`Google I/O`上宣布了一项非常令人兴奋的功能，该功能允许开发人员执行传统上需要详细了解各种API级别和可用于这些API的后台任务库的后台任务(简单点说就是”管理一些要在后台工作的任务, – 即使你的应用没启动也能保证任务能被执行”)，这就是[WorkManager](https://developer.android.com/reference/androidx/work/WorkManager),`WorkManager`提供了从其他`API`（例如`JobScheduler`，`Firebase.JobDispatcher`，`AlarmManager`和`Services`）中获得的功能，而无需研究哪种`API`可用于您的设备或`API`。

这句话是什么意思？既然能用`JobScheduler`和`AlarmManager`等，何必再出来个`WorkManager`呢？其实`WorkManager`在底层也是看你是什么版本来选到底是`JobScheduler`,`AlamarManager`来做。 
`JobScheduler`是在`SDK`21中才有的，而且在`SDK`21中还有`Bug`，稳定可用是从`SDK`23开始的. 而`AlarmManager`一直存在，但是`AlarmManager`也不是最好的选择，
注意：如果您的应用程序的目标是`API`级别26或更高，则当应用程序本身不在前台时，系统会对运行后台服务施加限制。在大多数情况下，您的应用程序应该使用预定作业。所以`WorkManager`在底层, 会根据你的设备情况, 选用`JobScheduler`,`Firebase`的`JobDispatcher`,或是`AlarmManager`。


`WorkManager`,它在应用被杀, 甚至设备重启后仍能保证你安排给他的任务能得到执行。那这样我们以后是不是可以把后台任务都用它来实现了?其实`Google`自己也说了:`WorkManager`并不是为了那种在应用内的后台线程而设计出来的. 这种需求你应该使用`ThreadPool`。

具体的规则如下:   

`WorkManager`根据以下标准在可用时使用对应的底层工作服务:    

- 在`API 23+`使用`JobScheduler`
- 在`API 14-22`中    
    - 如果在应用中使用了`Firebase JobDispatcher`并且有可选的`Firebase`依赖项就使用`Firebase JobDispatcher`
    - 否则使用自定义的`AlarmManager + BroadcastReceiver`的实现方式


![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/workmanager.png)




组成部分:   

- `WorkManager`:通过对应的参数来接受`work`并排队执行`work`。
- `Worker`:通过实现`doWork()`方法来指定在后台执行的具体功能。
- `WorkRequest`:代表一个独立的任务。它会告诉你哪个`worker`加入了，以及它需要满足哪些约束才能运行。`WorkRequest`是一个抽象类，您将使用[OneTimeWorkRequest](https://developer.android.com/reference/androidx/work/OneTimeWorkRequest)或[PeriodicWorkRequest](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest)。
- `WorkStatus`:为每个WorkRequest对象提供数据。



好了下面开始演示:  

首先加入依赖:   

```
dependencies {
    def work_version = "1.0.0-alpha02"
    implementation "android.arch.work:work-runtime:$work_version"
}
```

- 继承`Worker`类并实现`doWork()`方法

```kotlin
class MineWorker : Worker() {
    override fun doWork(): WorkerResult {
        Log.e("@@@", "dang dang dang !!!")
        return WorkerResult.SUCCESS
    }
}
```

- 接下来如果想要该`Worker`执行，需要调用`WorkManager`将该`Worker`添加到队列中

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val workRequest = OneTimeWorkRequest.Builder(MineWorker::class.java).build()
        val instance = WorkManager.getInstance()
        instance.enqueue(workRequest)

    }
}
```

好了，执行下:  
```
05-29 12:15:51.066 6360-6406/com.charon.workmanagerdemo E/@@@: dang dang dang !!!
```

并没什么卵用，因为这个示例代码就上来就执行一次。 很显然这个不合理啊，我想给该`worker`添加一些限制条件，例如我想在手机充电时，并且有网的情况下执行某个任务，
而且我想让它执行多次，这里可以通过`Constraints`来限制:   

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresCharging(true)
            .build()

    val workRequest = OneTimeWorkRequest.Builder(MineWorker::class.java)
            .setConstraints(constraints).build()

    val instance = WorkManager.getInstance()
    instance.enqueue(workRequest)
}
```
我们把手机网络关掉运行下，这次运行后并没有`log`打印。那现在把网络打开:   

```
05-29 12:21:02.061 7092-7241/? E/@@@: dang dang dang !!!
```


当然我们可以获取每个请求的状态，以及取消该请求:   
```kotlin
val statusById: LiveData<WorkStatus> = instance.getStatusById(workRequest.id)
instance.cancelWorkById(workRequest.id)
```
如果想要顺序的去执行不同的`OneTimeWorker`也是很方便的:  
```kotlin
WorkContinuation chain1 = WorkManager.getInstance()
    .beginWith(workA)
    .then(workB);
WorkContinuation chain2 = WorkManager.getInstance()
    .beginWith(workC)
    .then(workD);
WorkContinuation chain3 = WorkContinuation
    .combine(chain1, chain2)
    .then(workE);
chain3.enqueue();
```

上面都是用了`OneTimeWorkRequest`，如果你想定期的去执行某一个`worker`的话，可以使用`PeriodicWorkRequest`:   
```kotlin
val workRequest = PeriodicWorkRequest.Builder(MineWorker::class.java, 3, TimeUnit.SECONDS)
        .setConstraints(constraints)
        .setInputData(data)
        .build()
```

但是我使用这个定期任务执行的时候也只是执行了一次，并没有像上面的代码中那样3秒执行一次，什么鬼？ 
我们看看`PeriodicWorkRequest`的源码:    

```kotlin
/**
 * A class that represents a request for repeating work.
 */

public final class PeriodicWorkRequest extends WorkRequest {

    /**
     * The minimum interval duration for {@link PeriodicWorkRequest}, in milliseconds.
     * Based on {@see https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/job/JobInfo.java#110}.
     */
    public static final long MIN_PERIODIC_INTERVAL_MILLIS = 15 * 60 * 1000L; // 15 minutes.
```

上面说的很明白，最小间隔15分钟，- -！


总体来说,`WorkManager`并不是要取代线程池`AsyncTask/RxJava`.反而是像`AlarmManager`来做定时任务的意思.即保证你给它的任务能完成, 即使你的应用都没有被打开, 或是设备重启后也能让你的任务被执行.`WorkManager`在设计上设计得比较好.没有把`worker`,任务混为一谈,而是把它们解耦成`Worker`,`WorkRequest`.这样分层就清晰多了, 也好扩展.


到了这里突然有了一个大胆的想法。看到没有它能保证任务的执行。
我们之前写过一篇文章[Android卸载反馈](https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%E5%8D%B8%E8%BD%BD%E5%8F%8D%E9%A6%88.md)
里面用到了`c`中的`fork`来保证存活，达到常驻内存的功能，如果`PeriodicWorkRequest`的最小间隔时间比较短不是15分钟的话，那这里是不是也可以用`WorkManager`来实现？ 好了，不说了。



参考:   

- [官方文档](https://developer.android.com/topic/libraries/architecture/workmanager)
- [Codelabs android-workmanager](https://codelabs.developers.google.com/codelabs/android-workmanager/#0)
- [示例代码](https://github.com/googlecodelabs/android-workmanager)
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 