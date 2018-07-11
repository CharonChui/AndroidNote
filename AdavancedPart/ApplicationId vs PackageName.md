ApplicationId vs PackageName
===

曾几何时，自从转入`Studio`阵营后就发现多了个`applicationId "com.xx.xxx"`，虽然知道肯定会有区别，但是我却没有仔细去看，只想着把它和`packageName`设置成相同即可(原谅我的懒惰- -!)。         

直到今天在官网看`Gradle`使用时，终于忍不住要搞明白它俩的区别。         

在`Android`官方文档中有一句是这样描述`applicationId`的:`applicationId : the effective packageName`，真是言简意赅，那既然`applicationId`是有效的包明了，`packageName`算啥？     

所有`Android`应用都有一个包名。包名在设备上能唯一的标示一个应用，它在`Google Play`应用商店中也是唯一的。这就意味着一旦你使用一个包名发布应用后，你就永 远不能改变它的包名；如果你改了包名就会导致你的应用被认为是一个新的应用，并且已经使用你之前应用的用户将不会看到作为更新的新应用包。          

之前的`Android Gradle`构建系统中，应用的包名是由你的`manifest`文件中的根元素中的`package`属性定义的:       
 
`AndroidManifest.xml: `          

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.my.app"
    android:versionCode="1"
    android:versionName="1.0" >
```
然而，这里定义的包也有第二个目的：就是被用来命名你的`R`资源类(以及解析任何与`Activities`相关的类名)的包。在上面的示例中，生成的`R`类就是`com.example.my.app.R`，所以如果你在其他的包中想引用资源，就需要导入`com.example.my.app.R`。      
  
伴随着新的`Android Gradle`构建系统，你可以很简单的为你的应用构建多个不同的版本；例如，你可以同时为你的应用构建一个免费版本和一个专业版(使用`flavors`)，并且他们应该在`Google Play`商店中有不同的包，这样才能让他们可以被单独安装和购买，同时安装两个，等等。同样的你也可能同时为你的应用构建`debug`版、`alpha`版和`beta`版(使用`build types`)，这些也可以同样使用不同的包名。    
 
在这同时，你在代码中导入的`R`类必须一直保持一直；在为应用构建不同的版本时`.java`源文件都不应该发生变化。
       
因此,我们解耦了`package name`的两种用法:       

- 在生成的`.apk`中的`manifest`文件中使用的最终的包名以及在你的设备和`Google Play`商店中用来标示你的包名叫做`application id`的值。
- 在源代码中指向`R`类的包名以及在解析任何与`activity/service`注册相关的包名继续叫做`package name`。

可以在`gradle`文件中像如下指定`application id`:  
    
`app/build.gradle:`         

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "19.1"

    defaultConfig {
        applicationId "com.example.my.app"
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    ...
```

像之前一样，你需要在`Manifest`文件中指定你在代码中使用的`package name`，像上面`AndroidManifest.xml`的例子。        
下面进入关键部分了:当你按照上面的方式做完后，这两个包就是相互独立的了。你现在可以很简单的重构你的代码-通过修改`Manifest`中的包名来修改在你的`activitise`和`services`中使用的包和在重构你在代码中的引用声明。这不会影响你应用的最终`id`，也就是在`Gradle`文件中的`applicationId`。       

你可以通过以下`Gradle DSL`方法为应用的`flavors`和`build types`指定不同的`applicationId`:       

`app/buid.gradle: `      

```
productFlavors {
    pro {
        applicationId = "com.example.my.pkg.pro"
    }
    free {
        applicationId = "com.example.my.pkg.free"
    }
}

buildTypes {
    debug {
        applicationIdSuffix ".debug"
    }
}
....
```
(在`Android Studio`中你也可以通过图形化的`Project Structure`的对话框来更改上面所有的配置)

注意:为了兼容性，如果你在`build.gradle`文件中没有定义`applicationId` ，那`applicationId`就是与`AndroidManifest.xml`中配置的包名相同的默认值。在这种情况下，这两者显然脱不了干系，如果你试图重构代码中的包就将会导致同时会改变你应用程序的`id`！在`Android Studio`中新创建的项目都是同时指定他们俩。     

注意2:`package name`必须在默认的`AndroidManifest.xml`文件中指定。如果有多个`manifest`文件(例如对每个`flavor`制定一个`manifest`或者每个`build type`制定一个`manifest`)时，`package name`是可选的，但是如果你指定的话，它必须与主`manifest`中指定的`pakcage`相同。

		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 