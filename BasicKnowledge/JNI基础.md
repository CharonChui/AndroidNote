JNI基础
===

1. 将java中的字符串转换成C中字符串的工具方法              
    ```c
    char*   Jstring2CStr(JNIEnv*   env,   jstring   jstr){
         char*   rtn   =   NULL;
         jclass   clsstring   =   (*env)->FindClass(env,"java/lang/String");
         jstring   strencode   =   (*env)->NewStringUTF(env,"GB2312");
         jmethodID   mid   =   (*env)->GetMethodID(env,clsstring,   "getBytes",   "(Ljava/lang/String;)[B");
         jbyteArray   barr=   (jbyteArray)(*env)->CallObjectMethod(env,jstr,mid,strencode); // String .getByte("GB2312");
         jsize   alen   =   (*env)->GetArrayLength(env,barr);
         jbyte*   ba   =   (*env)->GetByteArrayElements(env,barr,JNI_FALSE);
         if(alen   >   0)
         {
          rtn   =   (char*)malloc(alen+1);         //"\0"
          memcpy(rtn,ba,alen);
          rtn[alen]=0;
         }
         (*env)->ReleaseByteArrayElements(env,barr,ba,0);  //
         return rtn;
    }
    ```

2. 程序被运行要经历两个步骤(1.编译 2.链接)                      
    编译就是将源文件编译成二进制代码，而链接则是将二进制代码转换成可执行的文件如.exe等头文件.(函数的声明,函数的清单文件)作用: 给编译器看的.     
    库函数: 头文件里面函数的实现.    
    作用:给连接器看的.    
 
3. jni开发的常见错误:        
    错误1: 忘记编写android.mk文件 unknown file: ./jni/Android.mk    
    Android NDK: Your APP_BUILD_SCRIPT points to an unknown file: ./jni/Android.mk      
    /cygdrive/c/android-ndk-r7b/build/core/add-application.mk:133: Android NDK: Aborting...    。 停止。          
    
    错误2:  ndk-build 没有任何反应.        
    忘记配置android.mk脚本       
	
    错误3: $ ndk-build jni/Android.mk:4:  遗漏分隔符 。 停止。      
中文的回车或者换行         

    错误4:java.lang.UnsatisfiedLinkError: hello      
	忘记加载了c代码的.so库 或者 函数的签名不正确,没有找到与之对应的c代码      
 
    错误5:07-30 java.lang.UnsatisfiedLinkError: Library Hel1o not found      
	没有找到对应的c代码库     
 
    错误6: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***       
    07-30 11:53:17.898: INFO/DEBUG(31): Build fingerprint:     
    'generic/sdk/generic/:2.2/FRF91/43546:eng/test-keys'      
    c代码里面有严重的逻辑错误,产生内存的泄露.    
 
    错误7:     
    make: *** [obj/local/armeabi/objs/Hello/Hello.o] Error 1         
    编译的时候 程序出现了问题,c语言的语法有问题           
    c语言代码编译错误的时候 先去解决第一个错误. 
 
4. Java调用JNI的前提        
    开发所使用的电脑(windows系统, x86的CPU)       
    目标代码: android手机上运行的.( linux系统, arm的CPU)           
    所以我们要模拟手机的系统,手机的处理器，生成手机上可以运行的二进制代码这就要用到交叉编译;            
    根据运行的设备的不同，可以将cpu分为：          
    - arm结构 ：主要在移动手持、嵌入式设备上。      
    - x86结构 ： 主要在台式机、笔记本上使用。如Intel和AMD的CPU 。      
    交叉编译: 在一种操作系统平台或者cpu平台下 编译生成 另外一个平台(cpu)可以运行的二进制代码.(使用NDK中的ndk-build命令)     


- 工具一:  交叉编译的工具链: NDK                 
    NDK全称：Native Development Kit 。           
    - NDK是一系列工具的集合，它有很多作用。          
       - 首先，NDK可以帮助开发者快速开发C(或C++)的动态库。      
       - 其次，NDK集成了交叉编译器。使用NDK，我们可以将要求高性能的应用逻辑使用C开发，从而提高应用程序的执行效率。     
       
    NDK工具是提供给Linux系统用的(随着版本的升级也可以直接在Windows下使用，但是现在仍不完善有bug)，
    所以要在windows下使用ndk的工具,必须要提供一个工具(linux环境的模拟器)

