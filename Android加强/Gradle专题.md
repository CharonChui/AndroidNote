Gradle专题
===

随着`Google`对`Eclipse`的无情抛弃以及`Studio`的不断壮大，`Android`开发者逐渐拜倒在`Studio`的石榴裙下。       
而作为`Studio`的默认编译方式，`Gradle`已逐渐普及。我最开始是被它的多渠道打包所吸引。关于多渠道打包，请看之前我写的文章[AndroidStudio使用教程(第七弹)](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%9F%BA%E7%A1%80/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%B8%83%E5%BC%B9).md)

接下来我们就系统的学习一下`Gradle`。    

简介
---

`Gradle`是以`Groovy`语言为基础，面向`Java`应用为主。基于`DSL(Domain Specific Language)`语法的自动化构建工具。

`Gradle`集合了`Ant`的灵活性和强大功能，同时也集合了`Maven`的依赖管理和约定，从而创造了一个更有效的构建方式。凭借`Groovy`的`DSL`和创新打包方式，`Gradle`提供了一个可声明的方式，并在合理默认值的基础上描述所有类型的构建。 `Gradle`目前已被选作许多开源项目的构建系统。

因为`Gradle`是基于`DSL`语法的，如果想看到`build.gradle`文件中全部可以选项的配置，可以看这里
[DSL Reference](http://google.github.io/android-gradle-dsl/current/)

基本的项目设置
---

一个`Gradle`项目通过一个在项目根目录中的`build.gradle`文件来描述它的构建。

###简单的`Build`文件

最简单的`Android`应用中的`build.gradle`都会包含以下几个配置：   
`Project`根目录的`build.gradle`:       
```java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.5.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```
`Module`中的`build.gradle`:       
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"
    ...
}
```


- `buildscript { ... }`配置了编译时的代码驱动. 这种情况下，它声明所使用的是`jCenter`仓库。还有一个声明所依赖的在`Maven`文件的路径。这里声明的包含了`Android`插件所使用的1.5.0版本的`Gradle`. ***注意:***这只会影响`build`中运行的代码，不是项目中。项目中需要声明它自己所需要仓库和依赖关系。
- `apply plugin : com.android.application`，声明使用`com.androdi.application`插件。这是构建`Android`应用所需要的插件。
- `android{...}`配置了所有`Android`构建时的参数。默认情况下，只有编译的目标版本以及编译工具的版本是需要的。

重要: 这里只能使用`com.android.application`插件。如果使用`java`插件将会报错。

###目录结构
`module/src/main`下的目录结构，因为有时候很多人把`so`放到`libs`目录就会报错:    

- java/
- res/
- AndroidManifest.xml
- assets/
- aidl/  
- jniLibs/
- jni/  
- rs/

###配置目录结构
如果项目的结构不标准的时候，可能就需要去配置它。`Android`插件使用了相似的语法，但是因为它有自己的`sourceSets`，所以要在`android`代码块中进行配置。下面就是一个从`Eclipse`的老项目结构中配置主要代码并且将`androidTest`的`sourceSet`设置给`tests`目录的例子:     
```
android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        androidTest.setRoot('tests')
    }
}
```
就像有些人就是要把`so`放到`libs`目录中(这类人有点犟)，那就需要这样进行修改。    
***注意:***因为在旧的项目结构中所有的源文件(`Java`,`AIDL`和`RenderScript`)都放到同一个目录中，我们需要将`sourceSet`中的这些新部件都设置给`src`目录。

Build Tasks
---
对构建文件声明插件时通常或自动创建一些列的构建任务去执行。不管`Java`插件还是`Android`插件都是这样。`Android`常规的任务如下：    

- `assemble`生成项目`output`目录中的内容的任务。
- `check`执行所有的检查的任务。
- `build`执行`assemble`和`check`的任务。
- `clean`清理项目`output`目录的任务。

在`Android`项目中至少会有两种`output`输出:一个`debug apk`和一个`release apk`。他们都有自己的主任务来分别执行构建:    
- `assemble`
    - `assembleDebug`
    - `assembleRelease`  
    
***提示:***`Gradle`支持通过命令行执行任务首字母缩写的方式。例如:      
在没有其他任务符合`aR`的前提下，`gradle aR`与`gradle assembleRelease`是相同的。

最后，构建插件创建了为所有`build type(debug, release, test)`类型安装和卸载的任务，只要他们能被安装(需要签名)。







		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 