AndroidStudio使用教程(第五弹)
===

Create and Build an Android Studio Project
---

接下来是以下这四个部分：     
- Create projects and modules.
- Work with the project structure.
- Eidt build files to configure the build process.
- Build and run your app. 

关于如何创建`Project`这里就不说了， 默认创建的`Project`中有一个`app`的`Module`。

Add a library module
---

接下来的部分说一下如何在`Project`中创建一个`library module`并且把该`library`变成程序的一个依赖`module`。

### Create a new library module

- 点击`File`菜单后选择`New Module`或在`Project`上右键选`New Module`.     
- 展开页面下方的`More Modules`选择`Android Library`后`Next`.      
- 输入名字这里为了演示方便名字叫做`mylibrary`后一直`Next`即可.      
完成之后打开该`Module`中的`build.gradle`你会看到`apply plugin: 'com.android.library'`说明这是一个`library`.    

### Add a dependency on a library module   
上一步我们创建了`mylibrary module`, 现在我们想让`app module`依赖与`mylibrary module`, 但是构建系统还不知道，我们需要修改`app module`下的`build.gradle`文件添加`mylibrary module`就可以了。 

```xml
...
dependencies {
    ...
    compile project(":mylibrary")
}
```

### Build the project in Android Studio
`Android Studio`中`build project`点击上面导航栏中的`Build`菜单然后选择`Make Project`, 这时窗口底部的状态栏就会显示`build`的进度。       
点击窗口右边底部的![Image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/AndroidStudio_5_2.png?raw=true)图标来显示`Gradle Console`.      
![Image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/AndroidStudio_5_3.png?raw=true)

在窗口右边栏点击`Gradle`窗口可以看到当前所有可用的`build tasks`, 双击里面的`task`即可执行。      
![Image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/AndroidStudio_5_4.png?raw=true)

### Build a release version
点击`Gradle tasks`页面， 展开`app`中的`task`然后双击`assembleRelease`即可。 

Configure the Build
---

接下来以`MyApplication Project`说明以下几个部分：     
- Use the syntax from the Android plugin for Gradle in build files.
- Declare dependencies.
- Configure ProGurad settings. 
- Configure signing settings.
- Work with build variants. 

###Build file basics
`Android Studio projects`中包含一个`build file`，每个`module`中也有一个`build file`名字为`build.gradle`.  
下面是`Project`中`app module`的`build.gradle`文件      
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "21.1.1"

    defaultConfig {
        applicationId "com.charon.myapplication"
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            runProguard false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
	compile project(":mylibrary")
}

```

















		![Image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/AndroidStudio_5_1.png?raw=true)	  

	
	                  
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 