Android卸载反馈
===

最初记得是在360安全卫士中出现的，在手机上卸载他的应用之后浏览器就会弹出一个反馈页面，让用户进行反馈，感觉这种功能对于产品改进特别有帮助。
但是仔细一想该怎么去实现却犯愁了，最开始想这也简单啊，不就是监听下自身被卸载就可以了，应该系统会有卸载的广播，可惜没有。甚至其他的一些
方法也是不行的，因为你程序都被卸载了，你的代码怎么会执行呢？皮之不存，毛将焉附。那360是怎样实现的呢？说句真心话360做的产品还是非常有创新
的，虽然有些时候他会损坏用户的利益，不过但从技术方面，着实让人信服。

既然`Java`实现不了，那就得考虑下其他的了，自然最先想到的就是`JNI`了，可惜`C`的部分不懂，上网搜了很多资料和介绍，找得到可以通过一下方式实现:   
- 通过`c`中的`fork`方法来复制一个子进程。复制出来的子进程在父进程被销毁后，仍然可以存在。
    `pid_t fpid = fork()`被调用前，就一个进程执行该段代码，这条语句执行之后，就会有两个进程执行该代码，两个进程执行没有固定先后顺序，只要看
	系统调度策略，`fork()`函数的特别之处在于调用一次会返回两次结果：   
	    - 返回值大于0，当前是父进程。
		- 返回0，当前是子进程。
		- 返回小于0的负值。(出错了，可能是内存不足或者是进程数已经达到系统最大值)

- `fork`出子进程后让子进程一直去监听`/data/data/packageName`是否存在，如果不存在了，那就说明程序被卸载了，但是这样一直去轮训判断肯定会浪费
    系统资源的，当然也会更加费电，对用户来讲肯定是有损害的。所以这种技术最好也不好用，不然大家的手机以后还能了得。万恶的产品。

- 得到程序被卸载之后弹出浏览器打开指定反馈页面。这就要用到`am`命令了，最早知道这个命令是在开发`TV`版的时候，遥控器找不到了，程序安装后无法
   打开了，用该命令就不怕了，哈哈。但是在`c`中怎么执行`am`命令呢？这就要用到`execlp()`函数，该函数就是`c`中执行系统命令的函数。
   

好了，主要的内容上面都分析完了，下面上代码。

- `Java`层定义`natvie`方法。   
```java
public class MainActivity extends Activity {
	private static final String TAG = "@@@";

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		String packageDir = "/data/data/" + getPackageName();
		initUninstallFeedback(packageDir, Build.VERSION.SDK_INT);
	}

	private native void initUninstallFeedback(String packagePath, int sdkVersion);

	static {
		System.loadLibrary("uninstall_feedback");
	}
}
```

- 创建jni目录，增加`Android.mk`以及`uninstall_feedback.c`文件。
`Android.mk`的内容:

```c
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := uninstall_feedback
LOCAL_SRC_FILES := uninstall_feedback.c

LOCAL_C_INCLUDES := $(LOCAL_PATH)/include
LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib -llog

include $(BUILD_SHARED_LIBRARY)
```

`uninstall_feedback.c`的实现: 

```c
/**
 * 将Java中的String转换成c中的char字符串
 */
char* Jstring2CStr(JNIEnv* env, jstring jstr) {
	char* rtn = NULL;
	jclass clsstring = (*env)->FindClass(env, "java/lang/String"); //String
	jstring strencode = (*env)->NewStringUTF(env, "GB2312"); // 得到一个java字符串 "GB2312"
	jmethodID mid = (*env)->GetMethodID(env, clsstring, "getBytes",
			"(Ljava/lang/String;)[B"); //[ String.getBytes("gb2312");
	jbyteArray barr = (jbyteArray)(*env)->CallObjectMethod(env, jstr, mid,
			strencode); // String .getByte("GB2312");
	jsize alen = (*env)->GetArrayLength(env, barr); // byte数组的长度
	jbyte* ba = (*env)->GetByteArrayElements(env, barr, JNI_FALSE);
	if (alen > 0) {
		rtn = (char*) malloc(alen + 1); //""
		memcpy(rtn, ba, alen);
		rtn[alen] = 0;
	}
	(*env)->ReleaseByteArrayElements(env, barr, ba, 0); //
	return rtn;
}

void Java_com_charon_uninstallfeedback_MainActivity_initUninstallFeedback(
		JNIEnv* env, jobject thiz, jstring packageDir, jint sdkVersion) {

	char * pd = Jstring2CStr(env, packageDir);

	//fork子进程，以执行轮询任务
	pid_t pid = fork();

	if (pid < 0) {
		// fork失败了
	} else if (pid == 0) {
		// 可以一直采用一直判断文件是否存在的方式去判断，但是这样效率稍低，下面使用监听的方式，死循环，每个一秒判断一次，这样太浪费资源了。
		int check = 1;
		while (check) {
			FILE* file = fopen(pd, "rt");
			if (file == NULL) {
				if (sdkVersion >= 17) {
					// Android4.2系统之后支持多用户操作，所以得指定用户
					execlp("am", "am", "start", "--user", "0", "-a",
							"android.intent.action.VIEW", "-d",
							"http://shouji.360.cn/web/uninstall/uninstall.html",
							(char*) NULL);
				} else {
					// Android4.2以前的版本无需指定用户
					execlp("am", "am", "start", "-a",
						"android.intent.action.VIEW", "-d",
							"http://shouji.360.cn/web/uninstall/uninstall.html",
							(char*) NULL);
				}
				check = 0;
			} else {
			}
			sleep(1);
		}
	} else {
	}
}

```

- 编译so文件。`Windows`下要用`cygwin`来操作。
上面的介绍是在`Eclipse`中进行的，用`ndk-build`命令来编译`so`。具体请看之前写的`JNI基础`这篇文章。

有关如何在[Android Stuido中进行ndk开发请看][1]。
   		

[1]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%B8%AD%E8%BF%9B%E8%A1%8Cndk%E5%BC%80%E5%8F%91.md "Android Stuido中进行ndk开发"

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 