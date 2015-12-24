AndroidStudio中进行ndk开发
===

- 创建工程，声明`native`方法。               
	```java
	private native void startDaemon(String serviceName, int sdkVersion);

    static {
        System.loadLibrary("daemon");
    }
	```
	
	
- 生成`class`文件。                 
    执行`Build-Make Project`命令，生成`class`文件。所在目录为`app_path/build/intermediates/classes/debug`


- 执行`javah`生成`.h文件`                    
	```
	C:\Users\Administrator>javah -help
	用法:
	  javah [options] <classes>
	其中, [options] 包括:
	  -o <file>                输出文件 (只能使用 -d 或 -o 之一)
	  -d <dir>                 输出目录
	  -v  -verbose             启用详细输出
	  -h  --help  -?           输出此消息
	  -version                 输出版本信息
	  -jni                     生成 JNI 样式的标头文件 (默认值)
	  -force                   始终写入输出文件
	  -classpath <path>        从中加载类的路径
	  -cp <path>               从中加载类的路径
	  -bootclasspath <path>    从中加载引导类的路径
	```	
	在`Studio Terminal`中进入到`src/main`目录下执行`javah`命令:       
	`javah -d jni -classpath <SDK_android.jar>;<APP_classes> <class>`
	
	`F:\DaemonService\app\src\main>javah -d jni -classpath C:\develop\android-sdk-windows\platforms\android-22\android.jar;..\..\build\intermediates\classes\debug com.charonchui.daemonservice.service.DaemonService`
	执行完成后就会在`src/main/jni`目录下生成`com_charonchui_daemonservice_service_DaemonService.h`文件。

- 在`module/src/main/jni`目录下创建对应的`.c`文件。



- 配置`ndk`路径，在项目右键`Moudle Setting`中设置。              
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_ndk_jni.png?raw=true)    
	
- 在`build.gradle`中配置`ndk`选项              

    ```java
	android {
		compileSdkVersion 23
		buildToolsVersion "23.0.1"

		defaultConfig {
			applicationId "com.charonchui.daemonservice"
			minSdkVersion 8
			targetSdkVersion 23
			versionCode 1
			versionName "1.0"

			ndk {
				moduleName "uninstall_feedback" // 配置so名字
				ldLibs "log"
	//            abiFilters "armeabi", "x86"  // 默认就是全部的，加了配置才会生成选中的
			}
		}
		buildTypes {
			release {
				minifyEnabled false
				proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			}
		}
	}
	```
	这里可能会出现错误:      
	- `Error: NDK integration is deprecated in the current plugin. Consider trying the new experimental plugin. For details, see http://tools.android.com/tech-docs/new-build-system/gradle-experimental. Set "android.useDeprecatedNdk=true" in gradle.properties to continue using the current NDK integration.`
	    解决方法就是在`gradle.properties`文件中添加`android:useDeprecatedNdk=true`就可以了。
	- `Error:Execution failed for task ':app:compileDebugNdk'.
		> com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process 'command 'E:\android-ndk-r10\ndk-build.cmd'' finished with non-zero exit value 2`
		解决方法就是在`jni`目录建一个任意名字的`.c`空文件就可以了。
	
- 执行Build      
	然后就可以在`app/build/intermediates/ndk/debug/obj/local`下看到所有架构的`so`了。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_ndk_build.png?raw=true)    
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 