- 工具二:  cygwin(windows下linux系统环境的模拟器, 主要是为了能够运行ndk的工具)       
    安装 devel shell       
	![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/jni_cygwin.png?raw=true)    
    linux 特点:所有的设备 硬件 都是以文件的方式定义的.     
	安装完后进入`cygwin`打印`make -v`命令如果能打印出`GNU Make ...`就说明安装木问题了。

- 工具三: cdt(c/c++ develop tools)  eclipse 的一个插件  用来让c\c++代码 语法高亮显示.         
    adt(android develop tools)          

- 工具四：           
    为了不用每次使用ndk-build命令都要进入到ndk的安装目录，这里要进行Path变量的配置。      
    配置cygwin的环境变量: 在cygwin安装目录,etc目录,profile的文件 32行 添加ndk工具所在的目录.
    `PATH="/usr/local/bin:/usr/bin:/cygdrive/d/android-ndk-r7b:${PATH}"`在这个后面加上:ndk-build的路径(注意：在linux中路径的分隔符不是分号而是冒号)，
	改成这样       
	`PATH="/usr/local/bin:/usr/bin:${PATH}:/cygdrive/d/android-ndk-r7b"`//注意这里的路径是在linux系统下的ndk路径而不是windows下的路径,/cygdrive/d/是在linux下看到的d盘。

### JNI开发步骤：

1. 创建一个android工程
2. JAVA代码中写声明native 方法 public native String helloFromJNI();
3. 用javah工具生成头文件
4. 创建jni目录,引入头文件,根据头文件实现c代码
5. 编写Android.mk文件
6. Ndk编译生成动态库
7. Java代码load 动态库.调用native代码

### JNI开发之Java中调用C代码步骤

1. 在java中定义一个要调用的C的方法(本地方法)            
	//1.定义一个native的方法            
	`public native String helloFromC();`

2. 在工程中新建一个jni文件夹(然后在这个文件夹中写c代码，在C中实现java里面定义的c方法默认的时候是自己手写c的方法名，           
	但是很麻烦这里要参考七里面提供的方式，用javah编译后，然后拷贝h的头文件到jni文件夹中，在从h文件拷贝方法的名字，然后实现该方法).  
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
	`#include <stdio.h>`         
	`#include <jni.h>`        
	//这个方法的名字的写法固定Java_表示这个方法由Java调用,cn_itcast_ndk表示java的包名DemoActivity表示
	//java中调用这个方法的类名helloFromC表示java中调用这个方法的方法名字
	```java
	jstring Java_cn_itcast_ndk_DemoActivity_helloFromC (JNIEnv* env , jobject obj){//这两个参数是固定必不可少的
		   //return (*(*env)).NewStringUTF(env,"hello from c!");//调用NewStringUTF这个方法new出来一个java中的String类型的字符串
		   return (*env)->NewStringUTF(env,"hello from c!" );//这个写法和上面这个注释掉的一样，只是更简洁一点
	}
	```	
	
3. 在jni文件夹中编写android.mk文件，在这个文件夹中声明要编译的c文件名以后编译后生成的文件名   
	```c
	LOCAL_PATH := $(call my-dir)  //将jni所在的目录返回去到LOCAL_PATH
	#clear_vars 一个函数 初始化编译工具链的所有的变量.
	#特点:清空所有的以LOCAL_开头的变量,但是不会清空LOCAL_PATH的变量
	 include $(CLEAR_VARS)  
	#指定编译后的文件的名称 符合linux系统下makefile的语法. 
	LOCAL_MODULE    := Hello
	#指定编译的源文件的名称 ,编译器非常智能
	LOCAL_SRC_FILES := Hello.c
	  #指定编译后的文件的类型. 默认编译成动态库 BUILD_SHARED_LIBRARY 扩展名.so .so代码体积很小  
	#                            静态库 BUILD_STATIC_LIBRARY 扩展名.a  .a代码体积很大
	include $(BUILD_SHARED_LIBRARY)
	```
