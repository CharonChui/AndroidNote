Android入门介绍
===

1. 3G、4G
    - 第三代移动通信技术`(3rd - Generation)`，速率一般在几百`Kbps`，较之前的`2G`和`2.5G`在数据传输速度上有很大提升。
    - 第四代移动通信技术`(4th - Generation)`，速度可达到100`Mbps`以上，几乎可以满足人们的所有传输数据的需求。
    
    目前主流的`3G`技术标准有三种：
    - `WCDMA`：全球80%以上的`3G`网络都是采用此种制式。中国联通运营。186
    - `CDMA2000`：目前日韩及北美使用较多。中国电信运营。 189
    - `TD-SCDMA`：中国自主知识产权的3G通信技术。中国移动运营。 188 

    目前主流的`4G`技术为`LTE`，但还没有被广泛应用：    
    `GSM` → `GPRS` → `EDGE` → `WCDMA` → `HSDPA` → `HSDPA+` → `LTE`
    
2. Android是什么
    1. 手机设备的软件栈内存，包括
    	- 一个完整的操作系统
    	- 中间件
    	- 关键的应用程序
	
	2. 底层是Linux内核
    	- 安全管理
    	- 内存管理
    	- 进程管理
    	- 电源管理
    	- 硬件驱动

3. Android体系结构
    - Applications:桌面、电话、浏览器等应用程序
    - Applications Framework:ActivityManager、 WindowManager、ContentProvider、ResourceManager等     
    - Libraries: SQLite库、SurfaceManager、WebKit、OppenGL等。
        - Android运行时
            - Core Libraries
            - Dalvik Virtual Machine
    - Linux Kernel: 硬件驱动、电源管理等

4. Dalvik VM和JVM的区别
    1. 编译后文件的格式： 
        - JVM: .java->.class->.jar
        - Dalvik: .java->.class->.dex->.odex
    2. 基于的架构不同
        - JVM基于栈的架构(栈内存)
        - Dalvik基于寄存器的架构(CPU)，执行效率比JVM要高
    3. Dalvik专门针对移动平台进行优化     
        JVM的jar包中会有很多class文件，每个class文件中都含有头信息、常量池、字段、方法等，而apk中只有一个dex，它里面包括了所有头信息、常量池、方法等。这样读取一个文件要比读取多个文件去找块。  

5. CPU处理器架构
    1. x86
        - intel
        - AMD
    2. ARM
        - 联发科
        - 高通
        - 海思
        - 三星

6. Android项目目录结构
    1. src：源代码
    2. gen：系统自动生成的文件，R.java 中记录了项目中各种资源ID
    3. res：系统资源，所有文件都会在R文件生成资源ID
        - drawable：图片
        - layout：界面布局
        - values：数据
        - anim：定义动画的XML
        - raw：原生文件
    4. assets：资源路径，不会在R文件注册
    5. project.properties：供Eclipse使用，读取该项目使用Android版本号，早期版本名为default.properties
    6. AndroidManifest.xml：清单文件，在软件安装的时候被读取      
        Android中的四大组件（Activity、ContentProvider、BroadcastReceiver、Service）都需要在该文件中注册程序所需的权限也需要在此文件中声明，例如：电话、短信、互联网、访问SD卡
    7. bin：二进制文件，包括class、资源文件、dex、apk等
    8. proguard.cfg：用来混淆代码的配置文件，防止别人反编译

7. APK 安装过程
    1. Eclipse将.java源文件编译成.class
    2. 使用dx工具将所有.class文件转换为.dex文件
    3. 再将.dex文件和所有资源打包并且签名成.apk文件
    4. 将.apk文件安装到虚拟机完成程序安装
    5. 启动程序 – 开启进程 – 开启主线程
    6. 创建Activity对象 – 执行OnCreate()方法
    7. 按照main.xml文件初始化界面

    简单的来说软件的安装都是两个过程
    - 拷贝apk中的一些文件到系统的某个目录      
	    1. `/data/app/`目录下   
	    2. 创建一个文件夹 `/data/data/com.test.helloworld/`来保存数据  
    - 在系统的packages.xml文件(类似于Windows的注册表)中里面配置应用权限等一些信息.  `/data/system/packages.xml`
    
8. Android安全学    
    Android安全学中的一个重要的设计点是在默认情况下应用程序没有权限执行对其它应用程序、操作系统或用户有害的操作。
	这些操作包括读/写用户的隐私数据（例如联系人或e-mail），读/写其它应用程序的文件，执行网络访问，保持设备活动，等等。 
	所有牵扯到付费或者可能与用户隐私相关的操作都要申请权限。

9. 测试分类    
    单元测试(Unit test) -> 功能测试( Function test) -> 集成测试(Intergation test)
	
10. Android单元测试
    - AndroidManifest.xml中进行配置,导入android的junit环境
    - 编写测试类继承Android的测试父类,AndroidTestCase这个类( AndroidTestCase是为了去模拟一个手机的运行环境，这个类中有一个getContext方法能获取到当前测试类的应用上下文对象，所以这个方法必须要等到测试框架初始化完成后才可以去调用)
    - 测试的方法名要求以小写的test开头,如不以test开头只能单独点这个方法运行,整体全部运行时没有这个方法，所有的测试方法都要抛出异常，要把异常抛给测试框架不能自己去捕获
 
    注意:测试的代码也是只能在手机上跑,它是在手机上测试完之后又将信息发送到了eclipse中


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
