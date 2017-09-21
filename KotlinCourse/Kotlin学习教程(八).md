Kotlin学习教程(八)
===


### `Anko`

[Anko](https://github.com/Kotlin/anko)是`Kotlin`官方出品用于`Android`开发的库。


> `Anko is a Kotlin library which makes Android application development faster and easier. It makes your code clean and easy to read, and lets you forget about rough edges of the Android SDK for Java.`


> Anko consists of several parts:
>
> Anko Commons: a lightweight library full of helpers for intents, dialogs, logging and so on;
> Anko Layouts: a fast and type-safe way to write dynamic Android layouts;
> Anko SQLite: a query DSL and parser collection for Android SQLite;
> Anko Coroutines: utilities based on the kotlinx.coroutines library.


一个Anko扩展函数可以被用来简化获取一个RecyclerView：
```kotlin
val forecastList: RecyclerView = find(R.id.forecast_list)
```



### Kotlin Android Extensions

相信每一位安卓开发人员对 findViewById() 这个方法再熟悉不过了，毫无疑问，潜在的 bug 和脏乱的代码令后续开发无从下手的。 尽管存在一系列的开源库能够为这个问题带来解决方案，然而对于运行时依赖的库，需要为每一个 View 注解变量字段。

现在 Kotlin 安卓扩展插件能够提供与这些开源库功能相同的体验，不需要添加任何额外代码，也不影响任何运行时体验。
另一个Kotlin团队研发的可以让开发更简单的插件是Kotlin Android Extensions。当前仅仅包括了view的绑定。这个插件自动创建了很多的属性来让我们直接访问XML中的view。这种方式不需要你在开始使用之前明确地从布局中去找到这些views。

这些属性的名字就是来自对应view的id，所以我们取id的时候要十分小心，因为它们将会是我们类中非常重要的一部分。这些属性的类型也是来自XML中的，所以我们不需要去进行额外的类型转换。

Kotlin Android Extensions的一个优点是它不需要在我们的代码中依赖其它额外的库。它仅仅由插件组层，需要时用于生成工作所需的代码，只需要依赖于Kotlin的标准库。

开发者仅需要在模块的 build.gradle 文件中启用 Gradle 安卓扩展插件即可：

apply plugin: 'kotlin-android-extensions'


默认创建`Kotlin`项目时已经添加了该依赖。   

```kotlin
// 使用来自主代码集的 R.layout.activity_main
import kotlinx.android.synthetic.main.activity_main.*

class MyActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textView.setText("Hello, world!")
        // 而不是 findViewById(R.id.textView) as TextView
    }
}
```
textView 是对 Activity 的一项扩展属性，与在 activity_main.xml 中的声明具有同样类型。













### 网络请求    


`Kotlin`提供了一些扩展函数来执行网络请求，同时也可以使用第三方库来执行.     
```kotlin
public class Request(val url: String) {
    public fun run() {
        val forecastJsonStr = URL(url).readText()
        Log.d(javaClass.simpleName, forecastJsonStr)
    }
}
```

如你所知，HTTP请求不被允许在主线程中执行，否则它会抛出异常。这是因为阻塞住UI线程是一个非常差的体验。Android中通用的做法是使用AsyncTask，但是这些类是非常丑陋的，并且使用它们无任何副作用地实现功能也是非常困难的。Anko提供了非常简单的DSL来处理异步任务，它满足大部分的需求。它提供了一个基本的async函数用于在其它线程执行代码，也可以选择通过调用uiThread的方式回到主线程。在子线程中执行请求如下这么简单：
```kotlin
async() {
    Request(url).run()
    uiThread { longToast("Request performed") }
}
```
UIThread有一个很不错的一点就是可以依赖于调用者。如果它是被一个Activity调用的，那么如果activity.isFinishing()返回true，则uiThread不会执行，这样就不会在Activity销毁的时候遇到崩溃的情况了。

##### 解析`gson`

Kotlin允许我们去定义一些行为与静态对象一样的对象。尽管这些对象可以用众所周知的模式来实现，比如容易实现的单例模式。
我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用companion objecvt。这个对象被这个类的所有对象所共享，就像Java中的静态属性或者方法。
```kotlin
public class ForecastRequest(val zipCode: String) {
    companion object {
        private val APP_ID = "15646a06818f61f7b8d7823ca833e1ce"
        private val URL = "http://api.openweathermap.org/data/2.5/" +
                "forecast/daily?mode=json&units=metric&cnt=7"
        private val COMPLETE_URL = "$URL&APPID=$APP_ID&q="
    }
    
    fun execute(): ForecastResult {
        val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
        return Gson().fromJson(forecastJsonStr, ForecastResult::class.java)
    }
}
```


### 创建Application 

```kotlin
class App : Application() {
    companion object {
        private var instance: Application? = null
        fun instance() = instance!!
    }
    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}

```




- with() 

```kotlin
inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }
```

- 扩张



https://leanpub.com/kotlin-for-android-developers

https://developer.android.com/kotlin/resources.html

https://www.kotlincn.net/docs/reference/coding-conventions.html


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