4. cmd进入到当前的工程的文件夹中(也可以进入到当前工程的jni目录中)，然后运行ndk-build工具就能将c文件编译成一个可执行的二进制文件. ->.so，        
    注意用ndk-build编译之后一定要刷新，不然eclipse会缓存旧的不加载新的进来
	![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/ndk_build.png?raw=true)    
	
	
5. 刷新工程，就能看到多出了两个文件夹

6. 在java中将要调用的c代码加载到java虚拟机中，通过静态代码块的方式
	```java
	public class DemoActivity extends Activity {
	   //1.定义一个native的方法
	   public native String helloFromC();
 
	   static{
			 //5.把要调用的c代码 给加载到java虚拟机里面
			System. loadLibrary("Hello");//注意写的是Hello不要加后缀
		}
	}
	```
	
7. 调用c代码            
	```java
	public void click(View view){
	  //调用c代码
	  Toast.makeText(this, helloFromC(), 1).show();
	  
	}
	```

7. 利用jdk的工具javah动态生成c方法名         
    在上面的调用c中的方法的时候，在c中区实现这个方法的时候的方法名字写起来很复杂，而且容易出去，在java在jdk中提供了一个工具javah，
	我们只要在windows的dos窗口cmd到classes目录下去执行javah 包名.类名就能够由class文件动态的生成一个c的h文件，在这个h文件中有该class文件中的native方法的名字       
    我们只要拷贝这个h文件到自己工程的jni目录中，然后在c文件中引入这个h文件，并拷贝这个h文件中的方法去实现就可以了      
    ```java
    #include <stdio.h>//这个<>是引入工具的h文件        
    #include "cn_itcast_ndk2_DemoActivity.h" //对于自己工程中的h文件用""来引入或者引入#include <jni.h>也可以      

	JNIEXPORT jstring JNICALL Java_cn_itcast_ndk2_DemoActivity_hello_1_1_1from_1_1_1c //拷贝h文件中生成的方法名
	  (JNIEnv * env, jobject obj){ 
		  return (*env)->NewStringUTF(env,"hello_from_c 2!" );

	}	
    ```
	然后就和上面的步骤一样了
 
	注意上面的这个javah的用法师在jdk1.6中用的，如果在jdk1.7中就不能这样用了
	对于jdk1.7在使用javah的工具的时候就不能够直接进入到classes目录下直接运行命令了，
	而是要将sdk中的platforms下的android版本中的android.jar这个路径加载到classPath的环境变量中(麻烦)，或者是直接进入到src目录下用javah包名.类名(简单常用)

8. 如何在c中向logcat中打印日志            
	如果想像logcat打印日志就要用到谷歌在ndk中提供的一个工具log.h的头文件
	步骤：
	1. 在c文件的头上面导入文件，加入下面的这四行代码      
        ```c
		#include <android/log.h> //导入log.h
		#define LOG_TAG "clog"  //指定打印到logcat的Tag
		#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__) //对后面的这个打印日志的方法起一个别名是LOGD
		#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
		```
	2. 在android.mk中加载文件            
        ```
		LOCAL_PATH := $(call my-dir)
		include $(CLEAR_VARS)

		LOCAL_MODULE    := Hello
		LOCAL_SRC_FILES := Hello.c
		LOCAL_LDLIBS += -llog   //新增加这一句，作用是 #把c语言调用的log函数对应的函数库加入到编译的运行时里面 #liblog.so，如果还要加载其他的就在后面继续 -lXXX
		include $(BUILD_SHARED_LIBRARY)
        ```
        
	3. 在c的代码中直接使用LOGD或者LOGI就能向logcat中输入打印信息
        ```c
		JNIEXPORT jint JNICALL Java_cn_itcast_ndk3_DataProvider_add
		(JNIEnv * env, jobject obj, jint x, jint y){
			LOGI( "x=%d",x);
			LOGD( "y=%d",y);
			int result = x+y;
			LOGD( "result=%d",result);
			return result;
		}
        ```

