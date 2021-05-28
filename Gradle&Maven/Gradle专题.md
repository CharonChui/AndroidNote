Gradle专题
===

随着`Google`对`Eclipse`的无情抛弃以及`Studio`的不断壮大，`Android`开发者逐渐拜倒在`Studio`的石榴裙下。       
而作为`Studio`的默认编译方式，`Gradle`已逐渐普及。我最开始是被它的多渠道打包所吸引。关于多渠道打包，请看之前我写的文章[AndroidStudio使用教程(第七弹)][1]

接下来我们就系统的学习一下`Gradle`。    

简介
---

[Gradle](https://gradle.org/releases/)是以`Groovy`语言为基础，面向`Java`应用为主。基于`DSL(Domain Specific Language)`语法的自动化构建工具。

`Gradle`集合了`Ant`的灵活性和强大功能，同时也集合了`Maven`的依赖管理和约定，从而创造了一个更有效的构建方式。凭借`Groovy`的`DSL`和创新打包方式，`Gradle`提供了一个可声明的方式，并在合理默认值的基础上描述所有类型的构建。 `Gradle`目前已被选作许多开源项目的构建系统。

[Groovy](http://www.groovy-lang.org/api.html)基于Java并拓展了Java。  Java程序员可以无缝切换到使用Groovy开发程序。Groovy说白了就是把写Java程序变得像写脚本一样简单。写完就可以执行，Groovy内部会将其编译成Javaclass然后启动虚拟机来执行。当然，这些底层的渣活不需要你管。*实际上，由于Groovy Code在真正执行的时候已经变成了Java字节码，所以JVM根本不知道自己运行的是Groovy代码*。

因为`Gradle`是基于`DSL`语法的，如果想看到`build.gradle`文件中全部可以选项的配置，可以看这里
[DSL Reference](http://google.github.io/android-gradle-dsl/current/)



![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_build_process.png?raw=true)



基本的项目设置
---

一个`Gradle`项目通过一个在项目根目录中的`build.gradle`文件来描述它的构建。

# Gradle Java 构建入门

## Java 插件

如你所见，Gradle 是一个通用工具。它可以通过脚本构建任何你想要实现的东西，真正实现开箱即用。但前提是你需要在脚本中编写好代码才行。

大部分 Java 项目基本流程都是相似的：编译源文件，进行单元测试，创建 jar 包。使用 Gradle  做这些工作不必为每个工程都编写代码。Gradle 已经提供了完美的插件来解决这些问题。插件就是 Gradle  的扩展，简而言之就是为你添加一些非常有用的默认配置。Gradle 自带了很多插件，并且你也可以很容易的编写和分享自己的插件。Java  plugin 作为其中之一，为你提供了诸如编译，测试，打包等一些功能。

Java  插件为工程定义了许多默认值，如Java源文件位置。如果你遵循这些默认规则，那么你无需在你的脚本文件中书写太多代码。当然，Gradle  也允许你自定义项目中的一些规则，实际上，由于对 Java 工程的构建是基于插件的，那么你也可以完全不用插件自己编写代码来进行构建。

后面的章节我们通过许多深入的例子介绍了如何使用 Java 插件来进行以来管理和多项目构建等。但在这个章节我们需要先了解 Java 插件的基本用法。

### 一个基本 Java 项目

来看一下下面这个小例子，想用 Java 插件，只需增加如下代码到你的脚本里。

### 采用 Java 插件

```
build.gradle
apply plugin: 'java'
```

备注:示例代码可以在 Gralde 发行包中的 samples/java/quickstart 下找到。

定义一个 Java 项目只需如此而已。这将会为你添加 Java 插件及其一些内置任务。

> 添加了哪些任务?
>
> 你可以运行 gradle tasks 列出任务列表。这样便可以看到 Java 插件为你添加了哪些任务。

标准目录结构如下:

```
project  
    +build  
    +src/main/java  
    +src/main/resources  
    +src/test/java  
    +src/test/resources  
```

Gradle 默认会从 `src/main/java` 搜寻打包源码，在 `src/test/java` 下搜寻测试源码。并且 `src/main/resources` 下的所有文件按都会被打包，所有 `src/test/resources` 下的文件 都会被添加到类路径用以执行测试。所有文件都输出到 build 下，打包的文件输出到 build/libs 下。



### 简单的`Build`文件

最简单的`Android`应用中的`build.gradle`都会包含以下几个配置：   
`Project`根目录的`build.gradle`:       

```
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


- `buildscript { ... }`配置了编译时的代码驱动. 这种情况下，它声明所使用的是`jCenter`仓库。还有一个声明所依赖的在`Maven`文件的路径。这里声明的包含了`Android`插件所使用的1.5.0版本的`Gradle`. 注意:这只会影响`build`中运行的代码，不是项目中。项目中需要声明它自己所需要仓库和依赖关系。
- `apply plugin : com.android.application`，声明使用`com.androdi.application`插件。这是构建`Android`应用所需要的插件。
- `android{...}`配置了所有`Android`构建时的参数。默认情况下，只有编译的目标版本以及编译工具的版本是需要的。

重要: 这里只能使用`com.android.application`插件。如果使用`java`插件将会报错。目录结构

`module/src/main`下的目录结构，因为有时候很多人把`so`放到`libs`目录就会报错:    

- java/
- res/
- AndroidManifest.xml
- assets/
- aidl/  
- jniLibs/
- jni/  
- rs/

### 配置目录结构
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
注意:因为在旧的项目结构中所有的源文件(`Java`,`AIDL`和`RenderScript`)都放到同一个目录中，我们需要将`sourceSet`中的这些新部件都设置给`src`目录。



projects 和 tasks是 Gradle 中最重要的两个概念。

任何一个 Gradle 构建都是由一个或多个 projects 组成。每个 project 包括许多可构建组成部分。  这完全取决于你要构建些什么。举个例子，每个 project 或许是一个 jar 包或者一个 web 应用，它也可以是一个由许多其他项目中产生的  jar 构成的 zip 压缩包。一个 project  不必描述它只能进行构建操作。它也可以部署你的应用或搭建你的环境。不要担心它像听上去的那样庞大。 Gradle 的  build-by-convention 可以让您来具体定义一个 project        到底该做什么。

每个 project 都由多个 tasks 组成。每个 task 都代表了构建执行过程中的一个原子性操作。如编译，打包，生成 javadoc，发布到某个仓库等操作。

到目前为止，可以发现我们可以在一个 project 中定义一些简单任务，后续章节将会阐述多项目构建和多项目多任务的内容。

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
    

提示:`Gradle`支持通过命令行执行任务首字母缩写的方式。例如:      
在没有其他任务符合`aR`的前提下，`gradle aR`与`gradle assembleRelease`是相同的。

最后，构建插件创建了为所有`build type(debug, release, test)`类型安装和卸载的任务，只要他们能被安装(需要签名)。

- `installDebug`
- `installRelease`
- `uninstallAll`
    - `uninstallDebug`
    - `uninstallRelease`
    - `uninstallDebugAndroidTest`
    
### 基本的`Build`定制

`Android`插件提供了一些列的`DSL`来让直接从构建系统中做大部分的定制。

##### `Manifest`整体部分
`DSL`提供了很多重要的配置`manifest`文件的参数，例如:     

- `minSdkVersion`
- `targetSdkVersion`
- `versionCode`
- `versionName`
- `applicationId`
- `testApplicationId`
- `testInstrumentationRunnder`

[Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)提供了一个完整的构建参数列表。

把这些`manifest`属性放到`build`文件中的一个重要功能就是它可以被动态的设置。例如，可以通过读取一个文件或者其他逻辑来获取版本名称。     

```
def computeVersionName() {
    ...
}

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"


    defaultConfig {
        versionCode 12 
        versionName computeVersionName()
        minSdkVersion 16
        targetSdkVersion 23
    }
}
```
注意:不要使用可能与现有给定冲突的方法名。例如`defaultConfig{...}`中使用`getVersionName()`方法将会自动使用`defaultConfig.getVersionName()`来带起自定义的方法。

##### `Build Types`

默认情况下`Android`插件会自动将应用程序设置成有一个`debug`版本和一个`release`版本。    
这就是通过调用`BuildType`对象完成。默认情况下会创建两个实例，一个`debug`实例和一个`release`实例。`Android`插件同样允许通过其他的`Build Types`来定制其他的实例。这就是通过`buildTypes`来设置的:     

```
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }


        jnidebug {
            initWith(buildTypes.debug)
            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```
上面的代码执行了以下操作:     
- 配置了默认`debug`的`Build Type`:     
    - 设置了它的`applicationId`。这样`debug`模式就能与`release`模式的`apk`同时安装在同一手机上。
- 创建了一个新的`jnidebug`的`Build Type`，并且把它设置为`debug`的拷贝。
- 通过允许`JNI`组件的`debug`和增加一个新的包名后缀来继续定制该`Build Type`。  

不管使用`initWith()`还是使用其他的代码块，创建一个新的`Build Types`都是非常简单的在`buildTypes`代码块中创建一个新的元素就可以了。

##### 签名配置

为应用签名需要使用如下几个部分:      
- `A keystore`
- `A keystore password`
- `A key alias name`
- `A key password`
- `The store type`


默认情况下有一个`debug`的配置，设置了一个`debug`的`keystore`，有一个已知的密码。`debug keystore`的位置是在`$HOME/.android/debug.keystore`，如果没有的话他会被默认创建。`Debug`的`Build Type`会默认使用该`debug`的签名设置。

当然也可以通过使用`DSL`语法中的`signingconfigs`部分来创建其他的配置来进行定制:    

```
android {
    signingConfigs {
        debug {
            storeFile file("debug.keystore")
        }


        myConfig {
            storeFile file("other.keystore")
            storePassword "android"
            keyAlias "androiddebugkey"
            keyPassword "android"
        }
    }


    buildTypes {
        foo {
            signingConfig signingConfigs.myConfig
        }
    }
}

```
上面的设置将把`debug keystore`的位置改为项目的根目录。同样也创建了一个新的签名配置，并且有一个新的`Build Type`使用它。



### `Dependencies, Android Libraries and Multi-project setup`

`Gradle`项目可以依赖其他的外部二进制包、或者其他的`Gradle`项目。

##### 本地包

想要配置依赖一个外部`jar`包，需要在`compile`的配置中添加一个`dependency`。下面的配置是添加了所有在`libs`目录的`jar`包:   

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}


android {
    ...
}
```
注意:`DSL`元素中的`dependencies`是`Gradle API`中的标准元素。不属于`andorid`元素。      
`compile`配置是用来编译主应用的。它配置的所有部分都会被打包到`apk`中。当然也有一些其他的配置:      

- `compile`: `main application`
- `androidTestCompile`:`test application`
- `debugCompile`:`debug Build Type`
- `release Compile`:`release Build Type`

当然我们可以使用`compile`和`<buildtype>.compile`这两种配置。创建一个新的`Build Type`通常会自动基于它的名字创建一个新的配置部分。这样在像`debug`版本而`release`版本不适用的一些特别的`library`时非常有用。

##### 远程仓库

`Gradle`只是使用`Maven`和`Ivy`仓库。但是仓库必须要添加到列表中，并且必须声明所依赖仓库的`Maven`或者`Ivy`定义。      

```
repositories {
     jcenter()
}


dependencies {
    compile 'com.google.guava:guava:18.0'
}


android {
    ...
}
```

注意:`jcenter()`是指定仓库`URL`的快捷设置。`Gradle`支持远程和本地仓库。
注意:`Gradle`会直接识别所有的依赖关系。这就意味着如果一个依赖库自身又依赖别的库时，他们会被一起下下来。

##### 本地`AAR`库     

```
dependencies {
	compile(name:'本地aar库的名字，不用加后缀', ext:'aar')
}
```



Gradle支持三种不同的仓库，分别是：Maven和Ivy以及文件夹。依赖包会在你执行build构建的时候从这些远程仓库下载，当然Gradle会为你在本地保留缓存，所以一个特定版本的依赖包只需要下载一次。

一个依赖需要定义三个元素：group，name和version。group意味着创建该library的组织名，通常这会是包名，name是该library的唯一标示。version是该library的版本号，我们来看看如何申明依赖：

```
dependencies {
       compile 'com.google.code.gson:gson:2.3'
       compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

上述的代码是基于groovy语法的，所以其完整的表述应该是这样的：

```
dependencies {
      compile group: 'com.google.code.gson', name: 'gson', version:'2.3'
      compile group: 'com.squareup.retrofit', name: 'retrofit'
           version: '1.9.0'
     }
     
```

### 为你的仓库预定义

为了方便，Gradle会默认预定义三个maven仓库：Jcenter和mavenCentral以及本地maven仓库。你可以同时申明它们：

```
repositories {
       mavenCentral()
       jcenter()
       mavenLocal()
   }
   
```

Maven和Jcenter仓库是很出名的两大仓库。我们没必要同时使用他们，在这里我建议你们使用jcenter，jcenter是maven中心库的一个分支，这样你可以任意去切换这两个仓库。当然jcenter也支持了https，而maven仓库并没有。

本地maven库是你曾使用过的所有依赖包的集合，当然你也可以添加自己的依赖包。默认情况下，你可以在你的home文件下找到.m2的文件夹。除了这些仓库外，你还可以使用其他的公有的甚至是私有仓库。



### 远程仓库

有些组织，创建了一些有意思的插件或者library,他们更愿意把这些放在自己的maven库，而不是maven中心库或jcenter。那么当你需要是要这些仓库的时候，你只需要在maven方法中加入url地址就好：

```
repositories {
       maven {
           url "http://repo.acmecorp.com/maven2"
       }
}
```

同样的，Ivy仓库也可以这么做。Apache Ivy在ant世界里是一个很出名的依赖管理工具。如果你的公司有自己的仓库，如果他们需要权限才能访问，你可以这么编写：

```
repositories {
       maven {
           url "http://repo.acmecorp.com/maven2"
           credentials {
               username 'user'
               password 'secretpassword'
           }
        } 
   }
```

> 注意：这不是一个好主意，最好的方式是把这些验证放在Gradle properties文件里，这些我们已经介绍过在第二章。

## 本地依赖

可能有些情况，你需要手动下载jar包，或者你想创建自己的library，这样你就可以复用在不同的项目，而不必将该library  publish到公有或者私有库。在上述情况下，可能你不需要网络资源，接下来我将介绍如何是使用这些jar依赖，以及如何导入so包，如何为你的项目添加依赖项目。

### 文件依赖

如果你想为你的工程添加jar文件作为依赖，你可以这样：

```
dependencies {
       compile files('libs/domoarigato.jar')
}
```

如果你这么做，那会很愚蠢，因为当你有很多这样的jar包时，你可以改写为：

```
dependencies {
       compile fileTree('libs')
 }
 
```

默认情况下，新建的Android项目会有一个lib文件夹，并且会在依赖中这么定义（即添加所有在libs文件夹中的jar）：

```
dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

这也意味着，在任何一个Android项目中，你都可以把一个jar文件放在到libs文件夹下，其会自动的将其添加到编译路径以及最后的APK文件。

### native包（so包）

用c或者c++写的library会被叫做so包，Android插件默认情况下支持native包，你需要把.so文件放在对应的文件夹中：

```
app
   ├── AndroidManifest.xml
   └── jniLibs
       ├── armeabi
       │   └── nativelib.so
       ├── armeabi-v7a
       │   └── nativelib.so
       ├── mips
       │   └── nativelib.so
       └── x86
           └── nativelib.so
           
```

## aar文件

如果你想分享一个library,该依赖包使用了Android api，或者包含了Android 资源文件，那么aar文件适合你。依赖库和应用工程是一样的，你可以使用相同的tasks来构建和[测试](http://lib.csdn.net/base/softwaretest)你的依赖工程，当然他们也可以有不同的构建版本。应用工程和依赖工程的区别在于输出文件，应用工程会生成APK文件，并且其可以安装在Android设备上，而依赖工程会生成.aar文件。该文件可以被Android应用工程当做依赖来使用。

### 创建和使用依赖工程模块

不同的是，你需要加不同的插件：

```
 apply plugin: 'com.android.library'
 
```

我们有两种方式去使用一个依赖工程。一个就是在你的工程里面，直接将其作为一个模块，另外一个就是创建一个aar文件，这样其他的应用也就可以复用了。

如果你把其作为模块，那你需要在settings.gradle文件中添加其为模块：

```
   include ':app', ':library'
   
```

在这里，我们就把它叫做library吧，如果你想使用该模块，你需要在你的依赖里面添加它，就像这样:

```
   dependencies {
       compile project(':library')
  }
```

### 使用aar文件

如果你想复用你的library，那么你就可以创建一个aar文件，并将其作为你的工程依赖。当你构建你的library项目，aar文件将会在  build/output/aar/下生成。把该文件作为你的依赖包，你需要创建一个文件夹来放置它，我们就叫它aars文件夹吧，然后把它拷贝到该文件夹里面，然后添加该文件夹作为依赖库：

```
repositories {
    flatDir {
        dirs 'aars' 
    }
}
```

这样你就可以把该文件夹下的所有aar文件作为依赖，同时你可以这么干：

```
 dependencies {
       compile(name:'libraryname', ext:'aar')
}
```

这个会告诉Gradle，在aars文件夹下，添加一个叫做libraryname的文件，且其后缀是aar的作为依赖。







##### 多项目设置

`Gradle`项目通常使用多项目设置来依赖其他的`gradle`项目。例如:      

- MyProject/         
    - app/          
    - libraries/        
        - lib1/           
        - lib2/     
        

`Gradle`会通过下面的名字来引用他们：     
`:app`       
`:libraries:lib1`          
`:libraries:lib2`      


每个项目都会有一个单独的`build`文件，并且在项目的根目录还会有一个`setting.gradle`文件：    
      
- MyProject/         
    - settings.gradle         
    - app/      
        - build.gradle       
    + libraries/      
        + lib1/       
            - build.gradle      
        + lib2/      
           - build.gradle      
    

`setting.gradle`文件中的内容非常简单。它指定了哪个目录是`Gralde`项目:     
```
include ':app', ':libraries:lib1', ':libraries:lib2'
```

`：app`这个项目可能会依赖其他的`libraries`，这样可以通过如下进行声明:    
```
dependencies {
     compile project(':libraries:lib1')
}
```

### `Library`项目

上面用到了`:libraries:lib1`和`:libraries:lib2`可以是`Java`项目，`:app`项目会使用他们俩的输出的`jar`包。但是如果你需要使用`android`资源等，这些`libraries`就不能是普通的`Java`项目了，他们必须是`Android Library`项目。    

##### 创建一个`Library`项目      

`Library`项目和普通的`Android`项目的区别比较少，由于`libraries`的构建类型与应用程序的构建不同，所有它会使用一个别的构建插件。但是他们所使用的插件内部有很多相同的代码，他们都是由`com.android.tools.build.gradle`这个`jar`包提供的。

```
buildscript {
    repositories {
        jcenter()
    }


    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.1'
    }
}


apply plugin: 'com.android.library'


android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"
}
```

##### 普通项目与`Library`项目的区别       

`Library`项目的主要输出我`.aar`包。它结合了代码(例如`jar`包或者本地`.so`文件)和资源(`manifest`,`res`,`assets`)。每个`library`也可以单独设置`Build Type`等来指定生成不同版本的`aar`。

### `Lint Support`

你可以通过指定对应的变量来设置`lint`的运行。可以通过添加`lintOptions`来进行配置:      

```
android {
    lintOptions {
        // turn off checking the given issue id's
        disable 'TypographyFractions','TypographyQuotes'

        // turn on the given issue id's
        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'

        // check *only* the given issue id's
        check 'NewApi', 'InlinedApi'
    }
}
```

### `Build`变量


构建系统的一个目标就是能对同一个应用创建多个不同的版本。    

##### `Product flavors`    

一个`product flavor`可以针对一个项目制定不同的构建版本。一个应用可以有多个不同的`falvors`来改变生成的应用。   
`Product flavors`是通过`DSL`语法中的`productFlavors`来声明的:    

```
android {
    ....


    productFlavors {
        flavor1 {
            ...
        }


        flavor2 {
            ...
        }
    }
}
```

##### `Build Type + Product Flavor = Build Variant`

像我们之前看到的，每个`Build Type`都会生成一个`apk`.`Product Flavors`也是同样的：项目的输出僵尸所有`Build Types`与`Product Flavors`的结合。每种结合方式称之为`Build Variant`。例如，如果有`debug`和`release`版本的`Build Types`，上面的例子就会生成4种`Build Variants`：    
- `Flavor1` - `debug`
- `Flavor1` - `release`
- `Flavor2` - `debug`
- `Flavor2` - `release`

没有配置`flavors`的项目仍然有`Build Variants`，它只是用了一个默认的`flavor/config`，没有名字，这导致`variants`的列表和`Build Types`的列表比较相同。

##### `Product Flavor`配置

```
android {
    ...


    defaultConfig {
        minSdkVersion 8
        versionCode 10
    }


    productFlavors {
        flavor1 {
            applicationId "com.example.flavor1"
            versionCode 20
         }


         flavor2 {
             applicationId "com.example.flavor2"
             minSdkVersion 14
         }
    }
}
```
注意`android.productFlavors.*`对象`ProductFlavor`有`android.defaultConfig`是相同的类型。这就意味着他们有相同的属性。    
`defaultConfig`为所有的`flavors`提供了一些基本的配置，每个`flavor`都已重写他们。在上面的例子中，这些配置有:       

- `flavor1`        
    - `applicationId`: `com.example.flavor1`      
    - `minSdkVersion`: 8       
    - `versionCode`: 20
- `flavor2`    
    - `applicationId`: `com.example.flavor2`      
    - `minSdkVersion`: 14       
    - `versionCode`: 10       
    

通常，`Build Type`配置会覆盖其他的配置。例如，`Build Type`的`applicationIdSuffix`会添加到`Product Flavor`的`applicationId`上。


最后，就像`Build Types`一样，`Product Flavors`也可以有他们自己的依赖关系。例如，如果有一个单独的`flavors`会使用一些广告或者支付，那这个`flavors`生成的`apk`就会使用广告的依赖，而其他的`flavors`就不需要使用。    
```
dependencies {
    flavor1Compile "..."
}
```

### `BuildConfig`      

在编译阶段，`Android Studio`会生成一个叫做`BuildConfig`的类，该类包含了编译时使用的一些变量的值。你可以观看这些值来改变不同变量的行为:       

```
private void javaCode() {
    if (BuildConfig.FLAVOR.equals("paidapp")) {
        doIt();
    else {
        showOnlyInPaidAppDialog();
    }
}
```

下面是`BuildConfig`中包含的一些值:      

- `boolean DEBUG` - `if the build is debuggable`      
- `int VERSION_CODE` 
- `String VERSION_NAME`
- `String APPLICATION_ID`
- `String BUILD_TYPE`- `Build Type`的名字，例如`release`      
- `String FLAVOR` - `flavor`的名字，例如`flavor1`      


### `ProGuard`配置      

`Android`插件默认会使用`ProGuard`插件，并且如果`Build Type`中使用`ProGuard`的`minifyEnabled`属性开启的话，会默认创建对应的`task`。   

```
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }

    productFlavors {
        flavor1 {
        }
        flavor2 {
            proguardFile 'some-other-rules.txt'
        }
    }
}
```

### `Tasks`控制     

基本的`Java`项目有一系列的`tasks`一起制作输出文件。     
`classes task`就是编译`Java`源码的任务。 我们可以在`build.gradle`中通过使用`classes`很简单的获取到它。就是`project.tasks.classes`.    

在`Android`项目中，更多的编译`task`，因为他们的名字通过`Build Types`和`Product Flavors`生成。

为了解决这个问题，`android`对象有两种属性:     
- `applicationVariants` - `only for the app plugin`     
- `libraryVariants` - `only for the library plugin`      
- `testVariants` - `for both plugins`      
这些都会返回一个`ApplicationVariant`, `LibraryVariant`,`TestVariant`的`DomainObjectCollection`接口的实现类对象。   
`DomainObjectCollection`提供了直接获取或者很方便的间接获取所有对象的方法。     
```
android.applicationVariants.all { variant ->
   ....
}
```

### 设置编译语言版本

可以使用`compileOptions`代码块来设置编译时使用的语言版本。默认是基于`compileSdkVersion`的值。
```
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_6
        targetCompatibility JavaVersion.VERSION_1_6
    }
}
```

### `Resource Shrinking`    

`Gradle`构建系统支持资源清理：对构建的应用会自动移除无用的资源。不仅会移除项目中未使用的资源，而且还会移除项目所以来的类库中的资源。注意，资源清理只能在与代码清理结合使用(例如`ProGuad`)。这就是为什么它能移除所依赖类库的无用资源。通常，类库中的所有资源都是使用的，只有类库中无用代码被移除后这些资源才会变成没有代码引用的无用资源。    

```
android {
    ...

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```



## Task

在Gradle中有一个原子性的操作叫做task，简单理解为task是Gradle脚本中最小可执行单元。

在build.gradle里可以通过task关键字来创建Task。

例如，可以在build.gradle中创建2个task:   

```groovy
task helloWorld {
    println "Hello World"
}
task myTask2 {
    println "configure task2"
}
```

在命令行里执行命令: gradle helloWorld： 

```
Hello World
configure task2
```

我们会发现当我们执行helloWorld时，task2的代码也被执行了。括号内部的代码我们称之为配置代码，在gradle脚本的配置阶段都会执行，也就是说不管执行脚本里的哪个任务，所有task里的配置代码都会执行。

这与我们期望的不一致，通常我们写程序时调用一个方法，这个方法里的代码才会执行，那么我们执行一个task时，这个task里的代码才会被执行才对。显然Gradle里的不一样，这个问题就设计到Task Action的概念。 



## Task Action

一个Task由一系列Action组成，当运行一个Task的时候，这个Task里的Action序列会按顺序依次执行。

前面例子中括号里的代码只是配置代码，它们并不是Action，Task里的Action只会在该Task真正运行时执行，Gradle里通过doFirst、doLast来为Task增加Action。

- doFirst: task执行时最先执行的操作
- doLast: task执行时最后执行的操作

```
task myTask1 {
    println "configure task1"
}
task myTask2 {
    println "configure task2"
}
myTask1.doFirst {
    println "task1 doFirst"
}
myTask1.doLast {
    println "task1 doLast"
}
myTask2.doLast {
    println "task2 doLast"
}
```

同样运行gradle myTask1，来执行myTask1，这次的结果如下:  

```
configure task1
configure task2

task1 doFirst
task1 doLast
```

可以看到所有Task的配置代码都会运行，而Task Action则只有该Task运行时才会执行。 

doLast有一种等价操作叫做leftShift，leftShift可以缩写为<<，下面几种写法效果是一模一样的:   

```
myTask1.doLast {
    println "task1 doLast"
}
myTask1 << {
    println "task1 doLast <<"
}
myTask1.leftShift {
    println "task1 doLast leftShift"
}
```

<<操作符只是一种Gradle里的语法糖

## Extension

先来看一段Android应用的Gradle配置代码:  

```groovy
android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "xxx"
        minSdkVersion 19
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

上面这个android打包配置就是Gradle的Extension，翻译成中文的意思就是扩展。它的作用就是通过实现自定义的Extension，可以在Gradle脚本中增加类似android这样命名空间的配置，Gradle可以识别这种配置，并读取里面的配置内容。 

每个Extension实际上都会与某个类相关联，在build.gradle中通过DSL来定义，Gradle会识别解析并生成一个对象实例，通过该类可以获取我们所配置的信息。

```groovy
outer {
    outerName "outer"
    msg "this is a outer message."

    inner {
        innerName "inner"
        msg "This is a inner message."
    }
}
```

形式上就是外面的Extension里面定义了另一个Extension，这种叫做nested Extension，也就是嵌套的 Extension。

## Project详解

每一个build.gradle脚本文件被Gradle加载解析后，都会生成一个对应的Project对象，在脚本中的配置方法其实都对应着Project中的API，如果想详细了解这些脚本的配置含义，有必要对Project类进行深入的了解。 



### Project类图

当构建进程启动后，Gradle基于build.gradle中的配置实例化org.gradle.api.Project类:  

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/build_gradle_project.png?raw=true)

#### 构建脚本配置

##### buildscript

配置该Project的构建脚本的classpath，在Android Studio中的root project中可以看到:  

```groovy
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
}
```



##### apply

```groovy
apply(options: Map<String, ?>)
```

我们通过该方法使用插件或者是其他脚本，options里主要选项有：

- from： 使用其他脚本，值可以为Project.uri(Object) 支持的路径
- plugin：使用其他插件，值可以为插件id或者是插件的具体实现类

例如：

```groovy
//使用插件，com.android.application 就是插件id
apply plugin: 'com.android.application'
//使用插件，MyPluginImpl 就是一个Plugin接口的实现类
apply plugin: MyPluginImpl

//引用其他gradle脚本，push.gradle就是另外一个gradle脚本文件
apply from: './push.gradle'
```

### Gradle属性



在与build.gradle文件同级目录下，定义一个名为gradle.properties的文件，里面定义的键值对，可以在Project中直接访问:  

```groovy
// gradle.properties
username="xxx"
password="yyy"
```

在build.gradle文件里可以直接访问:  

```groovy
println "username = ${username}"
println "password = ${password}"
```

### 扩展属性



在一个build.gradle里，可以通过变量定义来实现相关字符串的替换，比如：

    apply plugin: 'com.android.application'
     
    def mCompileSdkVersion = 28
    def libAndroidAppcompat = 'com.android.support:appcompat-v7:28.0.0'
    android {
        compileSdkVersion mCompileSdkVersion
    }
     
    dependencies {
        implementation libAndroidAppcompat
    }

gradle支持扩展属性，通过扩展属性也可以达到上述的目的：

    apply plugin: 'com.android.application'
     
    // 扩展属性
    ext {
        compileSdkVersion = 28
        libAndroidAppcompat = 'com.android.support:appcompat-v7:28.0.0'
    }
     
    android {
        compileSdkVersion this.compileSdkVersion
    }
     
    dependencies {
        implementation this.libAndroidAppcompat
    }

为每个子工程配置扩展属性，在根build.gradle中添加如下代码，子工程相关配置删除。

    subprojects {
        ext {
            compileSdkVersion = 28
            libAndroidAppcompat = 'com.android.support:appcompat-v7:28.0.0'
        }
    }

上述方法相当于每个子project定义了扩展属性，如果想定义一份，需要把根build.gradle改成如下：

    ext {
        compileSdkVersion = 28
        libAndroidAppcompat = 'com.android.support:appcompat-v7:28.0.0'
    }

子工程build.gradle改成如下：

    apply plugin: 'com.android.application'
     
    android {
        compileSdkVersion this.rootProject.compileSdkVersion
    }
     
    dependencies {
        implementation this.rootProject.libAndroidAppcompat
    }

如果把rootProject去掉，也是可以的

    apply plugin: 'com.android.application'
     
    android {
        compileSdkVersion this.compileSdkVersion
    }
     
    dependencies {
        implementation this.libAndroidAppcompat
    }

gradle规定，父project所有的属性都会被根project继承，所以可以直接在子project使用父project的属性。
可以把所有的扩展属性定义到一个独立的gradle文件中，在需要使用的build.gradle文件中使用apply from进行引入。

```
apply from : file('common.gradle')
```

修改根目录build.gradle文件如下:  

```groovy
println "-----root file config-----"

//配置 app 项目
project(":app") {
    ext {
        appParam = "test app"
    }
}

//配置所有的项目
allprojects {
    ext {
        allParam = "test all project"
    }   
}

//配置子项目
subprojects {
    ext {
        subParam = "test sub project"
    }
}

println "allParam = ${allParam}"
```





还可以通过ext命名空间来定义属性，我们称之为扩展属性。 

```groovy
ext {
  username = "hjy"
  age = 30
}

println username
println ext.age
println project.username
println project.ext.age
```

必须注意，默认的扩展属性，只能定义在 ext 命名空间下面。对扩展属性的访问方式，以上几种都支持。



如果你有新建一个kotlin项目的经历，那么你将看到Google推荐的方案

```
buildscript {
    ext.kotlin_version = '1.1.51'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```

在rootProject的build.gradle中使用**ext**来定义版本号全局变量。这样我们就可以在module的build.gradle中直接引用这些定义的变量。引用方式如下：

```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
}
```

你可以将这些变量理解为java的静态变量。通过这种方式能够达到不同module中的配置统一，但局限性是，一但配置项过多，所有的配置都将写到rootProject项目的build.gradle中，导致build.gradle臃肿。这不符合我们的所提倡的模块开发，所以应该想办法将ext的配置单独分离出来。

这个时候我就要用到之前的文章[Android Gradle系列-原理篇](https://mp.weixin.qq.com/s?__biz=MzIzNTc5NDY4Nw==&mid=2247483834&idx=1&sn=55264aaad1f018b55280beec93ed4cac&chksm=e8e0f82adf97713c5a43c67b67fbabd659578328a22a406c5a01bd69ccf550e88bf645b15457&token=330677494&lang=zh_CN#rd)中所介绍的apply函数。之前的文章我们只使用了apply三种情况之一的plugin(应用一个插件，通过id或者class名)，只使用在子项目的build.gradle中。

```
apply plugin: 'com.android.application'
```

这次我们需要使用它的**from**，它主要是的作用是**应用一个脚本文件**。作用接下来我们需要做的是将ext配置单独放到一个gradle脚本文件中。

首先我们在rootProject目录下创建一个gradle脚本文件，我这里取名为version.gradle。

然后我们在version.gradle文件中使用ext来定义变量。例如之前的kotlin版本号就可以使用如下方式实现

```
ext.deps = [:]
 
def versions = [:]
versions.support = "26.1.0"
versions.kotlin = "1.2.51"
versions.gradle = '3.2.1'
 
def support = [:]
support.app_compat = "com.android.support:appcompat-v7:$versions.support"
support.recyclerview = "com.android.support:recyclerview-v7:$versions.support"
deps.support = support
 
def kotlin = [:]
kotlin.kotlin_stdlib = "org.jetbrains.kotlin:kotlin-stdlib-jre7:$versions.kotlin"
kotlin.plugin = "org.jetbrains.kotlin:kotlin-gradle-plugin:$versions.kotlin"
deps.kotlin = kotlin
 
deps.gradle_plugin = "com.android.tools.build:gradle:$versions.gradle"
 
ext.deps = deps
 
def build_versions = [:]
build_versions.target_sdk = 26
build_versions.min_sdk = 16
build_versions.build_tools = "28.0.3"
ext.build_versions = build_versions
 
def addRepos(RepositoryHandler handler) {
    handler.google()
    handler.jcenter()
    handler.maven { url 'https://oss.sonatype.org/content/repositories/snapshots' }
}
ext.addRepos = this.&addRepos
```

> 因为gradle使用的是groovy语言，所以以上都是groovy语法

例如kotlin版本控制，上面代码的意思就是将有个kotlin相关的版本依赖放到deps的kotlin变量中，同时deps放到了ext中。其它的亦是如此。

既然定义好了，现在我们开始引入到项目中，为了让所有的子项目都能够访问到，我们使用**apply from**将其引入到rootProject的build.gradle中

```
buildscript {
    apply from: 'versions.gradle'
    addRepos(repositories)
    dependencies {
        classpath deps.gradle_plugin
        classpath deps.kotlin.plugin
    }
}
```

这时build.gradle中就默认有了ext所声明的变量，使用方式就如dependencies中的引用一样。

我们再看上面的addRepos方法，在关于Gradle原理的文章中已经分析了repositories会通过RepositoryHandler来执行，所以这里我们直接定义一个方法来统一调用RepositoryHandler。这样我们在build.gradle中就无需使用如下方式，直接调用addRepos方法即可

```
    //之前调用
    repositories {
        google()
        jcenter()
    }
    //现在调用
    addRepos(repositories)
```

另一方面，如果有多个module，例如有module1，现在就可以直接在module1中的build.gradle中使用定义好的配置

```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    // support
    implementation deps.support.app_compat
    //kotlin
    implementation deps.kotlin.kotlin_stdlib
}
```

上面我们还定义了sdk与tools版本，所以也可以一起统一使用，效果如下

```
android {
    compileSdkVersion build_versions.target_sdk
    buildToolsVersion build_versions.build_tools
    defaultConfig {
        applicationId "com.idisfkj.androidapianalysis"
        minSdkVersion build_versions.min_sdk
        targetSdkVersion build_versions.target_sdk
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    ...
}
```

一旦实现了统一配置，那么之后我们要修改相关的版本就只需在我们定义的version.gradle中修改即可。无需再对所用的module进行逐一修改与统一配置。



扩展属性也可以定义在gradle.properties中，在这个文件中只能定义key-value形式的扩展属性，而不能使用类似Map方式的定义，在使用上有一定的限制。
下面通过在gradle.properties定义开关控制一个模块是否引入项目中，在gradle.properties中定义：

isLoadTest = false

setting.gradle：

    include ':app', ':module1', ':module2'
    if(hasProperty('isLoadTest') ? isLoadTest.toBoolean() : false) {
        include 'test'
    }

所以gradle.properties也可以定义扩展属性，在使用的时候转换成对应的类型。
例如在gradle.properties定义：

mCompileSdkVersion = 28

使用的时候：

compileSdkVersion mCompileSdkVersion.toInteger()

在gradle.properties定义的属性不能和build.gradle已经存在的方法同名，否则编译的时候不报错，但是在使用时会提示属性找不到。



### 多模块构建的结构

通常情况下，一个工程包含多模块，这些模块会在一个父目录文件夹下。为了告诉gradle，该项目的结构以及哪一个子文件夹包含模块，你需要提供一个settings.gradle文件。每个模块可以提供其独立的build.gradle文件。我们已经学习了关于setting.gradle和build.gradle如何正常工作，现在我们只需要学习如何使用它们。

这是多模块项目的结构图：

```
 project
   ├─── setting.gradle
   ├─── build.gradle
   ├─── app
   │    └─── build.gradle
   └─── library
        └─── build.gradle
        
```

这是最简单最直接的方式来创建你的多模块项目了。setting.gradle文件申明了该项目下的所有模块，它应该是这样：

```
include ':app', ':library'
```

这保证了app和library模块都会包含在构建配置中。你需要做的仅仅只是为你的模块添加子文件夹。

为了在你的app模块中添加library模块做为其依赖包，你需要在app的build.gradle文件中添加以下内容：

```
dependencies {
      compile project(':library') 
}
```

为了给app添加一个模块作为依赖，你需要使用project()方法，该方法的参数为模块路径。

如果在你的模块中还包含了子模块，gradle可以满足你得要求。举个栗子，你可以把你的目录结构定义为这样：

```
project
├─── setting.gradle
├─── build.grade
├─── app
│    └─── build.gradle
└─── libraries
     ├─── library1
     │    └─── build.gradle
     └─── library2
          └─── build.gradle
          
```

该app模块依然位于根目录，但是现在项目有2个不同的依赖包。这些依赖模块不位于项目的根目录，而是在特定的依赖文件夹内。根据这一结构，你需要在settings.xml中这么定义：

```
include ':app', ':libraries:library1', ':libraries:library2'
```

你会注意到在子目录下申明模块也非常的容易。所有的路径都是围绕着根目录，即当你添加一个位于子文件夹下的模块作为另外一个模块的依赖包得实惠，你应该将路径定为根目录。这意味着如果在上例中app模块想要依赖library1,build.gradle文件需要这么申明：

```
dependencies {
       compile project(':libraries:library1')
}
```

如果你在子目录下申明了依赖，所有的路径都应该与根目录相关。这是因为gradle是根据你的项目的根目录来定义你的依赖包的。



## 构建的三个构建阶段

1. Initialization：配置构建环境以及有哪些Project会参与构建（解析settings.build）
2. Configuration：生成参与构建的Task的有向无环图以及执行属于配置阶段的代码（解析build.gradle）
3. Execution：按序执行所有Task

在第一步骤中，即初始化阶段，gradle会寻找到settings.grade文件。如果该文件不存在，那么gradle就会假定你只有一个单独的构建模块。如果你有多个模块，settings.gradle文件定义了这些模块的位置。如果这些子目录包含了其自己的build.gradle文件，gradle将会运行它们，并且将他们合并到构建任务中。这就解释了为什么你需要申明在一个模块中申明的依赖是相对于根目录。

一旦你理解了构建任务是如何将所有的模块聚合在一起的时候，那关于几种不同的构建多模块策略就会变得简单易懂。你可以配置所有的模块在根目录下的build.gradle。这让你能够简单的浏览到整个项目的配置，但是这将会变得一团乱麻，特别是当你的模块需要不同的插件的时候。另外一种方式是将每个模块的配置分隔开，这一策略保证了每个模块之间的互不干扰。这也让你跟踪构建的改变变得容易，因为你不需要指出哪个改变导致了哪个模块出现错误等。



#### 为你的项目添加模块



#### 添加Java依赖库

当你新建了一个Java模块，build.grade文件会是这样：

```
apply plugin: 'java'
   dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])
}
    
