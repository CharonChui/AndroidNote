Kotlin学习教程(九)
===


`Kotlin`团队为`Android`开发提供了一套超越标准语言功能的工具:    

- `Kotlin Android Extensions`是一个编译器扩展，可以让您摆脱代码中的`findViewById()`调用，并将其替换为合成编译器生成的属性。
- `Anko`是一个提供围绕`Android API`和`DSL`的一组`Kotlin`友好的包装器，可以用`Kotlin`代码替换`layout .xml`文件。


### `Anko`

[Anko](https://github.com/Kotlin/anko)是`Kotlin`官方出品用于`Android`开发的库。


> `Anko is a Kotlin library which makes Android application development faster and easier. It makes your code clean and 
> easy to read, and lets you forget about rough edges of the Android SDK for Java.`


> Anko consists of several parts:
>
> Anko Commons: a lightweight library full of helpers for intents, dialogs, logging and so on;
    > Intents;
    > Dialogs and toasts;
    > Logging;
    > Resources and dimensions;
> Anko Layouts: a fast and type-safe way to write dynamic Android layouts;
> Anko SQLite: a query DSL and parser collection for Android SQLite;
> Anko Coroutines: utilities based on the kotlinx.coroutines library.

`Anko`使用`DSL`提供了很多便捷的功能，可以直接用代码去写布局，不过我还是接受不了，感觉用`xml`写布局把布局和逻辑区分开挺好，这里就不介绍了，想用的可以去看看。 

这里就用几个简单的`commons`中的例子，平时想要启动另一个`activity`我们经常这样写:   
```kotlin
val intent = Intent(this, SomeOtherActivity::class.java)
intent.putExtra("id", 5)
intent.setFlag(Intent.FLAG_ACTIVITY_SINGLE_TOP)
startActivity(intent)
```
这里要四行代码，太多了，`anko`提供了更简单的方式:   
```kotlin
startActivity(intentFor<SomeOtherActivity>("id" to 5).singleTop())
```
如果不设置`flag`的话还可以这样写:   
```kotlin
startActivity<SomeOtherActivity>("id" to 5)
```

而且`anko`还提供了一些常用的`intent`的封装功能:   

- 打电话`makeCall(number) without tel`
- 发短信`sendSMS(number, [text]) without sms`
- 浏览网页`browse(url)`
- 分享`share(text, [subject])`
- 发邮件`email(email, [subject], [text])`

进行`toast`和`snakebar`提示:    
```kotlin
toast("Hi there!")
toast(R.string.message)
longToast("Wow, such duration")

snackbar(view, "Hi there!")
snackbar(view, R.string.message)
longSnackbar(view, "Wow, such duration")
snackbar(view, "Action, reaction", "Click me!") { doStuff() }
```

对话框:   
```kotlin
alert("Hi, I'm Roy", "Have you tried turning it off and on again?") {
    yesButton { toast("Oh…") }
    noButton {}
}.show()

// 列表对话框
val countries = listOf("Russia", "USA", "Japan", "Australia")
selector("Where are you from?", countries, { dialogInterface, i ->
    toast("So you're living in ${countries[i]}, right?")
})

// 进度对话框
val dialog = progressDialog(message = "Please wait a bit…", title = "Fetching data")
```

`log`：  
```kotlin
class SomeActivity : Activity(), AnkoLogger {
    private fun someMethod() {
        info("London is the capital of Great Britain")
        debug(5) // .toString() method will be executed
        warn(null) // "null" will be printed
    }
}
```
或者:   
```kotlin
class SomeActivity : Activity() {
    private val log = AnkoLogger<SomeActivity>(this)
    private val logWithASpecificTag = AnkoLogger("my_tag")

    private fun someMethod() {
        log.warning("Big brother is watching you!")
    }
}
```

`dimens`: 

可以直接使用`px2dip`和`px2sp`来进行尺寸转换。  


异步操作:   
```kotlin
doAsync {
    // Long background task
    uiThread {
        result.text = "Done"
    }
}
```

使用`anko`:    
```kotlin
// 创建一个verticallayout并且添加edittext和button，并给button设置点击事件
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```


### Kotlin Android Extensions

相信每一位安卓开发人员对`findViewById()`这个方法再熟悉不过了，毫无疑问，潜在的`bug`和脏乱的代码令后续开发无从下手的。 尽管存在一系列的
开源库能够为这个问题带来解决方案，然而对于运行时依赖的库，需要为每一个`View`注解变量字段。

现在`Kotlin`安卓扩展插件能够提供与这些开源库功能相同的体验，不需要添加任何额外代码，也不影响任何运行时体验。
另一个`Kotlin`团队研发的可以让开发更简单的插件是`Kotlin Android Extensions`。当前仅仅包括了`view`的绑定。这个插件自动创建了很多的属性
来让我们直接访问`XML`中的`view`。这种方式不需要你在开始使用之前明确地从布局中去找到这些`views`。

这些属性的名字就是来自对应`view`的`id`，所以我们取`id`的时候要十分小心，因为它们将会是我们类中非常重要的一部分。这些属性的类型也是来自
`XML`中的，所以我们不需要去进行额外的类型转换。

`Kotlin Android Extensions`的一个优点是它不需要在我们的代码中依赖其它额外的库。它仅仅由插件组层，需要时用于生成工作所需的代码，只需要依赖
于`Kotlin`的标准库。

开发者仅需要在模块的`build.gradle`文件中启用`Gradle`安卓扩展插件即可:   

```
apply plugin: 'kotlin-android-extensions'
```

我们在使用`Android Sutdio 3.0`创建`Kotlin`项目时默认已经添加了该依赖。   

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
`textView`是对`Activity`的一项扩展属性，与在`activity_main.xml`中的声明具有同样类型。


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

如你所知，`HTTP`请求不被允许在主线程中执行，否则它会抛出异常。这是因为阻塞住`UI`线程是一个非常差的体验。`Android`中通用的做法是使用
`AsyncTask`，但是这些类是非常丑陋的，并且使用它们无任何副作用地实现功能也是非常困难的。`Anko`提供了非常简单的`DSL`来处理异步任务，
它满足大部分的需求。它提供了一个基本的`async`函数用于在其它线程执行代码，也可以选择通过调用`uiThread`的方式回到主线程。
在子线程中执行请求如下这么简单:  
```kotlin
async() {
    Request(url).run()
    uiThread { longToast("Request performed") }
}
```
`UIThread`有一个很不错的一点就是可以依赖于调用者。如果它是被一个`Activity`调用的，那么如果`activity.isFinishing()`返回`true`，
则`uiThread`不会执行，这样就不会在`Activity`销毁的时候遇到崩溃的情况了。

##### 解析`gson`

`Kotlin`允许我们去定义一些行为与静态对象一样的对象。尽管这些对象可以用众所周知的模式来实现，比如容易实现的单例模式。
我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用`companion objecvt`。这个对象被这个类的所有对象所共享，就像`Java`中的静态
属性或者方法。
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



参考
===

- [Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers)
- [Resources to Learn Kotlin](https://developer.android.com/kotlin/resources.html)
- [Kotlin语言中文站](https://www.kotlincn.net/docs/reference/coding-conventions.html)


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