9. 如何将java的数据传递给c语言         
    就是java在方法中传值，然后c通过参数得到数据处理后返回和上面的一样

10. 将c中的字符串数组转成java中的string用到jni.h中的一个方法       
    `jstring (*NewStringUTF)(JNIEnv*, const char*);`

11. C中调用java
	c语言回调java的场景.
	1. 如果有一个操作已经有方便的java实现,才用c调用java可以避免重复发明一个轮子.
	2. 想在c代码里面通知界面更新ui.     
 
	C调用java的 思想类似于java中的反射，我们在c中就是通过反射的c实现来找到java中的这个方法，
    在getMethodID的第二个参数是一个方法的签名，这里我们可以通过jdk提供的一个工具javap，来到classes目录下，
	然后用 javap -s 类名.方法名  来得到一个方法的签名，这样就能列出来所有方法的签名

	```c
	/**
	 * env JNIEnv* java虚拟机环境的指针.
	 *
	 *jobject obj ,哪个对象调用的这个native的方法 , obj就代表的是哪个对象
	 */
	JNIEXPORT void JNICALL Java_cn_itcast_ndk4_DataProvider_callmethod1
	  (JNIEnv * env, jobject obj){
		   //思考 java中的反射
		   //1.找到某一个类的字节码
		   //   jclass      (*FindClass)(JNIEnv*, const char*);
		  jclass jclazz = (*env)->FindClass(env,"cn/itcast/ndk4/DataProvider" );
		   if(jclazz==0){
				LOGI( "LOAD CLAZZ ERROR");
		  } else{
				LOGI( "LOAD CLAZZ success" );
		  }
		   //2.找到类的字节码里面的方法.
		   // jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);

		  jmethodID  methodid = (*env)->GetMethodID(env,jclazz,"helloFromJava", "()V"); //最后一个参数是方法的签名
		   if(methodid==0){
				LOGI( "LOAD methodid ERROR" );
		  } else{
				LOGI( "LOAD methodid success" );
		  }

		   //3.调用方法
		   //void        (*CallVoidMethod)(JNIEnv*, jobject, jmethodID, ...);
		  (*env)->CallVoidMethod(env,obj,methodid);
	}
	```

12. 小知识      
	1. Android的API提供了SystemClock类，这个类中有一个方法         
		public static void sleep(long ms)，这个方法内部对Thread.sleep进行了封装对异常进行了try catch，平时用Thread.sleep还要自己进行捕捉，
		所以可以使用SystemColock.sleep()还简单
		
	2. Java中通过java虚拟机来调用c的代码，首先将c的库加载到虚拟机中，但是其实这个c代码并不是运行在java虚拟机中的，而是运行在虚拟机之外的一个单独的进程中
 
13. 自定义一个View控件(用于表示锅炉的压力大小)       
    1. 写一个类继承View(View是Android中所有能显示到界面上的东西全的父类)
    2. 重写onDraw()方法，这个方法是该控件被画到桌面上的时候调用的方法。
 
14. C++与C代码的不同           
    C++文件的后缀是cpp         
    C++与C的不同就是C++提供了模板、继承、抽象等     
	```c
	//将java字符串转成C++字符串的工具方法
	char*   Jstring2CStr(JNIEnv*   env,   jstring   jstr)
	{
		 char*   rtn   =   NULL;
		 jclass   clsstring   =   (env)->FindClass("java/lang/String");
		 jstring   strencode   =   (env)->NewStringUTF("GB2312");
		 jmethodID   mid   =   (env)->GetMethodID(clsstring,   "getBytes",   "(Ljava/lang/String;)[B");
		 jbyteArray   barr=   (jbyteArray)(env)->CallObjectMethod(jstr,mid,strencode); // String .getByte("GB2312");
		 jsize   alen   =   (env)->GetArrayLength(barr);
		 jbyte*   ba   =   (env)->GetByteArrayElements(barr,JNI_FALSE);
		 if(alen   >   0)
		 {
		  rtn   =   (char*)malloc(alen+1);         //"\0"
		  memcpy(rtn,ba,alen);
		  rtn[alen]=0;
		 }
		 (env)->ReleaseByteArrayElements(barr,ba,0);  //
		 return rtn;
	}
	JNIEXPORT jstring JNICALL Java_cn_itcast_cpp_DemoActivity_HelloFromC
	  (JNIEnv * env, jobject obj){
		//C代码
		//return (*env)->NewStringUTF(env,"haha from c"); 在C中env代表的是C中结构体的指针的指针
		//c++代码
		return env->NewStringUTF("haha from cpp");//在C++中env代表的是C++中结构体的指针
	}
	```
 
