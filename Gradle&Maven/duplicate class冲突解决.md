duplicate class冲突解决
===

```
Duplicate class com.x.util.Base64Encoder found in modules jetified-b64encode-1.0.8-runtime (com.x.x.x.x:b64encode:1.0.8) and jetified-b64encode_v2_0 (b64encode_v2_0.jar)
```
今天在开发过程中遇到了这个错误。提示Base64Encoder在com.x.x.x.x:b64encode:1.0.8和b64encode_v2_0.jar里面重复了，一个是1.0.8版本一个是2.0版本。
那么这里要做的就是exclude一个就可以。 

1. 首先需要找到是哪个依赖中有该依赖:  
通常情况下我们会直接查找一下该类就能看到有那几个库中包含，但是这里只能查到jar包的类，所以需要用另一种方式。
Terminal执行:  
```
./gradlew app:dependencies
```
执行后会将所有的依赖列出:  
```
|    \--- androidx.annotation:annotation:1.1.0 -> 1.2.0
+--- com.google.android.gms:play-services-tasks:17.2.1
|    \--- com.google.android.gms:play-services-basement:17.6.0
|         +--- androidx.collection:collection:1.0.0 -> 1.1.0
|         |    \--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|         +--- androidx.core:core:1.2.0 -> 1.5.0
|         |    +--- androidx.annotation:annotation:1.2.0
|         |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.3.1
|         |    |    +--- androidx.arch.core:core-runtime:2.1.0
|         |    |    |    +--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|         |    |    |    \--- androidx.arch.core:core-common:2.1.0
|         |    |    |         \--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|         |    |    +--- androidx.lifecycle:lifecycle-common:2.3.1 (*)
|         |    |    +--- androidx.arch.core:core-common:2.1.0 (*)
|         |    |    \--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|         |    +--- androidx.versionedparcelable:versionedparcelable:1.1.1
|         |    |    +--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|         |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|         |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|         \--- androidx.fragment:fragment:1.0.0 -> 1.3.4
|              +--- androidx.annotation:annotation:1.1.0 -> 1.2.0
|              +--- androidx.core:core:1.2.0 -> 1.5.0 (*)
|              +--- androidx.collection:collection:1.1.0 (*)
```

2. 接下来就是在该列表中搜索，找到后直接在`build.gradle`中使用exclude:  
```
implementation("com.xxx.xxx:pass-sdk-core-router:${PASS_VERSION}") {
    // 去除扫码相关功能
    exclude group: "com.xxx.passport", module: "pass-module-qrcode"
    // 去除人脸登录相关功能
    exclude group: "com.xxx.passport", module: "pass-module-face"
}
```

3. 但是有时候会发现有很多个库中都会有该依赖，一个一个的去添加不太适合，这时可以在app的buid.gradle中统一配置:  
```
android {
    compileSdkVersion rootProject.android.extCompileSdkVersion
    buildToolsVersion rootProject.android.extBuildToolsVersion
    useLibrary 'org.apache.http.legacy'
    compileOptions {
        targetCompatibility JavaVersion.VERSION_1_8
        sourceCompatibility JavaVersion.VERSION_1_8
    }
    configurations {
        implementation.exclude group: 'com.xxx.xxx.common.toolbox' , module:'b64encode'
    }
}
```











---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 