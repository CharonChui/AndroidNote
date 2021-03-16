LeakCanary源码分析
===

[LeakCanary](https://github.com/square/leakcanary)是一个用于检测内存泄漏的工具，可以用于Java和Android，是由著名开源组织Square贡献。

强烈建议使用LeakCanary 2.x版本，更高效、使用更简单，而且没有任何Java代码，它当泄露引用到达5时才会发起heap dump，同时使用了全新的heap parser，减少内存占用，提升速度。只需要在dependencies中加入leakcanary的依赖即可。而且debugimplementation只在debug模式下有效，所以不用担心用户在正式环境下也会出现LeanCanary收集。而且是完全使用kotlin实现了，同时使用了[see Shark](https://square.github.io/leakcanary/shark/)来进行heap内存分析，更节省内存。



### 使用

只需在app的build.gradle中按如下配置即可，没有任何其他代码：

```xml
dependencies {
  // debugImplementation because LeakCanary should only run in debug builds.
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.6'
}
```



到这里你肯定会有个疑问：一行Java代码都没有，它是如何做到监控的？ 后面我们就带着这个疑问来进行源码查看。不过再看源码之前，我们需要先了解一些引用的知识[**强引用、软引用、弱引用、虚引用**](https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%BC%BA%E5%BC%95%E7%94%A8%E3%80%81%E8%BD%AF%E5%BC%95%E7%94%A8%E3%80%81%E5%BC%B1%E5%BC%95%E7%94%A8%E3%80%81%E8%99%9A%E5%BC%95%E7%94%A8.md)

这篇文章中在介绍弱引用时是这样说的: 

同软引用，也用来描述非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。在对象没有其他引用的情况下，调用system.gc对象可被虚拟机回收，就是一个普通的员工，平常如果表现不佳会被开除（对象没有其他引用的情况下），遇到别人投诉（调用GC)上班看片,那开除是肯定了(被虚拟机回收)。

在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了***只具***有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

而LeakCanary的原因也正是使用了弱引用的这个特点。

### 监测原理

ReferenceQueue+WeakReference+手动调用GC可实现这个需求。

WeakReference和ReferenceQueue联合使用，
当被WeakReference引用的对象的生命周期结束，一旦被GC检查到，GC将会把该对象添加到ReferenceQueue中，待ReferenceQueue处理。
当GC过后对象一直不被加入ReferenceQueue，它可能存在内存泄漏。  

LeakCanary的基础是一个叫做ObjectWatcher Android的library。它hook了Android的生命周期，当activity和fragment 被销毁并且应该被垃圾回收时候自动检测。这些被销毁的对象被传递给ObjectWatcher， ObjectWatcher持有这些被销毁对象的弱引用（weak references）。如果弱引用在等待5秒钟并运行垃圾收集器后仍未被清除，那么被观察的对象就被认为是保留的（retained，在生命周期结束后仍然保留），并存在潜在的泄漏。LeakCanary会在Logcat中输出这些日志。

### 监测内容

LeakCanary会自动监测如下对象的泄露情况： 

- destroyed `Activity` instances
- destroyed `Fragment` instances
- destroyed fragment `View` instances
- cleared `ViewModel` instances



### 监测步骤

一旦LeakCanary被安装，它会自动监测和上报泄露，分为以下四步： 

1. Detecting retained objects.
2. Dumping the heap.
3. Analyzing the heap.
4. Categorizing leaks.



好了，我们接下来从源码的角度来看一下具体的实现，但是看到代码我懵逼了，从哪里看呢？ 我完全没头绪。 

![leakcanary_source](https://raw.githubusercontent.com/CharonChui/Pictures/master/leakcanary_source.png)

那就继续看官方介绍吧。

## 1. Detecting retained objects[¶](https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/#1-detecting-retained-objects)

LeakCanary hooks into the Android lifecycle to automatically detect when activities and fragments are destroyed and should be garbage collected. These destroyed objects are passed to an `ObjectWatcher`, which holds [weak references](https://en.wikipedia.org/wiki/Weak_reference) to them. LeakCanary automatically detects leaks for the following objects:

- destroyed `Activity` instances
- destroyed `Fragment` instances
- destroyed fragment `View` instances
- cleared `ViewModel` instances

You can watch any objects that is no longer needed, for example a detached view or a destroyed presenter:

```
AppWatcher.objectWatcher.watch(myDetachedView, "View was detached")
```

If the weak reference held by `ObjectWatcher` isn’t cleared after **waiting 5 seconds** and running garbage collection, the watched object is considered **retained**, and potentially leaking. LeakCanary logs this to Logcat:

```
D LeakCanary: Watching instance of com.example.leakcanary.MainActivity
  (Activity received Activity#onDestroy() callback) 

... 5 seconds later ...

D LeakCanary: Scheduling check for retained objects because found new object
  retained
```

LeakCanary waits for the count of retained objects to reach a threshold before dumping the heap, and displays a notification with the latest count.





## 2. Dumping the heap[¶](https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/#2-dumping-the-heap)

When the count of retained objects reaches a threshold, LeakCanary dumps the Java heap into a `.hprof` file (a **heap dump**) stored onto the Android file system (see [Where does LeakCanary store heap dumps?](https://square.github.io/leakcanary/faq/#where-does-leakcanary-store-heap-dumps)). Dumping the heap freezes the app for a short amount of time, during which LeakCanary displays the following toast:



## 3. Analyzing the heap[¶](https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/#3-analyzing-the-heap)

LeakCanary parses the `.hprof` file using [Shark](https://square.github.io/leakcanary/shark/) and locates the retained objects in that heap dump.



## How does LeakCanary get installed by only adding a dependency?[¶](https://square.github.io/leakcanary/faq/#how-does-leakcanary-get-installed-by-only-adding-a-dependency)

On Android, content providers are created after the Application instance is created but before Application.onCreate() is called. The `leakcanary-object-watcher-android` artifact has a non exported ContentProvider defined in its `AndroidManifest.xml` file. When that ContentProvider is installed, it adds activity and fragment lifecycle listeners to the application.

对于Application的onCreate()方法，官方文档中是这样说的: 

Called when the application is starting, before any activity, service, or receiver objects(excluding content providers)have been created.



看到这里，大呼内行...这个实现太巧妙了。尽然是通过在AndroidManifest.xml中注册一个内容提供者，这个内容提供者的创建会在Application.onCreate()之前，在它创建的时候去做install的操作。



这也太牛逼了，之前从来没想到ContentProvider竟然还有这个功能，以后估计很多第三方的库都会这样来实现，但是这样也会有一个问题，就是会影响到应用的冷启时间，好在LeakCanary只是在Debug的时候使用，也就不会存在这个问题了。 

好了，可以看代码了，先看一下这些module的依赖关系,Gradle中的module下Task中的help中的dependencies: 

```
debugRuntimeClasspath - Runtime classpath of compilation 'debug' (target  (androidJvm)).
+--- project :leakcanary-android
|    +--- project :leakcanary-android-core
|    |    +--- project :shark-android
|    |    |    +--- project :shark
|    |    |    |    +--- project :shark-graph
|    |    |    |    |    +--- project :shark-hprof
|    |    |    |    |    |    +--- project :shark-log
|    |    |    |    |    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21
|    |    |    |    |    |    |         +--- org.jetbrains.kotlin:kotlin-stdlib-common:1.4.21
|    |    |    |    |    |    |         \--- org.jetbrains:annotations:13.0
|    |    |    |    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    |    |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    |    |    |    \--- com.squareup.okio:okio:2.2.2
|    |    |    |    |         \--- org.jetbrains.kotlin:kotlin-stdlib:1.2.60 -> 1.4.21 (*)
|    |    |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    |    |    \--- com.squareup.okio:okio:2.2.2 (*)
|    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    +--- project :leakcanary-object-watcher-android
|    |    |    +--- project :leakcanary-object-watcher
|    |    |    |    +--- project :shark-log (*)
|    |    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    |    +--- project :leakcanary-android-utils
|    |    |    |    +--- project :shark-log (*)
|    |    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    |    +--- com.squareup.curtains:curtains:1.0.1
|    |    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.4.21 (*)
|    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    +--- project :leakcanary-object-watcher-android-androidx
|    |    |    +--- project :leakcanary-object-watcher-android (*)
|    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    +--- project :leakcanary-object-watcher-android-support-fragments
|    |    |    +--- project :leakcanary-object-watcher-android (*)
|    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    +--- project :plumber-android
|    |    |    +--- project :shark-log (*)
|    |    |    +--- project :leakcanary-android-utils (*)
|    |    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    |    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
|    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)
\--- org.jetbrains.kotlin:kotlin-stdlib:1.3.72 -> 1.4.21 (*)

```



那我们就去`leakcanary-object-watcher-android`库的AndroidManifest.xml中看一下这个内容提供者:  

```xml
  <application>
    <provider
        android:name="leakcanary.internal.AppWatcherInstaller$MainProcess"
        android:authorities="${applicationId}.leakcanary-installer"
        android:enabled="@bool/leak_canary_watcher_auto_install"
        android:exported="false"/>
  </application>
```

继续看一下AppWatcherInstaller类的实现:  

```kotlin
/**
 * Content providers are loaded before the application class is created. [AppWatcherInstaller] is
 * used to install [leakcanary.AppWatcher] on application start.
 */
internal sealed class AppWatcherInstaller : ContentProvider() {

  /**
   * [MainProcess] automatically sets up the LeakCanary code that runs in the main app process.
   */
  internal class MainProcess : AppWatcherInstaller()

  /**
   * When using the `leakcanary-android-process` artifact instead of `leakcanary-android`,
   * [LeakCanaryProcess] automatically sets up the LeakCanary code
   */
  internal class LeakCanaryProcess : AppWatcherInstaller()

  override fun onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    AppWatcher.manualInstall(application)
    return true
  }

  override fun query(
    uri: Uri,
    strings: Array<String>?,
    s: String?,
    strings1: Array<String>?,
    s1: String?
  ): Cursor? {
    return null
  }

  override fun getType(uri: Uri): String? {
    return null
  }

  override fun insert(
    uri: Uri,
    contentValues: ContentValues?
  ): Uri? {
    return null
  }

  override fun delete(
    uri: Uri,
    s: String?,
    strings: Array<String>?
  ): Int {
    return 0
  }

  override fun update(
    uri: Uri,
    contentValues: ContentValues?,
    s: String?,
    strings: Array<String>?
  ): Int {
    return 0
  }
}
```

上面的注释说的很明白，这个内容提供者的主要目的就是调用AppWatcher.install，我们继续看一下AppWatcher.manualInstall()方法:  

```kot
  @JvmOverloads
  fun manualInstall(
    application: Application,
    retainedDelayMillis: Long = TimeUnit.SECONDS.toMillis(5),
    watchersToInstall: List<InstallableWatcher> = appDefaultWatchers(application)
  ) {
  	// 检查是否是主线程
    checkMainThread()
    // 检查是否已经installed
    check(!isInstalled) {
      "AppWatcher already installed"
    }
    check(retainedDelayMillis >= 0) {
      "retainedDelayMillis $retainedDelayMillis must be at least 0 ms"
    }
    this.retainedDelayMillis = retainedDelayMillis
    if (application.isDebuggableBuild) {
      // debug模式的话就初始化泄露信息的logcat打印
      LogcatSharkLog.install()
    }
    // Requires AppWatcher.objectWatcher to be set
    LeakCanaryDelegate.loadLeakCanary(application)

    watchersToInstall.forEach {
      // 不传递的话，默认为appDefaultWatchers，将这里面的四个类型的检测器各自初始化
      it.install()
    }
  }
  
  // appDefaultWatchers()的实现为: 
    fun appDefaultWatchers(
    application: Application,
    reachabilityWatcher: ReachabilityWatcher = objectWatcher
  ): List<InstallableWatcher> {
    return listOf(
      ActivityWatcher(application, reachabilityWatcher),
      FragmentAndViewModelWatcher(application, reachabilityWatcher),
      RootViewWatcher(reachabilityWatcher),
      ServiceWatcher(reachabilityWatcher)
    )
  }
  
```

这里我们以ActivityWatcher为例，看一下它的install实现: 

```kotlin
/**
 * Expects activities to become weakly reachable soon after they receive the [Activity.onDestroy]
 * callback.
 */
class ActivityWatcher(
  private val application: Application,
  private val reachabilityWatcher: ReachabilityWatcher
) : InstallableWatcher {

  private val lifecycleCallbacks =
    object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityDestroyed(activity: Activity) {
        reachabilityWatcher.expectWeaklyReachable(
          activity, "${activity::class.java.name} received Activity#onDestroy() callback"
        )
      }
    }

  override fun install() {
    application.registerActivityLifecycleCallbacks(lifecycleCallbacks)
  }

  override fun uninstall() {
    application.unregisterActivityLifecycleCallbacks(lifecycleCallbacks)
  }
}
```

这里面的实现也很简单，就是通过Application.registerActivityLifecycleCallbacks然后在activity destroy的回调中去执行ReachabilityWatcher接口的expectWeaklyReachable，ReachabilityWatcher接口的实现类是ObjectWatcher类: 

```kotlin
class ObjectWatcher constructor(
  private val clock: Clock,
  private val checkRetainedExecutor: Executor,
  /**
   * Calls to [watch] will be ignored when [isEnabled] returns false
   */
  private val isEnabled: () -> Boolean = { true }
) : ReachabilityWatcher {  
  // ...
  // Map集合，用来存放要观察对象的key和弱引用,代码会为每个观察的对象生成一个唯一的key和弱应用  
  private val watchedObjects = mutableMapOf<String, KeyedWeakReference>()
  // 这个队列是与弱引用联合使用，当弱引用中的对象被回收后，这个弱引用会被放到这个队列中。
  // 也就是说只要存在与这个队列中的弱引用就代表该弱引用所包含的对象被回收了。  
  private val queue = ReferenceQueue<Any>()

  @Synchronized override fun expectWeaklyReachable(
    watchedObject: Any,
    description: String
  ) {
    if (!isEnabled()) {
      return
    }
    // 清理操作：先移除watchedObjects和queue中已经被回收的对象的弱引用
    removeWeaklyReachableObjects()
    // 生成唯一的key  
    val key = UUID.randomUUID()
      .toString()
    val watchUptimeMillis = clock.uptimeMillis()
    // 用要监听的对象创建KeyedWeakReference  
    val reference =
      KeyedWeakReference(watchedObject, key, description, watchUptimeMillis, queue)
    SharkLog.d {
      "Watching " +
        (if (watchedObject is Class<*>) watchedObject.toString() else "instance of ${watchedObject.javaClass.name}") +
        (if (description.isNotEmpty()) " ($description)" else "") +
        " with key $key"
    }

    watchedObjects[key] = reference
    // 这是一个延迟任务，延迟五秒后再次执行moveToRetained()方法
    checkRetainedExecutor.execute {
      moveToRetained(key)
    }
  }
}    
```

我们继续看moveToRetained()方法的实现:  

```kotlin
  @Synchronized private fun moveToRetained(key: String) {
    // 该方法会调用removeWeaklyReachableObjects再去执行一次清理操作  
    // 因为这个期间对象可能已经被回收了，所以需要再清理一次  
    removeWeaklyReachableObjects()
    val retainedRef = watchedObjects[key]
    // 等再执行完清理操作后，那还在watchedObjects集合中的对象就是没有被回收的对象了  
    if (retainedRef != null) {
      retainedRef.retainedUptimeMillis = clock.uptimeMillis()
      onObjectRetainedListeners.forEach { it.onObjectRetained() }
    }
  }
```

继续看OnObjectRetainedListener接口的onObjectRetained()方法，这里它的实现类是InternalLeakCanary:  

```kotlin
  private lateinit var heapDumpTrigger: HeapDumpTrigger

  override fun onObjectRetained() = scheduleRetainedObjectCheck()

  fun scheduleRetainedObjectCheck() {
    if (this::heapDumpTrigger.isInitialized) {
      heapDumpTrigger.scheduleRetainedObjectCheck()
    }
  }
```

继续看HeapDumpTrigger类的scheduleRetainedObjectCheck():  

```kotlin
  fun scheduleRetainedObjectCheck(
    delayMillis: Long = 0L
  ) {
    val checkCurrentlyScheduledAt = checkScheduledAt
    if (checkCurrentlyScheduledAt > 0) {
      return
    }
    checkScheduledAt = SystemClock.uptimeMillis() + delayMillis
    backgroundHandler.postDelayed({
      checkScheduledAt = 0
      checkRetainedObjects()
    }, delayMillis)
  }
```

这里会立即执行checkRetainedObjects()方法:    

```kotlin
private fun checkRetainedObjects() {
 	// 调用ObjectWatcher中watchedObjects集合中对象retainedUptimeMillis != -1的数量
    // 这些是可能发生泄漏的对象数量  
    var retainedReferenceCount = objectWatcher.retainedObjectCount
    if (retainedReferenceCount > 0) {
      // 如果数量> 0，调用gc
      gcTrigger.runGc()
      // 调用完gc后重新获取数量
      retainedReferenceCount = objectWatcher.retainedObjectCount
    }
	// 如果个数小于5，不做操作等待5秒再次进行检查未回收的个数，一直循环，直到大于等于5个或者等于0个，为了防止频发回收堆造成卡顿。
    // 大于5个后，如果处于debug模式，会再等20秒，再次执行循环gc的操作。防止debug模式会减慢回收
    if (checkRetainedCount(retainedReferenceCount, config.retainedVisibleThreshold)) return

    val now = SystemClock.uptimeMillis()
    val elapsedSinceLastDumpMillis = now - lastHeapDumpUptimeMillis
    // 距离上次堆栈分析是否大于等于1分钟，如果没有超过一分钟，也需要再次延迟（1分钟-当前距离上次的时间）再次循环gc的操作
    if (elapsedSinceLastDumpMillis < WAIT_BETWEEN_HEAP_DUMPS_MILLIS) {
      onRetainInstanceListener.onEvent(DumpHappenedRecently)
      showRetainedCountNotification(
        objectCount = retainedReferenceCount,
        contentText = application.getString(R.string.leak_canary_notification_retained_dump_wait)
      )
      scheduleRetainedObjectCheck(
        delayMillis = WAIT_BETWEEN_HEAP_DUMPS_MILLIS - elapsedSinceLastDumpMillis
      )
      return
    }

    dismissRetainedCountNotification()
    val visibility = if (applicationVisible) "visible" else "not visible"
    // 最终调用dump
    dumpHeap(
      retainedReferenceCount = retainedReferenceCount,
      retry = true,
      reason = "$retainedReferenceCount retained objects, app is $visibility"
    )
  }
```

继续看dumpHeap()方法:  

```kotlin
  private fun dumpHeap(
    retainedReferenceCount: Int,
    retry: Boolean,
    reason: String
  ) {
    saveResourceIdNamesToMemory()
    val heapDumpUptimeMillis = SystemClock.uptimeMillis()
    KeyedWeakReference.heapDumpUptimeMillis = heapDumpUptimeMillis
    // HeapDumper.dumpHeap()方法来获取Heap信息，生成hprof文件  
    when (val heapDumpResult = heapDumper.dumpHeap()) {
      is NoHeapDump -> {
        if (retry) {
          SharkLog.d { "Failed to dump heap, will retry in $WAIT_AFTER_DUMP_FAILED_MILLIS ms" }
          scheduleRetainedObjectCheck(
            delayMillis = WAIT_AFTER_DUMP_FAILED_MILLIS
          )
        } else {
          SharkLog.d { "Failed to dump heap, will not automatically retry" }
        }
        showRetainedCountNotification(
          objectCount = retainedReferenceCount,
          contentText = application.getString(
            R.string.leak_canary_notification_retained_dump_failed
          )
        )
      }
      is HeapDump -> {
        lastDisplayedRetainedObjectCount = 0
        lastHeapDumpUptimeMillis = SystemClock.uptimeMillis()
        // 清楚早于heapDumpUptimeMillis之前的元素  
        objectWatcher.clearObjectsWatchedBefore(heapDumpUptimeMillis)
        // 将heap信息文件交给HeapAnalyzerService进行分析  
        HeapAnalyzerService.runAnalysis(
          context = application,
          heapDumpFile = heapDumpResult.file,
          heapDumpDurationMillis = heapDumpResult.durationMillis,
          heapDumpReason = reason
        )
      }
    }
  }
```



所以我们要分两部分量看: 

##### 1. HeapDumper接口的实现类是AndroidHeapDumper.dumpHeap()

```kotlin
  override fun dumpHeap(): DumpHeapResult {
    val heapDumpFile = leakDirectoryProvider.newHeapDumpFile() ?: return NoHeapDump

    val waitingForToast = FutureResult<Toast?>()
    showToast(waitingForToast)

    if (!waitingForToast.wait(5, SECONDS)) {
      SharkLog.d { "Did not dump heap, too much time waiting for Toast." }
      return NoHeapDump
    }

    val notificationManager =
      context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    if (Notifications.canShowNotification) {
      val dumpingHeap = context.getString(R.string.leak_canary_notification_dumping)
      val builder = Notification.Builder(context)
        .setContentTitle(dumpingHeap)
      val notification = Notifications.buildNotification(context, builder, LEAKCANARY_LOW)
      notificationManager.notify(R.id.leak_canary_notification_dumping_heap, notification)
    }

    val toast = waitingForToast.get()

    return try {
      val durationMillis = measureDurationMillis {
        // 系统Debug.dumpHprofData方法进行堆转储，保存在yyyy-MM-dd_HH-mm-ss_SSS.hprof
        Debug.dumpHprofData(heapDumpFile.absolutePath)
      }
      if (heapDumpFile.length() == 0L) {
        SharkLog.d { "Dumped heap file is 0 byte length" }
        NoHeapDump
      } else {
        HeapDump(file = heapDumpFile, durationMillis = durationMillis)
      }
    } catch (e: Exception) {
      SharkLog.d(e) { "Could not dump heap" }
      // Abort heap dump
      NoHeapDump
    } finally {
      cancelToast(toast)
      notificationManager.cancel(R.id.leak_canary_notification_dumping_heap)
    }
  }
```





##### 2. HeapAnalyzerService.runAnalysis()

```kotlin
companion object {
    private const val HEAPDUMP_FILE_EXTRA = "HEAPDUMP_FILE_EXTRA"
    private const val HEAPDUMP_DURATION_MILLIS_EXTRA = "HEAPDUMP_DURATION_MILLIS_EXTRA"
    private const val HEAPDUMP_REASON_EXTRA = "HEAPDUMP_REASON_EXTRA"
    private const val PROGUARD_MAPPING_FILE_NAME = "leakCanaryObfuscationMapping.txt"

    fun runAnalysis(
      context: Context,
      heapDumpFile: File,
      heapDumpDurationMillis: Long? = null,
      heapDumpReason: String = "Unknown"
    ) {
      val intent = Intent(context, HeapAnalyzerService::class.java)
      intent.putExtra(HEAPDUMP_FILE_EXTRA, heapDumpFile)
      intent.putExtra(HEAPDUMP_REASON_EXTRA, heapDumpReason)
      heapDumpDurationMillis?.let {
        intent.putExtra(HEAPDUMP_DURATION_MILLIS_EXTRA, heapDumpDurationMillis)
      }
      startForegroundService(context, intent)
    }

    private fun startForegroundService(
      context: Context,
      intent: Intent
    ) {
      if (SDK_INT >= 26) {
        context.startForegroundService(intent)
      } else {
        // Pre-O behavior.
        context.startService(intent)
      }
    }
  }
```

这里面会去启动HeapAnalyzerService，然后会执行到onHandleIntentInForeground()中: 

```kotlin
override fun onHandleIntentInForeground(intent: Intent?) {
    if (intent == null || !intent.hasExtra(HEAPDUMP_FILE_EXTRA)) {
      SharkLog.d { "HeapAnalyzerService received a null or empty intent, ignoring." }
      return
    }

    // Since we're running in the main process we should be careful not to impact it.
    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)
    // 取出传递过来的hprof文件信息
    val heapDumpFile = intent.getSerializableExtra(HEAPDUMP_FILE_EXTRA) as File
    val heapDumpReason = intent.getStringExtra(HEAPDUMP_REASON_EXTRA)
    val heapDumpDurationMillis = intent.getLongExtra(HEAPDUMP_DURATION_MILLIS_EXTRA, -1)

    val config = LeakCanary.config
    // 将解析结果保存到HeapAnalysis中
    val heapAnalysis = if (heapDumpFile.exists()) {
      // 解析该文件  
      analyzeHeap(heapDumpFile, config)
    } else {
      missingFileFailure(heapDumpFile)
    }
    val fullHeapAnalysis = when (heapAnalysis) {
      is HeapAnalysisSuccess -> heapAnalysis.copy(
        dumpDurationMillis = heapDumpDurationMillis,
        metadata = heapAnalysis.metadata + ("Heap dump reason" to heapDumpReason)
      )
      is HeapAnalysisFailure -> heapAnalysis.copy(dumpDurationMillis = heapDumpDurationMillis)
    }
    onAnalysisProgress(REPORTING_HEAP_ANALYSIS)
    // 将解析结果回调回去
    config.onHeapAnalyzedListener.onHeapAnalyzed(fullHeapAnalysis)
  }
```

继续查看analyzeHeap() :  

```kotlin
  private fun analyzeHeap(
    heapDumpFile: File,
    config: Config
  ): HeapAnalysis {
    val heapAnalyzer = HeapAnalyzer(this)

    val proguardMappingReader = try {
      ProguardMappingReader(assets.open(PROGUARD_MAPPING_FILE_NAME))
    } catch (e: IOException) {
      null
    }
    return heapAnalyzer.analyze(
      heapDumpFile = heapDumpFile,
      leakingObjectFinder = config.leakingObjectFinder,
      referenceMatchers = config.referenceMatchers,
      computeRetainedHeapSize = config.computeRetainedHeapSize,
      objectInspectors = config.objectInspectors,
      metadataExtractor = config.metadataExtractor,
      proguardMapping = proguardMappingReader?.readProguardMapping()
    )
  }
```

继续使用Shark库中的HeapAnalyzer.analyze()方法：   

```kotlin
fun analyze(
    heapDumpFile: File,
    leakingObjectFinder: LeakingObjectFinder,
    referenceMatchers: List<ReferenceMatcher> = emptyList(),
    computeRetainedHeapSize: Boolean = false,
    objectInspectors: List<ObjectInspector> = emptyList(),
    metadataExtractor: MetadataExtractor = MetadataExtractor.NO_OP,
    proguardMapping: ProguardMapping? = null
  ): HeapAnalysis {
    val analysisStartNanoTime = System.nanoTime()

    if (!heapDumpFile.exists()) {
      val exception = IllegalArgumentException("File does not exist: $heapDumpFile")
      return HeapAnalysisFailure(
        heapDumpFile = heapDumpFile,
        createdAtTimeMillis = System.currentTimeMillis(),
        analysisDurationMillis = since(analysisStartNanoTime),
        exception = HeapAnalysisException(exception)
      )
    }

    return try {
      listener.onAnalysisProgress(PARSING_HEAP_DUMP)
      // 生成hprof文件中Record的关系图  
      val sourceProvider = ConstantMemoryMetricsDualSourceProvider(FileSourceProvider(heapDumpFile))
      sourceProvider.openHeapGraph(proguardMapping).use { graph ->
        val helpers =
          FindLeakInput(graph, referenceMatchers, computeRetainedHeapSize, objectInspectors)
        // 构建从GC Roots到监测对象的最短引用路径，并返回结果                                                 
        val result = helpers.analyzeGraph(
          metadataExtractor, leakingObjectFinder, heapDumpFile, analysisStartNanoTime
        )
        val lruCacheStats = (graph as HprofHeapGraph).lruCacheStats()
        val randomAccessStats =
          "RandomAccess[" +
            "bytes=${sourceProvider.randomAccessByteReads}," +
            "reads=${sourceProvider.randomAccessReadCount}," +
            "travel=${sourceProvider.randomAccessByteTravel}," +
            "range=${sourceProvider.byteTravelRange}," +
            "size=${heapDumpFile.length()}" +
            "]"
        val stats = "$lruCacheStats $randomAccessStats"
        result.copy(metadata = result.metadata + ("Stats" to stats))
      }
    } catch (exception: Throwable) {
      HeapAnalysisFailure(
        heapDumpFile = heapDumpFile,
        createdAtTimeMillis = System.currentTimeMillis(),
        analysisDurationMillis = since(analysisStartNanoTime),
        exception = HeapAnalysisException(exception)
      )
    }
  }
```

```kotlin
private fun FindLeakInput.analyzeGraph(
    metadataExtractor: MetadataExtractor,
    leakingObjectFinder: LeakingObjectFinder,
    heapDumpFile: File,
    analysisStartNanoTime: Long
  ): HeapAnalysisSuccess {
    listener.onAnalysisProgress(EXTRACTING_METADATA)
    val metadata = metadataExtractor.extractMetadata(graph)

    val retainedClearedWeakRefCount = KeyedWeakReferenceFinder.findKeyedWeakReferences(graph)
      .filter { it.isRetained && !it.hasReferent }.count()

    // This should rarely happens, as we generally remove all cleared weak refs right before a heap
    // dump.
    val metadataWithCount = if (retainedClearedWeakRefCount > 0) {
      metadata + ("Count of retained yet cleared" to "$retainedClearedWeakRefCount KeyedWeakReference instances")
    } else {
      metadata
    }

    listener.onAnalysisProgress(FINDING_RETAINED_OBJECTS)
    val leakingObjectIds = leakingObjectFinder.findLeakingObjectIds(graph)

    val (applicationLeaks, libraryLeaks, unreachableObjects) = findLeaks(leakingObjectIds)

    return HeapAnalysisSuccess(
      heapDumpFile = heapDumpFile,
      createdAtTimeMillis = System.currentTimeMillis(),
      analysisDurationMillis = since(analysisStartNanoTime),
      metadata = metadataWithCount,
      applicationLeaks = applicationLeaks,
      libraryLeaks = libraryLeaks,
      unreachableObjects = unreachableObjects
    )
  }
```













---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
