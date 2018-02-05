InstantRun详解
===

之前在写[AndroidStudio提高Build速度][1]]这篇文章的时候写到，想要快，就用`Instant Run`。最近有朋友发来邮件讨论它的原理，最近项目不忙，索性就来系统的学习下。


`Android Studio`2.0开始引入了`Instant Run`，它主要是在`Run`和`Debug`的时候可以去减少更新应用的时间。虽然第一次`Build`的时候可能会消耗稍长的时间来完成，但是`Instant Run`可以把更新内容推送到设备上，而无需重新`build`一个新的`apk`，这样就会很快速的让我们观察到改变。

`Instant Run`只支持在`build.gralde`文件中配置的`Gradle`版本是2.0.0以上并且`minSdkVersion`是15以上才可以。为了能更好的使用，请将`minSdkVrsion`设置到21以上。

部署完应用后，会在`Run`![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/as-irrun.png?raw=true)(或者(Debug![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/as-irdebug.png?raw=true)))图标上面出现一个小黄色的闪电符号，这就意味着`Instant Run`已经准备就绪，在你下次点击按钮的时候可以推送更新内容。它不需要重新构建一个新的`APK`,它只推送那些新改变的地方，有些情况下`app`都不需要重启就可以直接显示新改变的下效果。


### 配置你的应用来使用`Instant Run`

`Android Stuido`中项目使用`Gralde`2.0.0及以上版本会默认使用`Instant Run`。  
让项目更新到最新的版本:     

1. 点击`Settings`中的`Preferences`按钮。
2. 找到`Build,Execution,Deployment`中的`Instant Run`然后点击`Update Project`，如果`Update Project`部分没有显示，那说明你当前已经是最新版本了.   
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/update-project-dialog.png?raw=true)



### 修复类型

`Instant Run`通过热修复、暖修复或者冷修复来推送改变的代码和资源到你的设备或者模拟器上。它会根据你更改的内容来自动选择对应的修复类型。

- 热修复: 更改的内容不需要重启应用甚至不需要重启当前的`activity`就可以让显示内容的改变，对于大部分通过方法内容改变的更改都可以通过这种方式来修改。
- 暖修复: 需要重启`activity`才能让改变生效，在改变资源时会通过该方式。
- 冷修复: 应用需要重启(不是重新安装)。对于一些方法方法结构和实现等结构性的改变需要通过该方式。    



那具体更改内容和修复方式是怎么对应的呢？    

- 改变现在已经存在的方法          
    这将通过热修复来执行，这是最快的一种修复方式。你的应用会在运行的过程中，在下次调用该方法时直接使用新实现的方法。
    热修复不需要重新初始化对象。在看到具体的更改之前，你可能会需要重启当前的`activity`或者应用。默认情况下`Android Studio`在执行热修复后会自动重启当前的`activity`。如果你不想要它自动重启`activity`你也可以在设置中禁用。

- 更改或者移除已经存在的资源           
    这将通过暖修复来执行:这也是非常快的，但是`Instant Run`在推送这些更新内容时必须要重启当前的`activity`。你的应用仍然会继续运行，但是在重启`activity`时屏幕可能会闪动-这是正常滴。

- 结构性的改变例如:    

    - 增删或者改变:    
        - 一个注解
        - 一个变量
        - 一个静态变量
        - 一个方法结构
        - 一个静态方法结构

    - 更改该类集成的父类      
    - 更改接口的实现列表
    - 更改类的静态修饰符
    - 使用动态资源`id`重新布局

    上面的这些改变都将通过冷修复(API 21以上才支持)来执行.冷修复会稍微有些慢，因为虽然不需要构建一个新的`APK`但是必须要重启整个`app`才能推送这些结构性的代码改变。对于`API` 20及以下的版本，`Android Studio`将重新部署`APK`。