```

Java模块使用了Java插件，这意味着很多Android特性在这儿不能使用，因为你不需要。

build文件也有基本的库管理，你可以添加jar文件在libs文件夹下。你可以添加更多的依赖库，根据第三章的内容。

给你的app模块添加Java模块，这很简单，不是吗？

```
dependencies {
       compile project(':javalib')
}
```

这告诉了gradle去引入一个叫做javelin的模块吧，如果你为你的app模块添加了这个依赖，那么javalib模块将会总是在你的app模块构建之前构建。

#### 添加Android依赖库

同样的，我们利用Android studio的图形化界面创建Android模块，然后其构建文件如下：

```
apply plugin: 'com.android.library'
```

记住：Android依赖库不仅仅包含了Java代码，同样也会包含Android资源，像manifest和strings,layout文件，在你引入该模块后，你可以使用该模块的所有类和资源文件。





## Groovy

在Java中，打印一天String应该是这样的：

```
System.out.println("Hello, world!");
```

在Groovy中，你可以这么写：

```
println 'Hello, world!'
```

你应该主要到几点不同之处：

- 没有了System.out
- 没有了方括号
- 列结尾没有了；

这个例子同样使用了单引号，你可以使用双引号或者单引号，但是他们有不同的用法。双引号可以包含插入语句。插入是计算一个字符串包含placeholders的过程，并将placeholders的值替换，这些placeholder可以是变量甚至是方法。Placeholders必须包含一个方法或者变量，并且其被{}包围，且其前面有$修饰。如果其只有一个单一的变量，可以只需要$。下面是一些基本的用法：

```
def name = 'Andy'
def greeting = "Hello, $name!"
def name_size "Your name is ${name.size()} characters long."
```

greeting应该是“ Hello，Andy”，并且 name_size 为 Your name is 4 characters long.string的插入可以让你更好的动态执行代码。比如

```
 def method = 'toString'
 new Date()."$method"()
