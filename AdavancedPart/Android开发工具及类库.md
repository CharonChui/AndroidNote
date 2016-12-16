Android开发工具及类库
===

在项目开发过程中，总有一些必要的工具和类库。下面就简单介绍下我常用的一些(还在用`Eclipse`的请无视)。      

1. [volley](https://android.googlesource.com/platform/frameworks/volley)                                                   
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/volley.png?raw=true)                   
在`Google I/0 2013`中发布了`Volley`.`Volley`是`Android`平台上的网络通信库，能使网络通信更快，更简单，更健壮。
这是`Volley`名称的由来:`a burst or emission of many things or a large amount at once`.`Volley`特别适合数据量不大但是通信频繁的场景。   
`Github`上面已经有大神做了镜像，使用更方便有木有。[Volley On Github](https://github.com/mcxiaoke/android-volley)                     

2. [Gson](https://code.google.com/p/google-gson/)                    
`Json`转换神器。

3. [GsonFormat](https://github.com/zzz40500/GsonFormat)               
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/GsonFormat.gif?raw=true)                 
既然用了`Gson`怎么能少了该神器呢？

4. [android-butterknife-zelezny](https://github.com/avast/android-butterknife-zelezny)         
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/zelezny_animated.gif?raw=true)      
使用[butterknife](https://github.com/JakeWharton/butterknife)制作的`Android Studio/IDEA`插件。非常方便有木有。

5. [android-selector-chapek](https://github.com/inmite/android-selector-chapek)       
`selector`写起来是不是很麻烦？以后让`UI`规范化命名，然后就没有然后了。                
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/select_folder.png?raw=true)            
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/select_option.png?raw=true)         
接下来你就会在`drawable`目录发现对应的`selector`文件。           
        
6. [leakcanary](https://github.com/square/leakcanary)		  
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/screenshot.png?raw=true)       
内存泄漏你怕不怕？         
		
7. [fresco](https://github.com/facebook/fresco)		      
怎么能少了对图片的处理呢？`Fracebook`出品。更快、更强、更方便。       

8. [android-resource-remover](https://github.com/KeepSafe/android-resource-remover)                    
    开发过程中可能会经常遇到需求的变更，时间长了，项目中的无用资源就会越来越多。 虽然在`Gradle`中支持相应的配置来去除无用资源: 

	```xml                               
	buildTypes {
        debug {
            minifyEnabled false
            zipAlignEnabled false
            shrinkResources false
        }

        release {
            zipAlignEnabled true
            // remove unused resources
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
	```
    
    但是这只是在打包的时候不会打进去无用的资源，但是这些资源还是会在工程中。
    那我们怎么能快速的移除掉这些无用资源呢？答案也很简单，就是使用`lint`检查出无用的资源后用工具删除，这个工具就是`android-resource-remover`。 
    因为它是一个`python`脚本，所以如果不懂`python`的话使用起来会比较麻烦，下面就介绍一下具体的使用方法:            
    - 下载并安装`Python 2.x`版本
        去[Python](https://www.python.org/)下载后即可，这里要下载2.x版本，因为3.x版本对语法做了很多改动，可能会不兼容，下载完成后安装就可。安装完成后将安装路径加入到`Path`中。如`D:\Python;`。
    - 安装`android-resource-remover`     
	    在命令行输入下面的命令`pip install android-resource-remover`。 这里有些电脑可能会提示错误，是因为没有安装`pip`导致的，具体可以看[pip](https://pip.pypa.io/en/latest/installing.html)找到安装的方法。上面介绍了要下载`get-pip.py`后执行`python get-pip.py`就能安装了。
	- 将`D:\Python\Scripts`添加到`Path`中。
    - 将`lint`命令添加到`Path`中，`D:\android-sdk-windows\tools`.
    - 在`Studio`右侧的`Gradle`窗口中执行`lint`任务。 这样就会在`app/build/outputs`下生成`lint-results.xml`文件。下一步清理的时候需要使用`lint-results.xml`文件。               
	    ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/lint.png?raw=true)	
	- 进入到`Android Studio`中的具体项目中执行`./gradlew clean`后再执行`./gradlew lint && android-resource-remover --xml app/build/outputs/lint-results.xml`

9. [stetho](https://github.com/facebook/stetho)
    `facebook`出品。快速查看布局、数据库、网络请求。实在不能再方便了。   
10. [RxJava](https://github.com/ReactiveX/RxJava)
    用了后你会爱上它。
11. [Retrofilt](https://github.com/square/retrofit)
    `Square`出品。大神`JakeWharton`主导出品的网络请求框架。内部结合`OkHttp`。结合`RxJava`使用非常方便。   
12. [android-architecture](https://github.com/googlesamples/android-architecture)
    放到这里可能不太合适，因为它并不是工具和类库，而是`Google`官方发布的`Android`架构示例。非常值得参考。  
13. [AndroidWiFiADB](https://github.com/pedrovgs/AndroidWiFiADB)
    还在为数据线不够用而烦恼嘛?
    

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