- 更改`manifest`文件或者更改`manifest`文件中引用的资源或者更改`Android`桌面`widget`的`UI`实现(需要Clean and Rerun)               
    在更改清单文件或者清单文件中引用的资源时，`Android Studio`会自动的重新部署引用来实现这些内容的改变。这是因为例如应用名称、图标和`intent filters`这些东西是要在应用安装到设备时根据清单文件来决定的。
    在更新`Android UI widget`时，你需要执行`Clean and Rerun`才能看到更改的内容，因为在使用`Instant Run`时执行`clean`会需要很长的时间，所以你在更新`UI widget`时可以禁用`Instant Run`。 


##### 使用Rerun
  
如果修改了一些会影响到初始化的内容时，例如修改了应用的`onCreate()`方法，你需要重启你的应用来让这些改变生效，你可以点击`Rerun`图标。它将会停止应用运行，并且执行`clean`操作后重新部署一个新的`APK`到你的设备。   


### 实现原理      

正常情况下，我们修改了内容，想让让它生效，那就需要如下的步骤:           

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/change_run_list.png?raw=true)

那`Instant Run`的目标也非常简单:         
> 尽可能多的移除上面的步骤，来让剩下的部分尽可能更快。    

这就意味着:    

- 只构建和部署新更改的部分。
- 不去重新安装应用。
- 不去重启应用。
- 甚至不去重启`activity`。



![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/instant_run_list.png?raw=true)


普通情况下在你点击`Run`或者`Debug`按钮时，会执行如下的操作:     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/apk_progress.png?raw=true)     

清单文件会被整合然后与你应用的资源重启打包成`APK`.同样的，`.java`的源代码会被编译成二进制，然后转换成`.dex`文件，他们也会被包含到`APK`中。

在使用`Instant Run`的情况下，第一次点击`Run`和`Debug`时`Gradle`会执行一些额外的操作:     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/instant_apk_progress.png?raw=true)
`instrumentation`相关的二进制内容会被增加到`.class`文件中，并且一个新的`App Server`类会被注入到应用中。    
同时也会增加一个新的`Application`类，来注入一些自定义的`class loader`并且能启动`App Server`。因此`minifest`文件会被修改来保证使用该新加的`Application`类(如果你已经创建了自己的`Application`类，`Instant Run`的版本将会用你定义的来进行代理扩展) 

经过上面的操作`Instant Run`就开始执行了，以后如果你改变了代码部分，在重新点击`Run`或者`Debug`时，`Instant Run`将会通过热、暖、冷修复来尽可能的减少上面的构建过程。    

在`Instant Run`更改内容前，`Android Studio`会你的应用版本是否支持`Instant Run`并且`App Server`是否运行在对其有用的端口(内部使用了Socket)。 这样来确定你的应用是否在前台运行，并且找到`Studio`所需要的构建`ID`.   

总结一下，其实就是内部会对每个`class`文件注入`instant run`相关的代码，然后自定义一个`application`内部指定自定义的`classLoader`(也就是说不使用默认的`classLoader`了，只要用了`Instant Run`就需要使用它自定义的`classLoader`)，然后在应用程序里面开启一个服务器，`Studio`将修改的代码发送到该服务器，然后再通过自定义的`classLoader`加载代码的时候会去请求该服务器判断代码是否有更新，如果有更新就会通过委托机制加载新更新的代码然后注入到应用程序中，这样就完成了替换的操作。

### 热修复过程               


![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/hot_swapping.png?raw=true)

`Android Studio`会监听在开发过程中哪个文件发生了改变，并且通过一个自定义的`Gralde task`来只对修改的`class`文件生成对应的`.dex`文件。    
这些新的`.dex`文件会通过`Studio`传递发布给应用中运行的`App Server`。  

因为开始时这些文件的`class`已经存在到运行中的应用中-`Gralde`需要保证更新的版本来保证他们能覆盖之前已经存在的文件。这些更新文件的转换是由`App Server`通过自定义的`class loader`来完成的。    

