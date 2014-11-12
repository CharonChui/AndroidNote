AndroidStudio使用教程(第二弹)
===

- 迁移`Eclipse`工程到`Android Studio`

    官方文档中说`Android Studio`可以兼容`Eclipse`的现有工程，但需要做一些操作：    
	
    - `Eclipse`进行项目构建
	    首先升级`ADT`到最新版本, 好像是22之后，选择需要从`Eclipse`导出的工程，右键选择`Export`并选择`Android`下的`Generate Gradle Build Files`, 运行完成之后你会
	发现在项目目录中多了一个`build.gradle`, 这就是`Android Studio`所识别的文件。当然了官方也说了即使你不这样操作，`Android Studio`也会根据你项目中的`Ant`配置来导入，
	但是为了更好的兼容，他建议你这样做，既然建议了，我们就要听。
    
	- 导入 
	    在`Android Studio`中选择`Import Project`,并选择刚才工程目录下的`build.gradle`即可。

有些时候会发现导入之后在运行按钮左边显示不出`Module`来，可能是你导入之前的`SDK`版本不同导致的，只要在`build.gradle`中配置相应的`SDK`版本就可以了。
```java
android {
    compileSdkVersion 19
    buildToolsVersion "21.1.1"
	...
	}
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 