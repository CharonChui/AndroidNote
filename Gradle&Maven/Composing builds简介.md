Composing builds简介
===

在Android Studio项目中，经常会引用多个Module，而且会有多人同时参与项目开发，在这种背景下，会时常遇到版本冲突问题，出现不同的compileSdkVersion或者出现同一个库的多个不同版本等，导致我们的包体变大，项目运行时间变长，所以将依赖版本统一是一个项目优化的必经之路。

到目前为止Google为管理Gradle依赖提供了4种不同方法:  

- 手动管理

    在每个module中定义插件依赖库，每次升级依赖库时都需要手动更改（不建议使用）。

- ext的方式管理

    这是Google推荐管理依赖的方法 [Android官方文档](https://developer.android.com/studio/build/gradle-tips#configure-project-wide-properties)。但是无法跟踪依赖关系，可读性差，不便维护。

- Kotlin + buildSrc

    支持自动补全和单击跳转，依赖更新时将重新构建整个项目。

- Composing builds

    自动补全和单击跳转，依赖更新时不会重新构建整个项目。

    

## Groovy ext扩展函数的替代方式

我们在使用Groovy语言构建项目的时候，抽取config.gradle作为全局的变量控制，使用ext扩展函数来统一配置依赖，如下：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/config_gradle.png?raw=true)

```groovy
ext {
    android = [
            compileSdkVersion: 29,
            buildToolsVersion: "29",
            minSdkVersion    : 17,
            targetSdkVersion : 26,
            versionCode      : 102,
            versionName      : "1.0.2"
    ]

    version = [
            appcompatVersion       : "1.1.0",
            coreKtxVersion         : "1.2.0",
            supportLibraryVersion  : "28.0.0",
            androidTestVersion     : "3.0.1",
            junitVersion           : "4.12",
            glideVersion           : "4.11.0",
            okhttpVersion          : "3.11.0",
            retrofitVersion        : "2.3.0",
            constraintLayoutVersion: "1.1.3",
            gsonVersion            : "2.7",
            rxjavaVersion          : "2.2.2",
            rxandroidVersion       : "2.1.0",
            ..........省略...........
    ]

    dependencies = [
            //base
            "constraintLayout"      : "androidx.constraintlayout:constraintlayout:${version["constraintLayoutVersion"]}",
            "appcompat"             : "androidx.appcompat:appcompat:${version["appcompatVersion"]}",
            "coreKtx"               : "androidx.core:core-ktx:${version["coreKtxVersion"]}",
            "material"              : "com.google.android.material:material:1.2.1",

            //multidex
            "multidex"              : "com.android.support:multidex:${version["multidexVersion"]}",

            //okhttp
            "okhttp"                : "com.squareup.okhttp3:okhttp:${version["okhttpVersion"]}",
            "logging-interceptor"   : "com.squareup.okhttp3:logging-interceptor:${version["okhttpVersion"]}",

            //retrofit
            "retrofit"              : "com.squareup.retrofit2:retrofit:${version["retrofitVersion"]}",
            "converter-gson"        : "com.squareup.retrofit2:converter-gson:${version["retrofitVersion"]}",
            "adapter-rxjava2"       : "com.squareup.retrofit2:adapter-rxjava2:${version["retrofitVersion"]}",
            "converter-scalars"     : "com.squareup.retrofit2:converter-scalars:${version["retrofitVersion"]}",
			..........省略...........
    ]
}
```

依赖写完之后，在root路径下的build.gradle添加以下代码:   

```groovy
apply from: "config.gradle"
```

然后在需要依赖的module下的build.gradle中: 

```groovy
dependencies {
	...
    // Retrofit + okhttp 相关的依赖包
    api rootProject.ext.dependencies["retrofit"]
    ...
}
```

以上就是Groovy ext扩展函数的依赖管理方式，此方式可以做到版本依赖，但是最大的缺点就是无法跟踪代码，想要找到上面示例代码中的rootProject.ext.dependencies["retrofit"]这个依赖，需要手动切到config.gradle去搜索查找，可读性很差。



## buildSrc+kotlin

Android Gradle插件4.0支持在Gradle构建配置中使用Kotlin脚本 (KTS)，用于替代Groovy（过去在Gradle配置文件中使用的编程语言）。

将来，KTS会比Groovy更适合用于编写Gradle脚本，因为采用Kotlin编写的代码可读性更高，并且Kotlin提供了更好的编译时检查和IDE支持。