从现在开始，每次一个方法被调用时(不管在应用中的任何地方)，被注入到最初的`class`文件中的`instrumentation`都讲去连接`App Server`来检查它们是否有更新了。 如果有更新，执行操作将被指派到新的充在的`class`文件，这样就会执行新修改的方法。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/instant_run_app_server.png?raw=true)

如果你设置断点，你会发现你调用的是`override`名字的类中的方法。
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/instant_debug_override.gif?raw=true)


这种重新指定方法的方式在改变方法实现时非常有效，但是对于那种需要在`Activity`启动时加载的改变呢？   

### 暖修复      

暖修复会重启`Activity`。资源文件会在`Activity`启动时被加载，所以修改它们需要`activity`重新启动来进行加载。    

现在，更改任何资源都会导致所有的资源都被重新打包病转移到应用中-但是以后会支持增量包的功能来让只打包和部署那些新修改的资源。

> 注意，暖修复在更改`manifest`件或者`manifest`文件中引用的内容时无效，因为`manifest`文件中的内容需要在`apk`安装时读取。更改`Manifest`文件(或者它所引用的内容)需要被全部构建和部署。    



非常不幸的是，重启`activity`在做一些结构性的改变时无效。对于这些改变需要使用冷修复。   

### 冷修复     

在部署时，你的应用和他的子项目会被分派到10个不同的部分，每个都在自己单独的`dex`文件。不同部分的类会通过包名来分配。在使用冷修复时，修改的类需要其他所有相关的`class`文件都被重新打包后才能被部署到设备上。      
这就需要依赖`Android Runtime`能支持加载多个`.dex`文件,这是一个在`ART`中新增的功能，它只有在`Android 5.0(API 21)`以上的设备总才支持。    
对于`API 20`一下的设备，他们仍在使用`Dalvik`虚拟机，`Android Studio`需要部署整个`APK`。      



有些时候通过热修复来改变的代码，但是它会被应用首次运行时的初始化所影响，这时你就需要`restart`你的应用来让其生效(快捷键是Command + Ctrl + R)。    
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/instant_run_restart.gif?raw=true)


### Instant Run使用技巧及提示      

`Instant Run`是由`Android Studio`控制的，所以你只能通过`IDE`来`start/restart`你的`debug`实例，不要直接在设备中`start/restart`你的应用，不然的话就会发生错乱。    


##### Instant Run限制条件    

- 部署到多个设备

    `Instant Run`针对设备的不同`API Level`使用不同的技术来进行热、暖、冷修复。因此，如果同时部署一个应用到多个设备时，`Studio`会暂时关闭`Instant Run`功能。 


- Multidex

    如果你的项目支持传统的`Multidex`-也就是在`build.gradle`中配置了`multiDexEnabled true`和`minSdkVersion 20`及以下-当你部署应用到`4.4`以下的设备上时，`Android Studio`将会禁用`Instant Run`。   

    如果`minSdkVersion`设置为21或者更高时，`Instant Run`将自动配置来支持`multidex`，因为`Instant Run`只支持`debug`版本，有需要在发布`release`版的时候配置你的`build`变量来支持`multidex`。    

- 使用第三方插件 
    `Android Studio`在使用`Instant Run`时会暂时禁用`Java Code Coverage Library`和`ProGuard`。因为`Instant Run`只支持`debug`版本，这样也不会影响`release`版的构建。    

- 只能在主进程中运行    
    目前热修复只能在主进程中进行，如果在其他进程中热、暖修复将无法使用，只能用冷修复来替代。  



参考:         

- [官网](https://developer.android.com/studio/run/index.html#instant-run)
- [Instant Run: How Does it Work?!](https://medium.com/google-developers/instant-run-how-does-it-work-294a1633367f#.8xmpk8xvc)
    	

[1]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E6%8F%90%E9%AB%98Build%E9%80%9F%E5%BA%A6.md "AndroidStudio提高Build速度"

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 