```

这在Java中看起来很奇怪，但是这在groovy里是合法的。



Groovy里面创建类和Java类似，举个例子：

```
class MyGroovyClass {
       String greeting
       String getGreeting() {
           return 'Hello!'
        } 
}
```

注意到不论是类名还是成员变量都没有修饰符。其默认的修饰符是类和方法为public，成员变量为private。

当你想使用MyGroovyClass，你可以这样实例化：

```
def instance = new MyGroovyClass()
instance.setGreeting 'Hello, Groovy!'
instance.getGreeting()   
```

你可以利用def去创建变量，一旦你为你的类创建了实例，你就可以操作其成员变量了。get/set方法groovy默认为你添加 。你甚至可以覆写它。

如果你想直接使用一个成员变量，你可以这么干：

```
 println instance.getGreeting()
 println instance.greeting
 
```

而这二种方式都是可行的。

### 方法

和变量一样，你不必定义为你的方法定义返回类型。举个例子，先看java：

```
public int square(int num) {
       return num * num;
} 
square(2);
```

你需要将该方法定义为public，需要定义返回类型，以及入参，最后你需要返回值。

我们再看下Groovy的写法：

```
 def square(def num) {
       num * num
 }
 square 4
 
```

没有了返回类型，没有了入参的定义。def代替了修饰符，方法体内没有了return关键字。然而我还是建议你使用return关键字。当你调用该方法时，你不需要括号和分号。

我们设置可以写的更简单点：

```
def square = { num ->
       num * num
}
square 8
```

下面我将通过code的形式，列出几点

- 当调用的方法有参数时，可以不用()，看下面的例子





```
 1def printAge(String name, int age) {
 2    print("$name is $age years old")
 3}
 4
 5def printEmptyLine() {
 6    println()
 7}
 8
 9def callClosure(Closure closure) {
10    closure()
11}
12
13printAge "John", 24 //输出John is 24 years old
14printEmptyLine() //输出空行
15callClosure { println("From closure") } //输出From closure
```



- 如果最后的参数是闭包，可以将它写在括号的外面



```
1def callWithParam(String param, Closure<String> closure) {
2    closure(param)
3}
4
5callWithParam("param", { println it }) //输出param
6callWithParam("param") { println it } //输出param
7callWithParam "param", { println it } //输出param
```



- 调用方法时可以指定参数名进行传参，有指定的会转化到Map对象中，没有的将按正常传参



```
 1def printPersonInfo(Map<String, Object> person) {
 2    println("${person.name} is ${person.age} years old")
 3}
 4
 5def printJobInfo(Map<String, Object> job, String employeeName) {
 6    println("${employeeName} works as ${job.name} at ${job.company}")
 7}
 8
 9printPersonInfo name: "Jake", age: 29
