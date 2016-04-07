ApplicationId vs PackageName
===

曾几何时，自从转入`Studio`阵营后就发现多了个`applicationId "com.xx.xxx"`，虽然知道肯定会有区别，但是我却没有仔细去看，只想着把它和`packageName`设置成相同即可(原谅我的懒惰- -!)。         
直到今天在官网看`Gradle`使用时，终于忍不住要搞明白它俩的区别。     


http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename



所有的 Android 应用程序都有一个包名。包名是设备上的这个应用程序的唯一标识，也是在谷歌Play商店上的唯一标识。这意味着，一旦你已发布的程序使用了这个包名， 你就永远都无法改变它；否则会导致你的应用程序被当作是一个全新的应用程序，你之前的应用程序的用户将不会看到作为更新的安装包。
在此前Android Gradle 构建系统中，您的应用程序的包名由你的manifest文件的根元素里的package属性决定： 
AndroidManifest.xml: <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.example.my.app" android:versionCode="1" android:versionName="1.0" > 
然而，这里所定义的包也有第二个目的：它被用来命名你的资源类的包（以及解析任何相关的Activity的类名）。在上面的示例中，生成的 R 类将会是com.example.my.app.R，因此如果您其他包里面的代码需要引用这些资源，就需要导入com.example.my.app.R。 
使用新的 Android Gradle 构建系统，你可以轻松构建多个不同版本的应用程序；例如，您可以构建一个“free”版本和“pro”版本的应用程序 （通过使用flavors），并且这些不同版本的程序在 Google Play 商店上应该有不同的包，这样他们可以被单独安装和购买，或者是同时安装两个，等等。同样，您还可以同时创建“debug”、“alpha”和“beta”版本的应用程序 （使用build types），而这些版本的程序同样可以使用唯一的包名。 
同时，您要在代码中导入的 R 类必须在这段时间内保持不变 ；在您正在构建您的应用程序的不同版本时您的.java 源文件不应该被更改。 
因此，我们解耦了包名称的两种用法：
最终的方案是，在您生成的.apk 的manifest 中，并且用于在你的设备和 Google Play 商店来标识你的应用的包，叫做“application id”。
用于在源代码中来引用您的R类的，并且是解析任何相关的Activity/Service 注册的包，继续被称为“package”。
你可以在你 gradle 文件中，指定application id，如下所示： 
app/build.gradle: apply plugin: 'com.android.application' 
android { compileSdkVersion 19 buildToolsVersion "19.1" 
defaultConfig { applicationId "com.example.my.app" minSdkVersion 15 targetSdkVersion 19 versionCode 1 versionName "1.0" } ... 
像以前一样，你需要在 Manifest 文件中指定用于代码的包，就如上面的Andr 
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 