虽然与Groovy相比，KTS当前能更好地在Android Studio的代码编辑器中集成，但采用KTS的构建速度往往比采用Groovy慢，因此在迁移到KTS时应考虑构建性能。

KTS:是指Kotlin脚本，这是Gradle在构建配置文件中使用的一种[Kotlin语言形式](https://kotlinlang.org/docs/tutorials/command-line.html#run-scripts)。Kotlin脚本是[可从命令行运行](https://kotlinlang.org/docs/tutorials/command-line.html#using-the-command-line-to-run-scripts)的Kotlin代码。

Kotlin DSL:主要是指[Android Gradle插件Kotlin DSL](https://developer.android.com/reference/tools/gradle-api)，有时也指[底层Gradle Kotlin DSL](https://guides.gradle.org/migrating-build-logic-from-groovy-to-kotlin/)。



摘自 [Gradle 文档](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)：当运行Gradle时会检查根项目中是否存在一个名为buildSrc的目录，该目录包含了项目build相关的逻辑。然后Gradle会自动编译并测试这段代码，并将其放入构建脚本的类路径中, 对于多项目构建，只能有一个buildSrc目录，该目录必须位于根项目目录中,buildSrc是Gradle项目根目录下的一个目录，它可以包含我们的构建逻辑，与脚本插件相比，buildSrc应该是首选，因为它更易于维护、重构和测试代码。

buildSrc的方式，是最近几年特别流行的版本依赖管理方式。它有以下几个优点：

- 支持双向跟踪
- buildSrc是Android默认插件，全局只有这一个地方可以修改
- 支持Android Studio的代码补全

使用方式可参考：[Kotlin + buildSrc for Better Gradle Dependency Management](https://handstandsam.com/2018/02/11/kotlin-buildsrc-for-better-gradle-dependency-management/)

- 在项目根目录下新建一个名为buildSrc的文件夹( 名字必须是buildSrc，因为运行Gradle时会检查项目中是否存在一个名为buildSrc的目录 )
- 在buildSrc文件夹里创建名为build.gradle.kts的文件，添加以下内容： 

```kotlin
plugins {
    `kotlin-dsl`
}
repositories{
    jcenter
}    
```

- 在buildSrc/src/main/java/包名/目录下新建Deps.kt文件，添加以下内容: 

```groovy
object Versions {
    val appcompat = "1.1.0"
}

object Deps {
    val appcompat =  "androidx.appcompat:appcompat:${Versions.appcompat}"
}
```

- 重启Android Studio，项目里就会多出一个名为buildSrc的module。 




缺点: 

- buildSrc 依赖更新将重新构建整个项目，项目越大，重新构建的时间就越长，造成不必要的时间浪费。



### 脚本文件命名

- 用Groovy编写的Gradle build文件使用.gradle文件扩展名。
- 用Kotlin编写的Gradle build文件使用.gradle.kts文件扩展名。

### 常见误区

**用于定义字符串的双引号**。Groovy允许使用单引号来定义字符串，而Kotlin则要求使用双引号。

**基于句点表达式的字符串插值。**在Groovy中，您可以使用`$`前缀来表示基于句点表达式的字符串插值，例如以下代码段中的$project.rootDir：

```
myRootDirectory = "$project.rootDir/tools/proguard-rules-debug.pro"
```

但在Kotlin中，上述代码将对 `project`（而非 `project.rootDir`）调用 `toString()`。如需获取根目录的值，请使用大括号括住整个变量：

- `myRootDirectory = "${project.rootDir}/tools/proguard-rules-debug.pro"`
- **变量分配**。一些在Groovy中适用的分配方式现在会被视作setter（或者，对于列表、集合等，则适用“addX”)，因为属性在Kotlin中是只读的。

### 显式和隐式buildTypes

在Kotlin DSL中，某些buildTypes（如debug和release）是隐式提供的。但是，其他buildTypes则必须手动创建。

例如，在Groovy中，您可能有debug、release和staging` `buildTypes：

**Groovy**

```groovy
buildTypes debug {  ... } release {  ... } staging {  ... }
```

在KTS中，仅debug和release` `buildTypes是隐式提供的，而staging则必须由您手动创建：

**KTS**

```
buildTypes getByName("debug") {  ... } getByName("release") {  ... } create("staging") {  ... }
```

### 使用plugins代码块

如果您在build文件中使用plugins代码块，IDE将能够获知相关上下文信息，即使在构建失败时也是如此。IDE可使用这些信息执行代码补全并提供其他实用建议，从而帮助您解决KTS文件中存在的问题。

在您的代码中，将命令式apply plugin替换为声明式plugins代码块。Groovy中的以下代码:  

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-kapt'
apply plugin: 'androidx.navigation.safeargs.kotlin'
```

…在 KTS 中变为以下代码：

```kotlin
plugins {  
    id("com.android.application")  
    id("kotlin-android")  
    id("kotlin-kapt")  
    id("androidx.navigation.safeargs.kotlin") 
}
```

如需详细了解plugins代码块，请参阅 [Gradle 的迁移指南](https://docs.gradle.org/nightly/userguide/migrating_from_groovy_to_kotlin_dsl.html#applying_plugins)。





## Composing builds

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/composite_build.jpeg?raw=true)



Composing builds：A composite build is simply a build that includes other builds. In many ways a composite build is similar to a Gradle multi-project build, except that instead of including single projects, complete builds are included.
复合构建只是包含其他构建的构建. 在许多方面，复合构建类似于Gradle多项目构建，不同之处在于，它包括完整的builds，而不是包含单个projects: 

- 组合通常独立开发的构建，例如，在应用程序使用的库中尝试错误修复时
- 将大型的多项目构建分解为更小，更孤立的块，可以根据需要独立或一起工作

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/buildsrc_composingbuild.png?raw=true)

**使用方式**

1. 新建module，名为versionPlugin（自起）
2. 在该module下的build.gradle文件中，添加如下代码：

```groovy
buildscript {
    ext.kotlin_version = "1.5.10"
    repositories {
        mavenCentral()
    }
    dependencies {
        // 因为使用的Kotlin需要需要添加Kotlin插件
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin'
apply plugin: 'java-gradle-plugin'

repositories {
    mavenCentral()
}

compileKotlin {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
compileTestKotlin {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

gradlePlugin {
    plugins {
        version {
            // 在app模块需要通过id引用这个插件
            id = 'com.xx.xx.plugin'
            // 实现这个插件的类的路径
            implementationClass = 'com.xx.xx.versionplugin.Deps'
        }
    }
}
```

3. 在versionPlugin/src/main/java/包名/目录下新建Deps.kt文件，添加你的依赖配置，如：

```groovy
package com.xx.xx.versionplugin

class Deps : Plugin<Project> {
    override fun apply(project: Project) {
        // Possibly common dependencies or can stay empty
    }

    companion object {
        val appcompat = "androidx.appcompat:appcompat:1.1.0"
    }
}
```

或者也可以按依赖类型用不同的类配置，例如

```kotlin
object CustomLibs {
    ...
        object Glide {
        private const val glideVersion = "4.11.0"
        const val glide = "com.github.bumptech.glide:glide:$glideVersion"
        const val glideCompiler = "com.github.bumptech.glide:compiler:$glideVersion"
    }

    object Retrofit {
        private const val retrofitVersion = "2.9.0"
        const val retrofit = "com.squareup.retrofit2:retrofit:$retrofitVersion"
        const val converter_gson = "com.squareup.retrofit2:converter-gson:$retrofitVersion"
    }
}
```



4. 在settings.gradle文件内添加`includeBuild 'versionPlugin'`，注意是includeBuild哦~，Rebuild项目    

5. 后面就可以在需要使用的gradle文件中使用了，在app或其他module下的build.gradle，在首行添加以下内容：

```groovy
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt'
    // 通过id来使用该plugin，这个id就是在versionPlugin文件夹下build.gradle文件内定义的id
    id 'com.xx.xx.plugin'
}
```

使用如下:  

```groovy
dependencies {
    implementation CustomLibs.Glide.glide
    kapt CustomLibs.Glide.glideCompiler
}    
```





# 参考

- [Kotlin + buildSrc for Better Gradle Dependency Management](https://handstandsam.com/2018/02/11/kotlin-buildsrc-for-better-gradle-dependency-management/)
- [How to use Composite builds as a replacement of buildSrc in Gradle](https://medium.com/bumble-tech/how-to-use-composite-builds-as-a-replacement-of-buildsrc-in-gradle-64ff99344b58)
- [Stop using Gradle buildSrc. Use composite builds instead](https://proandroiddev.com/stop-using-gradle-buildsrc-use-composite-builds-instead-3c38ac7a2ab3)
- [再见吧 buildSrc, 拥抱 Composing builds 提升 Android 编译速度](https://juejin.cn/post/6844904176250519565)
- [composite_builds](https://docs.gradle.org/current/userguide/composite_builds.html)

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