10printJobInfo "Payne", name: "Android Engineer", company: "Google"
```

你会发现他们的调用都不需要括号，同时printJobInfo的调用参数的顺序不受影响。





### 闭包

闭包是一段匿名的方法体，其可以接受参数和返回值。它们可以定义变量或者可以将参数传给方法。

你可以简单的使用方括号来定义闭包，如果你想详细点，你也可以这么定义：

```
Closure square = {
       it * it
}
square 16
```

添加了Closure，让其更加清晰。注意，当你没有显式的为闭包添加一个参数，groovy会默认为你添加一个叫做it。你可以在所有的闭包中使用it，如果调用者没有定义任何参数，那么it将会是null，这会使得你的代码更加简洁。

在grade中，我们经常使用闭包，例如Android代码体和dependencies也是。

### Collections

在groovy中，有二个重要的容器分别是lists和maps。

创建一个list很容易，我们不必初始化：

```
List list = [1, 2, 3, 4, 5]
```

为list迭代也很简单，你可以使用each方法：

```
list.each() { element ->
       println element
}
```

你甚至可以使得你的代码更加简洁，使用it:

```
list.each() {
       println it
}
```

map和list差不多：

```
Map pizzaPrices = [margherita:10, pepperoni:12]
```

如果你想取出map中的元素，可以使用get方法：

```
pizzaPrices.get('pepperoni')
pizzaPrices['pepperoni']
```

同样的groovy有更简单的方式：

```
pizzaPrices.pepperoni
```



[1]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%B8%83%E5%BC%B9).md       "AndroidStudio使用教程(第七弹)"












---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 