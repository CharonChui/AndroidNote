AndroidStudio提高Build速度
===

`Android Studio`自动发布以来，凭借其强大的功能，很快让开发者都投入到它的阵营下。但是也问题，就是`Build`速度太慢了。

之前也根据网上的资料，修改了一些配置，但是一次二分多种有时候还是让人抓狂。今天索性记录一下。首先看一下[Google+](https://plus.google.com/+AndroidDevelopers/posts/ECrb9VQW9XP)关于该问题的讨论。


- 使用`daemon`及`parallel`模式

    在下面的目录中创建一个名为`gradle.properties`的文件:     

    - `/home/<username>/.gradle/ (Linux)`
    - `/Users/<username>/.gradle/ (Mac)`
    - `C:\Users\<username>\.gradle (Windows)`

    文件的内容为: 
    ```
    org.gradle.daemon=true
    org.gradle.parallel=true
    ```
    
    经过上面的这一步修改是对所有工程都有效果的，如果你只想对某一个工程配置的话，那就在该工程目录下的`gralde.properties`中进行配置。
    ```
    # Project-wide Gradle settings.
    
    # IDE (e.g. Android Studio) users:
    # Gradle settings configured through the IDE *will override*
    # any settings specified in this file.
    
    # For more details on how to configure your build environment visit
    # http://www.gradle.org/docs/current/userguide/build_environment.html
    
    # Specifies the JVM arguments used for the daemon process.
    # The setting is particularly useful for tweaking memory settings.
    # Default value: -Xmx10248m -XX:MaxPermSize=256m
    # org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
    
    # When configured, Gradle will run in incubating parallel mode.
    # This option should only be used with decoupled projects. More details, visit
    # http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
    # org.gradle.parallel=true
    ```
    打开时默认如上。我们给加添加上面的配置就好。
    
    当然你也可以通过`Studio`的设置中进行修改。        
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_speed.png?raw=true)


- 使用`offline`模式      
  
    下一步就是开始`offline`模式，因为我们经常会在`gradle`中使用一下依赖库时用`+`这样的话就能保证你的依赖库是最新的版本，但是这样在每次`build`的时候都会去检查是不是最新的版本，所以就会耗时。        
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_offline.png?raw=true)

    在开发过程中是不建议使用动态版本的，在`Studio`中使用动态版本的`gradle`中间中使用`ALT+ENTER`键进行修复。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_daymaic_version_tip.png?raw=true)
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_dymaic_version_fix.png?raw=true)
    详细有关为什么不要使用动态版本的介绍，请参考[Don't use dynamic versions for your dependencies](http://blog.danlew.net/2015/09/09/dont-use-dynamic-versions-for-your-dependencies/)

- 增加内存使用`SSD`         
    首先是增大内存,    `Mac`中在`Applications`中找到`Sutio`然后右键显示包内容`Contents/bin/studio.vmoptions`。
    打开该文件后修改就可以了，我是的是:     
    ```
    #
    # *DO NOT* modify this file directly. If there is a value that you would like to override,
    # please add it to your user specific configuration file.
    #
    # See http://tools.android.com/tech-docs/configuration
    #
    -Xms1024m
    -Xmx4096m
    -XX:MaxPermSize=768m
    -XX:ReservedCodeCacheSize=768m
    -XX:+UseCompressedOops
    ```
    我没看见`DO NOT`的提示- -!
    
    - Xms 是JVN启动起始时的堆内存，堆内存是分配给对象的内容。
    - Xmx 是能使用的最大堆内存。


- 使用`Instant Run`   
         
    `Instantt Run`放在这里说可能不合适，但是用他确实能大大的减少运行时间。              
    如果还不了解的话可以参考[Instant Run](http://tools.android.com/tech-docs/instant-run)           
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_instantrun.png?raw=true)
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 