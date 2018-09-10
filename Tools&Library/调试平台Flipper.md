调试平台Flippe

===

子日:工欲善其事必先利其器

[Flipper](https://fbflipper.com/docs)是之前`Facebook`的一个内部调试工具，旨在帮助开发人员以交互式和可扩展的方式检查和理解`iOS`及`Android`应用程序的结构和行为。

根据`Facebook`工程师`EmilSjölander`的说法，`Flipper`基于[Stetho](http://facebook.github.io/stetho/)的经验基础而构建，`Stetho`是一个`Android`调试桥，允许开发人员使用`Chrome DevTools`调试他们的应用程序。

`Facebook`推荐开发者使用`Flipper`来替代`Stetho`，除非是还没有从`Stetho`移植到`Flipper`的一些功能，例如基于`Dumper`的命令行工具。


前段时间`facebook`将`Flipper`开源了。最开始的时候是叫做`Sonar`后来改成`Flipper`了。


这里就以`android`应用为例简单介绍一下它的使用。 
`Flipper`包括两部分:  

- 桌面应用
- `Flipper SDK`:移动应用程序需要集成`Flipper SDK`，`Flipper SDK`负责与基于`Electron`的桌面应用程序通信，以显示调试数据。

在扩展性方面，`Flipper`提供了一个插件`API`，开发人员可以使用这组`API`创建自己的插件来可视化和调试应用程序数据。`Flipper`初始版本包含许多即用型插件，例如:

- `Logs`:用于检查应用程序的系统日志，默认集成的，不用手动添加。
- `Layout Inspector`:用于检查`iOS`和`Android`应用程序的布局。
- `Network Inspector`:用于检查网络流量。
- `SharedPreferece`:用于查看`sharedpreference`文件的。

这些只是`Flipper`提供的一些基本的功能。根据`Sjölander`的说法，`Facebook`工程师还开发了插件来监控`GraphQL`请求、跟踪性能标记等。


下面介绍一下如何使用：   

### 首先，去官网下载桌面应用，下载完后是这个样子的。

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/sonar_desktop.png?raw=true" width="100%" height="100%">

我们可以看到左边只有`log`的部分。


### 接下来在`app`中嵌入`flipper`的`sdk`

- 首先需要保证已声明如下权限:   
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

- 然后在`build.gradle`文件中添加依赖

```java
repositories {
  jcenter()
}

dependencies {
  debugImplementation 'com.facebook.flipper:flipper:0.6.18'
}
```

- 自定义一个`Application`，并在`onCreate()`方法中初始化`flipper`

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        SoLoader.init(this, false);

        if (BuildConfig.DEBUG && SonarUtils.shouldEnableSonar(this)) {
            final SonarClient client = AndroidSonarClient.getInstance(this);
            client.addPlugin(new NetworkSonarPlugin());
            client.addPlugin(new SharedPreferencesSonarPlugin(this));
            client.start();
        }
    }
}
```

上面我们添加了两个`plugin`，我们执行看一下:   

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/sonar_add_plugin.png?raw=true" width="100%" height="100%">

看到了吗？左边现在有`log`、`network`和`sharedpreference`三个部分了。 但是这三个是远远不够的啊，我还想查看布局。 

这里需要注意一下，如果使用了`okhttp`,可以使用拦截器系统自动`hook`到现在的堆栈。

```java
import com.facebook.sonar.plugins.network.SonarOkhttpInterceptor;

new OkHttpClient.Builder()
    .addNetworkInterceptor(new SonarOkhttpInterceptor(networkSonarPlugin))
    .build();
```

- 增加对查看布局的支持

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        SoLoader.init(this, false);

        if (BuildConfig.DEBUG && SonarUtils.shouldEnableSonar(this)) {
            final SonarClient client = AndroidSonarClient.getInstance(this);
            client.addPlugin(new NetworkSonarPlugin());
            client.addPlugin(new SharedPreferencesSonarPlugin(this));
            // 增加布局plugin
            final DescriptorMapping descriptorMapping = DescriptorMapping.withDefaults();
            client.addPlugin(new InspectorSonarPlugin(this, descriptorMapping));
            client.start();
        }
    }
}
```

执行一下:  
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/sonar_add_plugun_layout.png?raw=true" width="100%" height="100%">


- 沙箱插件

这个我没搞懂到底是干什么用的，也没用过。

> The Sandbox plugin is useful for developers that had to test changes of their apps by pointing them to some Sandbox environment. Through this plugin and a few lines of code in the client, the app can get a callback and get the value that the user has input through Sonar. At this point, the developer can plugin its logic to save this setting in its app.

```java
import com.facebook.sonar.plugins.SandboxSonarPlugin;
import com.facebook.sonar.plugins.SandboxSonarPluginStrategy;

final SandboxSonarPluginStrategy strategy = getStrategy(); // Your strategy goes here
client.addPlugin(new SandboxSonarPlugin(strategy));
```
 
在上面的代码中，我们都是直接通过`addPlugin()`方法去添加的，我们看一下系统提供的`Plugin`。 
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/sonar_plugin_list.png?raw=true" width="100%" height="100%">

其实我们也可以自定义`Plugin`，自定义一个类实现`SonarPlugin`接口，然后去发送或者接受数据，一般也用不到，这里就不说了:
```java
public class MySonarPlugin implements SonarPlugin {
  private SonarConnection mConnection;

  @Override
  public String getId() {
    return "MySonarPlugin";
  }

    @Override
    public void onConnect(SonarConnection connection) throws Exception {
        mConnection = connection;
        // Using the SonarConnection object you can register a receiver of a desktop method call and respond with data.
        connection.receive("getData", new SonarReceiver() {
            @Override
            public void onReceive(SonarObject params, SonarResponder responder) throws Exception {
                responder.success(
                        new SonarObject.Builder()
                                .put("data", MyData.get())
                                .build());
            }
        });

        // You don't have to wait for the desktop to request data though, you can also push data directly to the desktop.
        connection.send("MyMessage",
                new SonarObject.Builder()
                        .put("message", "Hello")
                        .build());



    }

  @Override
  public void onDisconnect() throws Exception {
    mConnection = null;
  }
}
```



---

- 邮箱 ：charon.chui@gmail.com  
-
Good Luck! 
