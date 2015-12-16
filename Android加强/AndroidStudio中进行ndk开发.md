AndroidStudio中进行ndk开发
===

- 在`module/src/main/`下创建`jni`文件夹

    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_ndk_jni.png?raw=true)    

- 配置`ndk`路径，在项目右键`Moudle Setting`中设置。
	
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
				moduleName "app"
			}
		}
		buildTypes {
			release {
				minifyEnabled false
				proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			}
		}
		
		productFlavors {
//        // for detailed abiFilter descriptions, refer to "Supported ABIs" @
//        // https://developer.android.com/ndk/guides/abis.html#sa
//        create("arm") {
//            ndk.abiFilters.add("armeabi")
//        }
//        create("arm7") {
//            ndk.abiFilters.add("armeabi-v7a")
//        }
//        create("arm8") {
//            ndk.abiFilters.add("arm64-v8a")
//        }
//        create("x86") {
//            ndk.abiFilters.add("x86")
//        }
//        create("x86-64") {
//            ndk.abiFilters.add("x86_64")
//        }
//        create("mips") {
//            ndk.abiFilters.add("mips")
//        }
//        create("mips-64") {
//            ndk.abiFilters.add("mips64")
//        }
        // To include all cpu architectures, leaves abiFilters empty
        create("all")
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
	可以在`lib`目录中找到响应的`so`文件，注意名字是以`lib`+`module name`命名的。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_ndk_build.png?raw=true)    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 