15. 对于JNI中的中文乱码问题        
    老版本的ndk r7之前 r6 r5 r5 crystal r4(编译的时候 语言集 是iso-8859-1)           
	在使用老版本ndk 编译出来的so文件的时候 要手动的进行转码.先用iso8859-1解码，再用utf-8编码，在r7(包括)之后的我们只要将C文件的格式改为UTF-8就可以了
 
16. 文件的格式及格式转换
    格式转换的原理：                
    1. 读取一段数据到内存     
    2. 分析这一段数据
    3. 修改里面的内容
    4. 写到文件里面
 
	文件的格式：        
	文件的存储方式是二进制0101这样        
	那么怎么设别文件的类型呢？     
	1. 根据扩展名
	2. 根据文件的头信息(头信息才是一个文件的真正的格式)，有些文件我们修改了扩展名也可以打开，
		这是因为打开文件的程序区扫描了文件的头信息，并用头信息中的类型来打开了这个文件

17. C中读取数据    
	```c
	#include<stdio.h>
	main(){
	//用  法: FILE *fopen(char *filename, char *type); //第二个参数是打开的方式 rt就是读文件， rb就是读二进制
	   FILE*    fp = fopen("1.txt","rt");
	//用  法: int fread (void *ptr, int size, int nitems, FILE *stream); 
							//ptr要读的数据 放在哪一块内存空间里面.
							//size 一次读的数据的长度.
							//nitems 读多少次
							//stream 从哪个文件里面 读
	 
	  char* buffer = malloc(sizeof(char)*12);
	 int len=   fread(buffer,sizeof(char),12,fp);
	printf("读了%d个char\n",len);
	   printf("str=%s\n",buffer);
	   fclose(fp);  //关闭掉流
	 
	  system("pause");     
	}
	```
	
18. C中写文件
	```c
	#include<stdio.h>
	main(){
	//用  法: FILE *fopen(char *filename, char *type);
	   FILE*    fp = fopen("1.txt","wt");
	//用  法: int fwrite(void *ptr, int size, int nitems, FILE *stream); 
							//ptr要向文件写的是哪一块内存里面的数据
							//size 一次写的数据的长度.
							//nitems 写多少次
							//stream 写到哪个文件里面
	   char* str="hello from c";
	   int len = fwrite(str,sizeof(char),12,fp);
	   printf("写了%d个char\n",len);
	   fclose(fp);  //关闭掉流
	 
	   system("pause");     
	}
	``` 
	
19. C语言文件操作模式
	“rt” 只读打开一个文本文件，只允许读数据       
	“wt” 只写打开或建立一个文本文件，只允许写数据        
	“at” 追加打开一个文本文件，并在文件末尾写数据      
	“rb” 只读打开一个二进制文件，只允许读数据      
	“wb” 只写打开或建立一个二进制文件，只允许写数据      
	“ab” 追加打开一个二进制文件，并在文件末尾写数据       
	“rt+” 读写打开一个文本文件，允许读和写        
	“wt+” 读写打开或建立一个文本文件，允许读写        
	“at+” 读写打开一个文本文件，允许读，或在文件末追加数据         
	“rb+” 读写打开一个二进制文件，允许读和写           
	“wb+” 读写打开或建立一个二进制文件，允许读和写            
	“ab+” 读写打开一个二进制文件，允许读，或在文件末追加数据        

	对于文件使用方式有以下几点说明： 
	文件使用方式由r,w,a,t,b，+六个字符拼成，各字符的含义是： 

	r(read): 读           
	w(write): 写         
	a(append): 追加               
	t(text): 文本文件，可省略不写        
	b(banary): 二进制文件        

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck!