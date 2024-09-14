# Jetpack Compose简介



Jetpack Compose是用于构建原生Android UI的现代工具包。 Jetpack Compose使用更少的代码，强大的工具和直观的Kotlin API，简化并加速了Android上的UI开发。这是Android Developers 官网对它的描述。



在Android中，UI工具包的历史可追溯到至少10年前。自那时以来，情况发生了很大变化，例如我们使用的设备，用户的期望，以及开发人员对他们所使用的开发工具和语言的期望。

以上只是我们需要新UI工具的一个原因，另外一个重要的原因是`View.java`这个类实在是太大了，有太多的代码，它大到你甚至无法在Githubs上查看该文件，因为它实际上包含了`30000`行代码，这很疯狂，而我们所使用的几乎每一个Android UI 组件都需要继承于View。

GogleAndroid团队的Anna-Chiara表示，他们对已经实现的一些API感到遗憾，因为他们也无法在不破坏功能的情况下收回、修复或扩展这些API，因此现在是一个崭新起点的好时机。

这就是为什么Jetpack Compose 让我们看到了曙光。


### 精简代码

编写更少的代码会影响到所有开发阶段:作为代码撰写者，需要测试和调试的代码会更少，出现bug的可能性也更小。

与使用Android View系统(按钮、列表或动画)相比，Compose可让您使用更少的代码实现更多的功能。      
而且编写代码只需要采用Kotlin，而不必拆分成Kotlin和XML部分。

### 直观易读

Compose使用声明性API，这意味着您只需描述界面，Compose会负责完成其余工作。    


### 加速开发

Compose与您所有的现有代码兼容：您可以从View调用Compose代码，也可以从Compose调用View。    


### 旧项目支持Compose

以Android平台而言，Compose实际上都承载在ComposeView上，如果想要在旧项目中使用Compose开发，就需要在使用处添加一个ComposeView。   

可以在XML中静态声明或在程序中动态构造出一个ComposeView实例，例如在activity_main.xml中添加ComposeView，如下所示:      

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.compose.ui.platform.ComposeView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/root"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</androidx.compose.ui.platform.ComposeView>
```

```kotlin
class MainActivity2 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main2)
        findViewById<ComposeView>(R.id.root).setContent {
            Text("Hello World")
        }
    }
}
```


### 简单的可组合函数

使用Compose，您可以通过定义一组接受数据而发出界面元素的可组合函数来构建界面。     
下面是一个简单的示例，接受一个字符串并现实一个问号信息:    

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name)
}
```

- 此函数带有@Composable注释。所有可组合函数都必须带有此注释。此注释可告知Compose编译器:此函数旨在将数据转换为界面。     
- 此函数可以在界面中显示文本。为此，它会调用Text()可组合函数，该函数实际上会创建文本界面元素。可组合函数通过调用其他可组合函数来发出界面层次结构。 



### 在Composet中使用View组件

不少功能性的传统视图控件在Compose中没有对应的Composable实现，例如SurfaceView、
WebView、MapView等。     

因此在Compose中可能会有使用传统View控件的需求。     
Compose提供了名为AndroidView的Composable组件，允许在Composable中插入任意基于继承自View的传统试图控件。   

```kotlin
class MainActivity2 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AndroidView(factory = {context ->
                WebView(context).apply {
                    settings.javaScriptEnabled = true
                    webViewClient = WebViewClient()
                    loadUrl("https://www.baidu.com")
                }
            }, modifier = Modifier.fillMaxSize())
        }
    }
}
```
AndroidView中传入一个工厂的实现，工厂方法中可以获取Composable所在的Context，并基于其构建View视图控件。  

### Modifier

Modifier是一个装饰或者添加行为的有序的，不变的集合。     
例如background、padding、宽高、焦点点击事件等。   
例如给Text设置单行、给Button设置个中点击状态等行为。    
其实就是所有控件的通用属性都在Modifier中。  


在Compose中，每个组件都是一个带有@Composable注解的函数，被称为Composable。   
Compose中已经预置了很多基础的Composable组件，他们都是基于Material Design规范设计，例如Button、TextField、TopAppBar等。   

在布局方面，Compose提供了Column、Row、Box三种布局组件，类似于传统视图开发中的LinearLayout(Vertical)、LinearLayout(Horizontal)、RelativeLayout。   
