Android学习笔记  
===

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/note.jpg)

> 十年生死两茫茫，不思量，自难忘，华年短暂，陈辞岁月悠悠伤，        
> 满腔热血已芜荒，展未来，后生强，战战兢兢，如履薄冰心彷徨，            
> 青丝化雪、鬓角成霜，已是英雄迟暮，人生怎慷慨激昂？


目录
===  

- [史上最适合Android开发者学习的Harmony OS Next语言教程](https://github.com/CharonChui/HarmonyOSNextStudyNote)

- [源码解析][43] 
    - [自定义View详解][1]
    - [Activity界面制过程详解][2]
    - [Activity启动过程][3]
    - [Android Touch事件分发详解][4]
    - [AsyncTask详解][5]
    - [butterknife源码详解][6]
    - [InstantRun详解][7]
    - [ListView源码分析][8]
    - [VideoView源码分析][9]
    - [View绘制过程详解][10]
    - [LeakCanary源码分析][284]
    - [网络部分][11]
        - [HttpURLConnection详解][12]
        - [HttpURLConnection与HttpClient][13]
        - [volley-retrofit-okhttp之我们该如何选择网路框架][14]
        - [Volley源码分析][15]
        - [Retrofit详解(上)][16]
        - [Retrofit详解(下)][17]
- [Dagger2][199]        
    - [1.Dagger2简介(一).md][200]
    - [2.Dagger2入门demo(二).md][201]    
    - [3.Dagger2入门demo扩展(三).md][202]
    - [4.Dagger2单例(四).md][203]
    - [5.Dagger2Lay和Provider(五).md][204]
    - [6.Dagger2Android示例代码(六).md][205]
    - [7.Dagger2之dagger-android(七).md][206]            
    - [8.Dagger2与MVP(八).md][207]    
    - [9.Dagger2原理分析(九).md][212]
- [音视频开发][44]
    - [搭建nginx+rtmp服务器][18]
    - [视频播放相关内容总结][19]
    - [Android音视频开发][21]
        - [Android WebRTC简介][22]    
        - [DLNA简介][24]
        - [AudioTrack简介][214]
        - [播放器性能优化][230]
        - [MediaExtractor、MediaCodec、MediaMuxer][245]
        - [SurfaceView与TextureView][226]
        - [视频解码之软解与硬解][20]
        - [音视频同步原理][326]
        - [音视频场景][327]
        - [1.音视频基础知识][328]
        - [2.系统播放器MediaPlayer][329]
        - [11.播放器组件封装][330]
        - [MediaMetadataRetriever][344]
    - [DNS及HTTPDNS][23]
    - [流媒体协议][224]
        - [流媒体协议][246]
        - [HLS][247]
        - [DASH][248]
        - [HTTP FLV][249]
        - [RTMP][250]
    - [ExoPlayer][216]
        - [1. ExoPlayer简介.md][217]
        - [2. ExoPlayer MediaSource简介][218]
        - [3. ExoPlayer源码分析之prepare方法][219]
        - [4. ExoPlayer源码分析之prepare序列图][220]
        - [5. ExoPlayer源码分析之PlayerView][221]
    - [视频封装格式][225]
        - [MP4格式详解][251]
        - [FLV][252]
        - [TS][253]
        - [fMP4 vs ts][254]
        - [fMP4格式详解][255]
        - [视频封装格式][256]
        - [M3U8][321]
        - [AVI][341]
    - [视频编码][257]
        - [视频编码原理][331]
        - [AV1][258]
        - [H264][259]
        - [H265][260]
    - [音频编码][335]    
        - [音频编码格式][336]
        - [AAC][337]
        - [PCM][338]
        - [WAV][339]
    - [关键帧][227]
    - [CDN及PCDN][228]
    - [P2P技术][229]
        - [P2P][261]
        - [P2P原理_NAT穿透][262]
    - [OpenGL][231]
        - [1.OpenGL简介][232]
        - [2.GLSurfaceView简介][233]
        - [3.GLSurfaceView源码解析][234]
        - [4.GLTextureView实现][235]
        - [5.OpenGL ES绘制三角形][236]
        - [6.OpenGL ES绘制矩形及圆形][237]
        - [7.OpenGL ES着色器语言GLSL][238]
        - [8.GLES类及Matrix类][239]
        - [9.OpenGL ES纹理][240]
        - [10.GLSurfaceView+MediaPlayer播放视频][241]
        - [11.OpenGL ES滤镜][242]
        - [12.FBO][332]
    - [弹幕][243]
        - [Android弹幕实现][244]
    - [FFmpeg][322]    
        - [1.FFmpeg简介][323]
        - [2.FFmpeg常用命令行][324]
        - [3.FFmpeg切片][325]
        - [4.开发环境配置][333]
        - [5. FFmpeg核心功能][334]
        - [6.视频播放简介][340]
    - [OpenCV][342]
        - [1.OpenCV简介][343]

- [操作系统][263]
    - [1.操作系统简介][264]
    - [2.进程与线程][265]
    - [3.内存管理][266]
    - [4.调度][267]
    - [5.I/O][268]
    - [6.文件管理][269]
    - [7.嵌入式系统][270]
    - [8.虚拟机][271]
        - [Android内核][274]
            - [1.Android进程间通信][275]
            - [2.Android线程间通信之Handler消息机制][276]
            - [3.Android Framework框架][277]
            - [4.ActivityManagerService简介][278]
            - [5.Android消息获取][279]
            - [6.屏幕绘制基础][280]
            - [7.View绘制原理][281]
            - [8.WindowManagerService简介][282]
            - [9.PackageManagerService简介][283]
- [架构设计][272]
    - [1.架构简介][273]

- [Jetpack][287]
    - [Jetpack简介][288]
    - [architecture][289]
        - [1.简介][293]
        - [2.ViewBinding简介][294]
        - [3.Lifecycle简介][295]
        - [4.ViewModel简介][296]
        - [5.LiveData简介][297]
        - [6.DataBinding简介][298]
        - [7.Room简介][299]
        - [8.PagingLibrary简介][300]
        - [9.App Startup简介][301]
        - [10.DataStore简介][302]
        - [11.Hilt简介][303]
        - [12.Navigation简介][304]
        - [13.Jetpack MVVM简介][305]
        - [14.findViewById的过去及未来][306]
    - [ui][290]
        - [Jetpack Compose简介][307]
        - [material][308]
            - [1.MaterialToolbar简介][309]
            - [2.NavigationView简介][310]
            - [3.NestedScrollView简介][311]
            - [4.CoordinatorLayout简介][312]
            - [5.AppBarLayout简介][313]
            - [6.CollapsingToolbarLayout简介][314]
            - [7.Snackbar简介][315]
            - [8.TabLayout简介][316]
    - [foundation][291]
        - [1.简介][317]
    - [behavior][292]
        - [1.简介][318]

-  [图片加载][45]
    - [Glide简介(上)][25]
    - [Glide简介(下)][26]
    - [图片加载库比较][27]
    - [Coil简介][320]


- [RxJava][46]
    - [RxJava详解(一)][28]
    - [RxJava详解(二)][29]
    - [RxJava详解(三)][30]
    - [RxJava详解之执行原理(四)][209]
    - [RxJava详解之操作符执行原理(五)][210]
    - [RxJava详解之线程调度原理(六)][211]
    - [RxJava系列全家桶][31]

- [开发工具][47]
    - [目前流行的开发组合][32]
    - [性能优化相关工具][33]
    - [Android开发工具及类库][34]
    - [Github个人主页绑定域名][35]
    - [Markdown学习手册][36]
    - [MAT内存分析][37]
    - [调试平台Sonar][213]
    - [Icon制作][223]

- [Kotlin学习][48]
    - [1.Kotlin_简介&变量&类&接口][180]
    - [2.Kotlin_高阶函数&Lambda&内联函数][181]
    - [3.Kotlin_数字&字符串&数组&集合][182]
    - [4.Kotlin_表达式&关键字][183]
    - [5.Kotlin_内部类&密封类&枚举&委托][184]
    - [6.Kotlin_多继承问题][185]
    - [7.Kotlin_注解&反射&扩展][186]
    - [8.Kotlin_协程][187]
    - [9.Kotlin_androidktx][188]
    - [10.Kotlin_设计模式][197]



- [Gradle&Maven][49]
    - [Gradle专题][39]
    - [发布library到Maven仓库][40]
    - [Composing builds简介][319]

- [应用发布][50]
    - [使用Jenkins实现自动化打包][198]
    - [Android应用发布][41]
    - [Zipalign优化][42]

- [Android Studio使用教程][51]
    - [AndroidStudio使用教程(第一弹)][55]
    - [AndroidStudio使用教程(第二弹)][56]    
    - [AndroidStudio使用教程(第三弹)][57]
    - [AndroidStudio使用教程(第四弹)][58]
    - [AndroidStudio使用教程(第五弹)][59]
    - [AndroidStudio使用教程(第六弹)][60]
    - [AndroidStudio使用教程(第七弹)][61]   
    - [Android Studio你可能不知道的操作][62]
    - [AndroidStudio提高Build速度][63]
    - [AndroidStudio中进行ndk开发][64]

- [进阶部分][52]  
    - [布局优化][65]
    - [屏幕适配之百分比方案详解][66]
    - [热修复实现][67]
    - [如何让Service常驻内存][68]
    - [通过Hardware Layer提高动画性能][69]
    - [性能优化][70]
    - [注解使用][71]
    - [Android6.0权限系统][72]
    - [Android开发不申请权限来使用对应功能][73]
    - [Android开发中的MVP模式详解][74]
    - [Android启动模式详解][75]
    - [Android卸载反馈][76]
    - [ApplicationId vs PackageName][77]
    - [AndroidRuntime_ART与Dalvik][78]
    - [BroadcastReceiver安全问题][79]
    - [Crash及ANR分析][80]
    - [Library项目中资源id使用case时报错][81]
    - [Mac下配置adb及Android命令][82]
    - [RecyclerView专题][84]
    - [ConstraintLaayout简介][194]
    - [Android WorkManager][208]
    - [OOM问题分析][215]

- [Java基础及算法][53]
    - [数据结构和算法][192]
	- [八种排序算法][189]
	- [线程池的原理][190]
	- [设计模式][191]
    - [动态代理][193]
    - [常用命令行大全][85]
    - [单例的最佳实现方式][86]
    - [数据结构][87]
    - [获取今后多少天后的日期][88]
    - [剑指Offer(上)][89]
    - [剑指Offer(下)][90]
    - [强引用、软引用、弱引用、虚引用][91]
    - [生产者消费者][92]
    - [数据加密及解密][93]
    - [死锁][94]
    - [算法][95]
    - [网络请求相关内容总结][96]
    - [线程池的原理][97]
    - [Java并发编程之原子性、可见性以及有序性][98]
    - [Base64加密][99]
    - [Git简介][100]
    - [hashCode与equals][101]
    - [HashMap实现原理分析][102]
    - [Java基础面试题][103]
    - [JVM垃圾回收机制][104]
    - [MD5加密][105]
    - [MVC与MVP及MVVM][106]
    - [RMB大小写转换][107]
    - [Vim使用教程][108]
    - [volatile和Synchronized区别][109]
    - [Http与Https的区别][195]
    - [Top-K问题][196]
    - [Java内存模型][285]
    - [JVM架构][286]
- [基础部分][54] 
    - [安全退出应用程序][110]
    - [病毒][111]
    - [超级管理员(DevicePoliceManager)][112]
    - [程序的启动、卸载和分享][113]
    - [代码混淆][114]
    - [读取用户logcat日志][115]
    - [短信广播接收者][116]
    - [多线程断点下载][117]
    - [黑名单挂断电话及删除电话记录][118]
    - [横向ListView][119]
    - [滑动切换Activity(GestureDetector)][120]
    - [获取联系人][121]
    - [获取手机及SD卡可用存储空间][122]
    - [获取手机中所有安装的程序][123]
    - [获取位置(LocationManager)][124]
    - [获取应用程序缓存及一键清理][125]
    - [开发中异常的处理][126]
    - [开发中Log的管理][127]
    - [快捷方式工具类][128]
    - [来电号码归属地提示框][129]
    - [来电监听及录音][130]
    - [零权限上传数据][131]
    - [内存泄漏][132]
    - [屏幕适配][133]
    - [任务管理器(ActivityManager)][134]
    - [手机摇晃][135]
    - [竖着的Seekbar][136]
    - [数据存储][137]
    - [搜索框][138]
    - [锁屏以及解锁监听][139]
    - [文件上传][140]
    - [下拉刷新ListView][141]
    - [修改系统组件样式][142]
    - [音量及屏幕亮度调节][143]
    - [应用安装][144]
    - [应用后台唤醒后数据的刷新][145]
    - [知识大杂烩][146]
    - [资源文件拷贝的三种方式][147]
    - [自定义背景][148]
    - [自定义控件][149]
    - [自定义状态栏通知][150]
    - [自定义Toast][151]
    - [adb logcat使用简介][152]
    - [Android编码规范][153]
    - [Android动画][154]
    - [Android基础面试题][155]
    - [Android入门介绍][156]
    - [Android四大组件之ContentProvider][157]
    - [Android四大组件之Service][158]
    - [Ant打包][159]
    - [Bitmap优化][160]
    - [Fragment专题][161]
    - [Home键监听][162]
    - [HttpClient执行Get和Post请求][163]
    - [JNI_C语言基础][164]
    - [JNI基础][165]
    - [ListView专题][166]
    - [Parcelable及Serializable][167]
    - [PopupWindow细节][168]
    - [Scroller简介][169]
    - [ScrollingTabs][170]
    - [SDK Manager无法更新的问题][171]
    - [Selector使用][172]
    - [SlidingMenu][173]
    - [String格式化][174]
    - [TextView跑马灯效果][175]
    - [WebView总结][176]
    - [Widget(窗口小部件)][177]
    - [Wifi状态监听][178]
    - [XmlPullParser][179]
    - [反编译][222]
    

[1]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/%E8%87%AA%E5%AE%9A%E4%B9%89View%E8%AF%A6%E8%A7%A3.md        "自定义View详解"
[2]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Activity%E7%95%8C%E9%9D%A2%E7%BB%98%E5%88%B6%E8%BF%87%E7%A8%8B%E8%AF%A6%E8%A7%A3.md  "Activity界面绘制过程详解"
[3]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Activity%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.md    "Activity启动过程"
[4]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Android%20Touch%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E8%AF%A6%E8%A7%A3.md    "Android Touch事件分发详解"
[5]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/AsyncTask%E8%AF%A6%E8%A7%A3.md   "AsyncTask详解"
[6]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/butterknife%E6%BA%90%E7%A0%81%E8%AF%A6%E8%A7%A3.md   "butterknife源码详解"
[7]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/InstantRun%E8%AF%A6%E8%A7%A3.md   "InstantRun详解"
[8]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/ListView源码分析.md   "ListView源码分析"
[9]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/VideoView%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md   "VideoView源码分析"
[10]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/View%E7%BB%98%E5%88%B6%E8%BF%87%E7%A8%8B%E8%AF%A6%E8%A7%A3.md   "View绘制过程详解"
[11]: https://github.com/CharonChui/AndroidNote/tree/master/SourceAnalysis/Netowork   "网络部分"
[12]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/HttpURLConnection%E8%AF%A6%E8%A7%A3.md   "HttpURLConnection详解"
[13]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/HttpURLConnection%E4%B8%8EHttpClient.md   "HttpURLConnection与HttpClient"
[14]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/volley-retrofit-okhttp%E4%B9%8B%E6%88%91%E4%BB%AC%E8%AF%A5%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9%E7%BD%91%E8%B7%AF%E6%A1%86%E6%9E%B6.md   "volley-retrofit-okhttp之我们该如何选择网路框架"
[15]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/Volley%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md   "Volley源码分析"
[16]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/Retrofit%E8%AF%A6%E8%A7%A3(%E4%B8%8A).md   "Retrofit详解(上)"
[17]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Netowork/Retrofit%E8%AF%A6%E8%A7%A3(%E4%B8%8B).md   "Retrofit详解(下)"
[18]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%90%AD%E5%BB%BAnginx%2Brtmp%E6%9C%8D%E5%8A%A1%E5%99%A8.md   "搭建nginx+rtmp服务器"
[19]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E7%9B%B8%E5%85%B3%E5%86%85%E5%AE%B9%E6%80%BB%E7%BB%93.md   "视频播放相关内容总结"
[20]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/%E8%A7%86%E9%A2%91%E8%A7%A3%E7%A0%81%E4%B9%8B%E8%BD%AF%E8%A7%A3%E4%B8%8E%E7%A1%AC%E8%A7%A3.md   "视频解码之软解与硬解"
[21]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91   "Android音视频开发"
[22]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/Android%20WebRTC%E7%AE%80%E4%BB%8B.md   "Android WebRTC简介"
[23]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/DNS%E5%8F%8AHTTPDNS.md   "DNS及HTTPDNS"
[24]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/DLNA%E7%AE%80%E4%BB%8B.md   "DLNA简介"
[25]: https://github.com/CharonChui/AndroidNote/blob/master/ImageLoaderLibrary/Glide%E7%AE%80%E4%BB%8B(%E4%B8%8A).md   "Glide简介(上)"
[26]: https://github.com/CharonChui/AndroidNote/blob/master/ImageLoaderLibrary/Glide%E7%AE%80%E4%BB%8B(%E4%B8%8B).md   "Glide简介(下)"
[27]: https://github.com/CharonChui/AndroidNote/blob/master/ImageLoaderLibrary/%E5%9B%BE%E7%89%87%E5%8A%A0%E8%BD%BD%E5%BA%93%E6%AF%94%E8%BE%83.md   "图片加载库比较"
[28]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/1.RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%80).md   "RxJava详解(一)"
[29]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/2.RxJava%E8%AF%A6%E8%A7%A3(%E4%BA%8C).md   "RxJava详解(二)"
[30]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/3.RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%89).md   "RxJava详解(三)"
[31]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/7.RxJava%E7%B3%BB%E5%88%97%E5%85%A8%E5%AE%B6%E6%A1%B6.md   "RxJava系列全家桶"
[32]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/%E7%9B%AE%E5%89%8D%E6%B5%81%E8%A1%8C%E7%9A%84%E5%BC%80%E5%8F%91%E7%BB%84%E5%90%88.md   "目前流行的开发组合"
[33]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%9B%B8%E5%85%B3%E5%B7%A5%E5%85%B7.md   "性能优化相关工具"
[34]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/Android%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7%E5%8F%8A%E7%B1%BB%E5%BA%93.md   "Android开发工具及类库"
[35]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/Github%E4%B8%AA%E4%BA%BA%E4%B8%BB%E9%A1%B5%E7%BB%91%E5%AE%9A%E5%9F%9F%E5%90%8D.md   "Github个人主页绑定域名"
[36]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/Markdown%E5%AD%A6%E4%B9%A0%E6%89%8B%E5%86%8C.md   "Markdown学习手册"
[37]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/MAT%E5%86%85%E5%AD%98%E5%88%86%E6%9E%90.md   "MAT内存分析"
[38]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/Kotlin%E5%AD%A6%E4%B9%A0%E6%95%99%E7%A8%8B(%E4%B8%80).md   "Kotlin学习教程(一)(未完)"
[39]: https://github.com/CharonChui/AndroidNote/blob/master/Gradle%26Maven/Gradle%E4%B8%93%E9%A2%98.md   "Gradle专题"
[40]: https://github.com/CharonChui/AndroidNote/blob/master/Gradle%26Maven/%E5%8F%91%E5%B8%83library%E5%88%B0Maven%E4%BB%93%E5%BA%93.md   "发布library到Maven仓库"
[41]: https://github.com/CharonChui/AndroidNote/blob/master/AppPublish/Android%E5%BA%94%E7%94%A8%E5%8F%91%E5%B8%83.md   "Android应用发布"
[42]: https://github.com/CharonChui/AndroidNote/blob/master/AppPublish/Zipalign%E4%BC%98%E5%8C%96.md   "Zipalign优化"
[43]: https://github.com/CharonChui/AndroidNote/tree/master/SourceAnalysis   "源码解析"
[44]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment   "音视频开发"
[45]: https://github.com/CharonChui/AndroidNote/tree/master/ImageLoaderLibrary   "图片加载"
[46]: https://github.com/CharonChui/AndroidNote/tree/master/RxJavaPart   "RxJava"
[47]: https://github.com/CharonChui/AndroidNote/tree/master/Tools%26Library   "开发工具"
[48]: https://github.com/CharonChui/AndroidNote/tree/master/KotlinCourse   "Kotlin学习"
[49]: https://github.com/CharonChui/AndroidNote/tree/master/Gradle%26Maven   "Gradle&Maven"
[50]: https://github.com/CharonChui/AndroidNote/tree/master/AppPublish   "应用发布"
[51]: https://github.com/CharonChui/AndroidNote/tree/master/AndroidStudioCourse   "Android Studio使用教程"
[52]: https://github.com/CharonChui/AndroidNote/tree/master/AdavancedPart   "进阶部分"
[53]: https://github.com/CharonChui/AndroidNote/tree/master/JavaKnowledge   "Java基础及算法"
[54]: https://github.com/CharonChui/AndroidNote/tree/master/BasicKnowledge   "基础部分"
[55]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%B8%80%E5%BC%B9).md   "AndroidStudio使用教程(第一弹)"
[56]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%BA%8C%E5%BC%B9).md   "AndroidStudio使用教程(第二弹)"
[57]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%B8%89%E5%BC%B9).md   "AndroidStudio使用教程(第三弹)"
[58]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E5%9B%9B%E5%BC%B9).md   "AndroidStudio使用教程(第四弹)"
[59]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%BA%94%E5%BC%B9).md   "AndroidStudio使用教程(第五弹)"
[60]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E5%85%AD%E5%BC%B9).md   "AndroidStudio使用教程(第六弹)"
[61]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(%E7%AC%AC%E4%B8%83%E5%BC%B9).md   "AndroidStudio使用教程(第七弹)"
[62]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/Android%20Studio%E4%BD%A0%E5%8F%AF%E8%83%BD%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84%E6%93%8D%E4%BD%9C.md   "Android Studio你可能不知道的操作"
[63]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E6%8F%90%E9%AB%98Build%E9%80%9F%E5%BA%A6.md   "AndroidStudio提高Build速度"
[64]: https://github.com/CharonChui/AndroidNote/blob/master/AndroidStudioCourse/AndroidStudio%E4%B8%AD%E8%BF%9B%E8%A1%8Cndk%E5%BC%80%E5%8F%91.md   "AndroidStudio中进行ndk开发"
[65]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E5%B8%83%E5%B1%80%E4%BC%98%E5%8C%96.md   "布局优化"
[66]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E5%B1%8F%E5%B9%95%E9%80%82%E9%85%8D%E4%B9%8B%E7%99%BE%E5%88%86%E6%AF%94%E6%96%B9%E6%A1%88%E8%AF%A6%E8%A7%A3.md   "屏幕适配之百分比方案详解"
[67]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E7%83%AD%E4%BF%AE%E5%A4%8D%E5%AE%9E%E7%8E%B0.md   "热修复实现"
[68]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E5%A6%82%E4%BD%95%E8%AE%A9Service%E5%B8%B8%E9%A9%BB%E5%86%85%E5%AD%98.md   "如何让Service常驻内存"
[69]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E9%80%9A%E8%BF%87Hardware%20Layer%E6%8F%90%E9%AB%98%E5%8A%A8%E7%94%BB%E6%80%A7%E8%83%BD.md   "通过Hardware Layer提高动画性能"
[70]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md   "性能优化"
[71]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/%E6%B3%A8%E8%A7%A3%E4%BD%BF%E7%94%A8.md   "注解使用"
[72]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android6.0%E6%9D%83%E9%99%90%E7%B3%BB%E7%BB%9F.md   "Android6.0权限系统"
[73]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%E5%BC%80%E5%8F%91%E4%B8%8D%E7%94%B3%E8%AF%B7%E6%9D%83%E9%99%90%E6%9D%A5%E4%BD%BF%E7%94%A8%E5%AF%B9%E5%BA%94%E5%8A%9F%E8%83%BD.md   "Android开发不申请权限来使用对应功能"
[74]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%E5%BC%80%E5%8F%91%E4%B8%AD%E7%9A%84MVP%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3.md   "Android开发中的MVP模式详解"
[75]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3.md   "Android启动模式详解"
[76]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%E5%8D%B8%E8%BD%BD%E5%8F%8D%E9%A6%88.md   "Android卸载反馈"
[77]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/ApplicationId%20vs%20PackageName.md   "ApplicationId vs PackageName"
[78]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/AndroidRuntime_ART%E4%B8%8EDalvik.md   "AndroidRuntime_ART与Dalvik"
[79]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/BroadcastReceiver%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98.md   "BroadcastReceiver安全问题"
[80]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Crash%E5%8F%8AANR%E5%88%86%E6%9E%90.md  "Crash及ANR分析"

[81]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Library%E9%A1%B9%E7%9B%AE%E4%B8%AD%E8%B5%84%E6%BA%90id%E4%BD%BF%E7%94%A8case%E6%97%B6%E6%8A%A5%E9%94%99.md   "Library项目中资源id使用case时报错"
[82]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Mac%E4%B8%8B%E9%85%8D%E7%BD%AEadb%E5%8F%8AAndroid%E5%91%BD%E4%BB%A4.md   "Mac下配置adb及Android命令"
[84]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/RecyclerView%E4%B8%93%E9%A2%98.md   "RecyclerView专题"
[85]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%A4%A7%E5%85%A8.md   "常用命令行大全"
[86]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%8D%95%E4%BE%8B%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F.md   "单例的最佳实现方式"
[87]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E7%AE%97%E6%B3%95/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.md   "数据结构"
[88]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E8%8E%B7%E5%8F%96%E4%BB%8A%E5%90%8E%E5%A4%9A%E5%B0%91%E5%A4%A9%E5%90%8E%E7%9A%84%E6%97%A5%E6%9C%9F.md   "获取今后多少天后的日期"
[89]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%89%91%E6%8C%87Offer(%E4%B8%8A).md   "剑指Offer(上)"
[90]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/剑指Offer(下).md   "剑指Offer(下)"
[91]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%BC%BA%E5%BC%95%E7%94%A8%E3%80%81%E8%BD%AF%E5%BC%95%E7%94%A8%E3%80%81%E5%BC%B1%E5%BC%95%E7%94%A8%E3%80%81%E8%99%9A%E5%BC%95%E7%94%A8.md   "强引用、软引用、弱引用、虚引用"
[92]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85.md   "生产者消费者"
[93]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E6%95%B0%E6%8D%AE%E5%8A%A0%E5%AF%86%E5%8F%8A%E8%A7%A3%E5%AF%86.md   "数据加密及解密"
[94]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E6%AD%BB%E9%94%81.md   "死锁"
[95]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E7%AE%97%E6%B3%95/%E7%AE%97%E6%B3%95.md   "算法"
[96]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E7%9B%B8%E5%85%B3%E5%86%85%E5%AE%B9%E6%80%BB%E7%BB%93.md   "网络请求相关内容总结"
[97]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E5%8E%9F%E7%90%86.md   "线程池的原理"
[98]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Java并发编程之原子性、可见性以及有序性.md  "Java并发编程之原子性、可见性以及有序性"
[99]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Base64%E5%8A%A0%E5%AF%86.md   "Base64加密"
[100]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Git%E7%AE%80%E4%BB%8B.md   "Git简介"
[101]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/hashCode%E4%B8%8Eequals.md   "hashCode与equals"
[102]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/HashMap%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md   "HashMap实现原理分析"
[103]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Java%E5%9F%BA%E7%A1%80%E9%9D%A2%E8%AF%95%E9%A2%98.md   "Java基础面试题"
[104]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/JVM%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6.md   "JVM垃圾回收机制"
[105]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/MD5%E5%8A%A0%E5%AF%86.md   "MD5加密"
[106]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/MVC%E4%B8%8EMVP%E5%8F%8AMVVM.md   "MVC与MVP及MVVM"
[107]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/RMB%E5%A4%A7%E5%B0%8F%E5%86%99%E8%BD%AC%E6%8D%A2.md   "RMB大小写转换"
[108]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Vim%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B.md   "Vim使用教程"
[109]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/volatile%E5%92%8CSynchronized%E5%8C%BA%E5%88%AB.md   "volatile和Synchronized区别"
[110]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%AE%89%E5%85%A8%E9%80%80%E5%87%BA%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F.md   "安全退出应用程序"
[111]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E7%97%85%E6%AF%92.md   "病毒"
[112]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%B6%85%E7%BA%A7%E7%AE%A1%E7%90%86%E5%91%98(DevicePoliceManager).md   "超级管理员(DevicePoliceManager)"
[113]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%90%AF%E5%8A%A8%E3%80%81%E5%8D%B8%E8%BD%BD%E5%92%8C%E5%88%86%E4%BA%AB.md   "程序的启动、卸载和分享"
[114]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E4%BB%A3%E7%A0%81%E6%B7%B7%E6%B7%86.md   "代码混淆"
[115]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%AF%BB%E5%8F%96%E7%94%A8%E6%88%B7logcat%E6%97%A5%E5%BF%97.md   "读取用户logcat日志"
[116]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E7%9F%AD%E4%BF%A1%E5%B9%BF%E6%92%AD%E6%8E%A5%E6%94%B6%E8%80%85.md   "短信广播接收者"
[117]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%96%AD%E7%82%B9%E4%B8%8B%E8%BD%BD.md   "多线程断点下载"
[118]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E9%BB%91%E5%90%8D%E5%8D%95%E6%8C%82%E6%96%AD%E7%94%B5%E8%AF%9D%E5%8F%8A%E5%88%A0%E9%99%A4%E7%94%B5%E8%AF%9D%E8%AE%B0%E5%BD%95.md   "黑名单挂断电话及删除电话记录"
[119]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%A8%AA%E5%90%91ListView.md   "横向ListView"
[120]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%BB%91%E5%8A%A8%E5%88%87%E6%8D%A2Activity(GestureDetector).md   "滑动切换Activity(GestureDetector)"
[121]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%8E%B7%E5%8F%96%E8%81%94%E7%B3%BB%E4%BA%BA.md   "获取联系人"
[122]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%8E%B7%E5%8F%96%E6%89%8B%E6%9C%BA%E5%8F%8ASD%E5%8D%A1%E5%8F%AF%E7%94%A8%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4.md   "获取手机及SD卡可用存储空间"
[123]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%8E%B7%E5%8F%96%E6%89%8B%E6%9C%BA%E4%B8%AD%E6%89%80%E6%9C%89%E5%AE%89%E8%A3%85%E7%9A%84%E7%A8%8B%E5%BA%8F.md   "获取手机中所有安装的程序"
[124]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%8E%B7%E5%8F%96%E4%BD%8D%E7%BD%AE(LocationManager).md   "获取位置(LocationManager)"
[125]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%8E%B7%E5%8F%96%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E7%BC%93%E5%AD%98%E5%8F%8A%E4%B8%80%E9%94%AE%E6%B8%85%E7%90%86.md   "获取应用程序缓存及一键清理"
[126]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%BC%80%E5%8F%91%E4%B8%AD%E5%BC%82%E5%B8%B8%E7%9A%84%E5%A4%84%E7%90%86.md   "开发中异常的处理"
[127]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%BC%80%E5%8F%91%E4%B8%ADLog%E7%9A%84%E7%AE%A1%E7%90%86.md   "开发中Log的管理"
[128]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F%E5%B7%A5%E5%85%B7%E7%B1%BB.md   "快捷方式工具类"
[129]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%9D%A5%E7%94%B5%E5%8F%B7%E7%A0%81%E5%BD%92%E5%B1%9E%E5%9C%B0%E6%8F%90%E7%A4%BA%E6%A1%86.md   "来电号码归属地提示框"
[130]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%9D%A5%E7%94%B5%E7%9B%91%E5%90%AC%E5%8F%8A%E5%BD%95%E9%9F%B3.md   "来电监听及录音"
[131]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E9%9B%B6%E6%9D%83%E9%99%90%E4%B8%8A%E4%BC%A0%E6%95%B0%E6%8D%AE.md   "零权限上传数据"
[132]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F.md   "内存泄漏"
[133]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%B1%8F%E5%B9%95%E9%80%82%E9%85%8D.md   "屏幕适配"
[134]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E4%BB%BB%E5%8A%A1%E7%AE%A1%E7%90%86%E5%99%A8(ActivityManager).md   "任务管理器(ActivityManager)"
[135]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%89%8B%E6%9C%BA%E6%91%87%E6%99%83.md   "手机摇晃"
[136]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E7%AB%96%E7%9D%80%E7%9A%84Seekbar.md   "竖着的Seekbar"
[137]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8.md   "数据存储"
[138]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%90%9C%E7%B4%A2%E6%A1%86.md   "搜索框"
[139]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E9%94%81%E5%B1%8F%E4%BB%A5%E5%8F%8A%E8%A7%A3%E9%94%81%E7%9B%91%E5%90%AC.md   "锁屏以及解锁监听"
[140]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0.md   "文件上传"
[141]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E4%B8%8B%E6%8B%89%E5%88%B7%E6%96%B0ListView.md   "下拉刷新ListView"
[142]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E4%BF%AE%E6%94%B9%E7%B3%BB%E7%BB%9F%E7%BB%84%E4%BB%B6%E6%A0%B7%E5%BC%8F.md   "修改系统组件样式"
[143]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E9%9F%B3%E9%87%8F%E5%8F%8A%E5%B1%8F%E5%B9%95%E4%BA%AE%E5%BA%A6%E8%B0%83%E8%8A%82.md   "音量及屏幕亮度调节"
[144]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%BA%94%E7%94%A8%E5%AE%89%E8%A3%85.md   "应用安装"
[145]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%BA%94%E7%94%A8%E5%90%8E%E5%8F%B0%E5%94%A4%E9%86%92%E5%90%8E%E6%95%B0%E6%8D%AE%E7%9A%84%E5%88%B7%E6%96%B0.md   "应用后台唤醒后数据的刷新"
[146]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E7%9F%A5%E8%AF%86%E5%A4%A7%E6%9D%82%E7%83%A9.md   "知识大杂烩"
[147]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%E6%8B%B7%E8%B4%9D%E7%9A%84%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F.md   "资源文件拷贝的三种方式"
[148]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%87%AA%E5%AE%9A%E4%B9%89%E8%83%8C%E6%99%AF.md   "自定义背景"
[149]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E4%BB%B6.md   "自定义控件"
[150]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%87%AA%E5%AE%9A%E4%B9%89%E7%8A%B6%E6%80%81%E6%A0%8F%E9%80%9A%E7%9F%A5.md   "自定义状态栏通知"
[151]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E8%87%AA%E5%AE%9A%E4%B9%89Toast.md   "自定义Toast"
[152]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/adb%20logcat%E4%BD%BF%E7%94%A8%E7%AE%80%E4%BB%8B.md   "adb logcat使用简介"
[153]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E7%BC%96%E7%A0%81%E8%A7%84%E8%8C%83.md   "Android编码规范"
[154]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E5%8A%A8%E7%94%BB.md   "Android动画"
[155]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E5%9F%BA%E7%A1%80%E9%9D%A2%E8%AF%95%E9%A2%98.md   "Android基础面试题"
[156]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E5%85%A5%E9%97%A8%E4%BB%8B%E7%BB%8D.md   "Android入门介绍"
[157]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6%E4%B9%8BContentProvider.md   "Android四大组件之ContentProvider"
[158]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Android%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6%E4%B9%8BService.md   "Android四大组件之Service"
[159]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Ant%E6%89%93%E5%8C%85.md   "Ant打包"
[160]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Bitmap%E4%BC%98%E5%8C%96.md   "Bitmap优化"
[161]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Fragment%E4%B8%93%E9%A2%98.md   "Fragment专题"
[162]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Home%E9%94%AE%E7%9B%91%E5%90%AC.md   "Home键监听"
[163]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/HttpClient%E6%89%A7%E8%A1%8CGet%E5%92%8CPost%E8%AF%B7%E6%B1%82.md   "HttpClient执行Get和Post请求"
[164]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/JNI_C%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80.md   "JNI_C语言基础"
[165]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/JNI%E5%9F%BA%E7%A1%80.md   "JNI基础"
[166]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/ListView%E4%B8%93%E9%A2%98.md   "ListView专题"
[167]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Parcelable%E5%8F%8ASerializable.md   "Parcelable及Serializable"
[168]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/PopupWindow%E7%BB%86%E8%8A%82.md   "PopupWindow细节"
[169]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Scroller%E7%AE%80%E4%BB%8B.md   "Scroller简介"
[170]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/ScrollingTabs.md   "ScrollingTabs"
[171]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/SDK%20Manager%E6%97%A0%E6%B3%95%E6%9B%B4%E6%96%B0%E7%9A%84%E9%97%AE%E9%A2%98.md   "SDK Manager无法更新的问题"
[172]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Selector%E4%BD%BF%E7%94%A8.md   "Selector使用"
[173]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/SlidingMenu.md   "SlidingMenu"
[174]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/String%E6%A0%BC%E5%BC%8F%E5%8C%96.md   "String格式化"
[175]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/TextView%E8%B7%91%E9%A9%AC%E7%81%AF%E6%95%88%E6%9E%9C.md   "TextView跑马灯效果"
[176]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/WebView%E6%80%BB%E7%BB%93.md   "WebView总结"
[177]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Widget(%E7%AA%97%E5%8F%A3%E5%B0%8F%E9%83%A8%E4%BB%B6).md   "Widget(窗口小部件)"
[178]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/Wifi%E7%8A%B6%E6%80%81%E7%9B%91%E5%90%AC.md   "Wifi状态监听"
[179]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/XmlPullParser.md   "XmlPullParser"


[180]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/1.Kotlin_%E7%AE%80%E4%BB%8B%26%E5%8F%98%E9%87%8F%26%E7%B1%BB%26%E6%8E%A5%E5%8F%A3.md "1.Kotlin_简介&变量&类&接口"
[181]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/2.Kotlin_%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0%26Lambda%26%E5%86%85%E8%81%94%E5%87%BD%E6%95%B0.md "2.Kotlin_高阶函数&Lambda&内联函数.md"
[182]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/3.Kotlin_%E6%95%B0%E5%AD%97%26%E5%AD%97%E7%AC%A6%E4%B8%B2%26%E6%95%B0%E7%BB%84%26%E9%9B%86%E5%90%88.md "3.Kotlin_数字&字符串&数组&集合"
[183]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/4.Kotlin_%E8%A1%A8%E8%BE%BE%E5%BC%8F%26%E5%85%B3%E9%94%AE%E5%AD%97.md "4.Kotlin_表达式&关键字"
[184]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/5.Kotlin_%E5%86%85%E9%83%A8%E7%B1%BB%26%E5%AF%86%E5%B0%81%E7%B1%BB%26%E6%9E%9A%E4%B8%BE%26%E5%A7%94%E6%89%98.md "5.Kotlin_内部类&密封类&枚举&委托"
[185]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/6.Kotlin_%E5%A4%9A%E7%BB%A7%E6%89%BF%E9%97%AE%E9%A2%98.md "6.Kotlin_多继承问题"
[186]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/7.Kotlin_%E6%B3%A8%E8%A7%A3%26%E5%8F%8D%E5%B0%84%26%E6%89%A9%E5%B1%95.md "7.Kotlin_注解&反射&扩展"
[187]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/8.Kotlin_%E5%8D%8F%E7%A8%8B.md "8.Kotlin_协程"
[188]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/9.Kotlin_androidktx.md "9.Kotlin_androidktx"
[189]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%85%AB%E7%A7%8D%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95.md "八种排序算法"
[190]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%AE%80%E4%BB%8B.md "线程池简介"
[191]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md "设计模式"
[192]: https://github.com/CharonChui/AndroidNote/tree/master/JavaKnowledge/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E7%AE%97%E6%B3%95 "数据结构和算法"
[193]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86.md "动态代理"
[194]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/ConstraintLaayout%E7%AE%80%E4%BB%8B.md "ConstraintLaayout简介"
[195]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Http%E4%B8%8EHttps%E7%9A%84%E5%8C%BA%E5%88%AB.md "Http与Https的区别"
[196]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Top-K%E9%97%AE%E9%A2%98.md "Top-K问题"
[197]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/10.Kotlin_%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md "10.Kotlin_设计模式"
[198]: https://github.com/CharonChui/AndroidNote/blob/master/AppPublish/%E4%BD%BF%E7%94%A8Jenkins%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E5%8C%96%E6%89%93%E5%8C%85.md "使用Jenkins实现自动化打包"
[199]: https://github.com/CharonChui/AndroidNote/tree/master/Dagger2 "Dagger2"
[200]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/1.Dagger2%E7%AE%80%E4%BB%8B(%E4%B8%80).md  "1.Dagger2简介(一).md"
[201]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/2.Dagger2%E5%85%A5%E9%97%A8demo(%E4%BA%8C).md  "2.Dagger2入门demo(二).md"
[202]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/3.Dagger2%E5%85%A5%E9%97%A8demo%E6%89%A9%E5%B1%95(%E4%B8%89).md  "3.Dagger2入门demo扩展(三).md"
[203]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/4.Dagger2%E5%8D%95%E4%BE%8B(%E5%9B%9B).md  "4.Dagger2单例(四).md"
[204]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/5.Dagger2Lay%E5%92%8CProvider(%E4%BA%94).md  "5.Dagger2Lay和Provider(五).md"
[205]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/6.Dagger2Android%E7%A4%BA%E4%BE%8B%E4%BB%A3%E7%A0%81(%E5%85%AD).md  "6.Dagger2Android示例代码(六).md"
[206]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/7.Dagger2%E4%B9%8Bdagger-android(%E4%B8%83).md  "7.Dagger2之dagger-android(七).md"
[207]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/8.Dagger2%E4%B8%8EMVP(%E5%85%AB).md  "8.Dagger2与MVP(八).md"
[208]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/Android%20WorkManager.md  "Android WorkManager.md"

[209]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/4.RxJava%E8%AF%A6%E8%A7%A3%E4%B9%8B%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86(%E5%9B%9B).md  "4.RxJava详解之执行原理(四)"
[210]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/5.RxJava%E8%AF%A6%E8%A7%A3%E4%B9%8B%E6%93%8D%E4%BD%9C%E7%AC%A6%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86(%E4%BA%94).md  "5.RxJava详解之操作符执行原理(五)"
[211]: https://github.com/CharonChui/AndroidNote/blob/master/RxJavaPart/6.RxJava%E8%AF%A6%E8%A7%A3%E4%B9%8B%E7%BA%BF%E7%A8%8B%E8%B0%83%E5%BA%A6%E5%8E%9F%E7%90%86(%E5%85%AD).md  "6.RxJava详解之线程调度原理(六)"
[212]: https://github.com/CharonChui/AndroidNote/blob/master/Dagger2/9.Dagger2%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90(%E4%B9%9D).md "9.Dagger2原理分析(九)"
[213]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/%E8%B0%83%E8%AF%95%E5%B9%B3%E5%8F%B0Sonar.md "调试平台Sonar"
[214]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/AudioTrack%E7%AE%80%E4%BB%8B.md   "AudioTrack简介"
[215]: https://github.com/CharonChui/AndroidNote/blob/master/AdavancedPart/OOM%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90.md  "OOM问题分析"
[216]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/ExoPlayer  "ExoPlayer"
[217]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/ExoPlayer/1.%20ExoPlayer%E7%AE%80%E4%BB%8B.md  "1. ExoPlayer简介"
[218]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/ExoPlayer/2.%20ExoPlayer%20MediaSource%E7%AE%80%E4%BB%8B.md "2. ExoPlayer MediaSource简介"
[219]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/ExoPlayer/3.%20ExoPlayer%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B9%8Bprepare%E6%96%B9%E6%B3%95.md "3. ExoPlayer源码分析之prepare方法"
[220]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/ExoPlayer/4.%20ExoPlayer%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B9%8Bprepare%E5%BA%8F%E5%88%97%E5%9B%BE.md "4. ExoPlayer源码分析之prepare序列图"
[221]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/ExoPlayer/5.%20ExoPlayer%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B9%8BPlayerView.md "5. ExoPlayer源码分析之PlayerView"
[222]: https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%8F%8D%E7%BC%96%E8%AF%91.md "反编译"
[223]: https://github.com/CharonChui/AndroidNote/blob/master/Tools%26Library/Icon%E5%88%B6%E4%BD%9C.md "Icon制作"
[224]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE "流媒体协议"
[225]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F "视频封装格式"
[226]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/SurfaceView%E4%B8%8ETextureView.md "SurfaceView与TextureView"
[227]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E5%85%B3%E9%94%AE%E5%B8%A7.md "关键帧"
[228]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/CDN%E5%8F%8APCDN.md "CDN及PCDN"
[229]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/P2P%E6%8A%80%E6%9C%AF "P2P"
[230]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/%E6%92%AD%E6%94%BE%E5%99%A8%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md "播放器性能优化"
[231]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/OpenGL  "OpenGL"
[232]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/1.OpenGL%E7%AE%80%E4%BB%8B.md  "1.OpenGL简介"
[233]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/2.GLSurfaceView%E7%AE%80%E4%BB%8B.md "2.GLSurfaceView简介"
[234]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/3.GLSurfaceView%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md "3.GLSurfaceView源码解析"
[235]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/4.GLTextureView%E5%AE%9E%E7%8E%B0.md  "4.GLTextureView实现.md"
[236]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/5.OpenGL%20ES%E7%BB%98%E5%88%B6%E4%B8%89%E8%A7%92%E5%BD%A2.md "5.OpenGL ES绘制三角形"
[237]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/6.OpenGL%20ES%E7%BB%98%E5%88%B6%E7%9F%A9%E5%BD%A2%E5%8F%8A%E5%9C%86%E5%BD%A2.md "6.OpenGL ES绘制矩形及圆形"
[238]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/7.OpenGL%20ES%E7%9D%80%E8%89%B2%E5%99%A8%E8%AF%AD%E8%A8%80GLSL.md "7.OpenGL ES着色器语言GLSL"
[239]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/8.GLES%E7%B1%BB%E5%8F%8AMatrix%E7%B1%BB.md "8.GLES类及Matrix类"

[240]:  https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/9.OpenGL%20ES%E7%BA%B9%E7%90%86.md "9.OpenGL ES纹理"
[241]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/10.GLSurfaceView%2BMediaPlayer%E6%92%AD%E6%94%BE%E8%A7%86%E9%A2%91.md " 10.GLSurfaceView+MediaPlayer播放视频"
[242]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/11.OpenGL%20ES%E6%BB%A4%E9%95%9C.md "11.OpenGL ES滤镜"
[243]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/Danmaku "弹幕"
[244]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Danmaku/Android%E5%BC%B9%E5%B9%95%E5%AE%9E%E7%8E%B0.md "Android弹幕实现"
[245]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/MediaExtractor%E3%80%81MediaCodec%E3%80%81MediaMuxer.md "MediaExtractor、MediaCodec、MediaMuxer"
[246]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE.md  "流媒体协议"
[247]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE/HLS.md "HLS"
[248]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE/DASH.md "DASH"
[249]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE/HTTP%20FLV.md "HTTP FLV"
[250]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E6%B5%81%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE/RTMP.md "RTMP"
[251]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/MP4%E6%A0%BC%E5%BC%8F%E8%AF%A6%E8%A7%A3.md "MP4格式详解"
[252]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/FLV.md "FLV"
[253]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/TS.md "TS"
[254]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/fMP4%20vs%20ts.md "fMP4 vs ts"
[255]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/fMP4%E6%A0%BC%E5%BC%8F%E8%AF%A6%E8%A7%A3.md "fMP4格式详解"
[256]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F.md "视频封装格式"
[257]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81  "视频编码"
[258]:https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81/AV1.md   "AV1"
[259]:https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81/H264.md  "H264"
[260]:https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81/H265.md  "H265"
[261]:  https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/P2P%E6%8A%80%E6%9C%AF/P2P.md "P2P"
[262]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/P2P%E6%8A%80%E6%9C%AF/P2P%E5%8E%9F%E7%90%86_NAT%E7%A9%BF%E9%80%8F.md "P2P原理_NAT穿透"
[263]: https://github.com/CharonChui/AndroidNote/tree/master/OperatingSystem "操作系统"
[264]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/1.%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%AE%80%E4%BB%8B.md "1.操作系统简介"
[265]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/2.%E8%BF%9B%E7%A8%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B.md "2.进程和线程"
[266]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/3.%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.md "3.内存管理"
[267]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/4.%E8%B0%83%E5%BA%A6.md "4.调度"

[268]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/5.I:O.md "5.I/O"

[269]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/6.%E6%96%87%E4%BB%B6%E7%AE%A1%E7%90%86.md "6.文件管理"
[270]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/7.%E5%B5%8C%E5%85%A5%E5%BC%8F%E7%B3%BB%E7%BB%9F.md "7.嵌入式系统"
[271]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/8.%E8%99%9A%E6%8B%9F%E6%9C%BA.md "8.虚拟机"
[272]: https://github.com/CharonChui/AndroidNote/tree/master/Architect "架构设计"
[273]: https://github.com/CharonChui/AndroidNote/blob/master/Architect/1.%E6%9E%B6%E6%9E%84%E7%AE%80%E4%BB%8B.md "1.架构简介"
[274]: https://github.com/CharonChui/AndroidNote/tree/master/OperatingSystem/AndroidKernal  "Android内核"
[275]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/1.Android%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1.md "1.Android进程间通信"
[276]:  https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/2.Android%E7%BA%BF%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%E4%B9%8BHandler%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6.md "2.Android线程间通信之Handler消息机制"
[277]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/3.Android%20Framework%E6%A1%86%E6%9E%B6.md "3.Android Framework框架"
[278]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/4.ActivityManagerService%E7%AE%80%E4%BB%8B.md "4.ActivityManagerService简介"
[279]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/5.Android%E6%B6%88%E6%81%AF%E8%8E%B7%E5%8F%96.md "5.Android消息获取"
[280]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/6.%E5%B1%8F%E5%B9%95%E7%BB%98%E5%88%B6%E5%9F%BA%E7%A1%80.md "6.屏幕绘制基础"
[281]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/7.View%E7%BB%98%E5%88%B6%E5%8E%9F%E7%90%86.md "7.View绘制原理"
[282]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/8.WindowManagerService%E7%AE%80%E4%BB%8B.md "8.WindowManagerService简介"

[283]: https://github.com/CharonChui/AndroidNote/blob/master/OperatingSystem/AndroidKernal/9.PackageManagerService%E7%AE%80%E4%BB%8B.md "9.PackageManagerService简介"

[ 284 ]: https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/LeakCanary%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md. "LeakCanary源码分析"
[285]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.md "Java内存模型"
[286]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/JVM%E6%9E%B6%E6%9E%84.md "JVM架构"
[287]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack "Jetpack"
[288]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/Jetpack%E7%AE%80%E4%BB%8B.md  "Jetpack简介"
[289]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack/architecture "architecture"
[290]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack/ui  "ui"
[291]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack/foundation  "foundation"
[292]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack/behavior "behavior"
[293]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/1.%E7%AE%80%E4%BB%8B.md "1.简介"
[294]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/2.ViewBinding%E7%AE%80%E4%BB%8B.md "2.ViewBinding简介"
[295]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/3.Lifecycle%E7%AE%80%E4%BB%8B.md "3.Lifecycle简介"
[296]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/4.ViewModel%E7%AE%80%E4%BB%8B.md "4.ViewModel简介"
[297]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/5.LiveData%E7%AE%80%E4%BB%8B.md "5.LiveData简介"
[298]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/6.DataBinding%E7%AE%80%E4%BB%8B.md "6.DataBinding简介"
[299]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/7.Room%E7%AE%80%E4%BB%8B.md "7.Room简介"
[300]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/8.PagingLibrary%E7%AE%80%E4%BB%8B.md "8.PagingLibrary简介"
[301]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/9.App%20Startup%E7%AE%80%E4%BB%8B.md "9.App Startup简介"
[302]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/10.DataStore%E7%AE%80%E4%BB%8B.md "10.DataStore简介"
[303]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/11.Hilt%E7%AE%80%E4%BB%8B.md "11.Hilt简介"
[304]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/12.Navigation%E7%AE%80%E4%BB%8B.md "12.Navigation简介"
[305]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/13.Jetpack%20MVVM%E7%AE%80%E4%BB%8B.md "13.Jetpack MVVM简介"
[306]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/architecture/14.findViewById%E7%9A%84%E8%BF%87%E5%8E%BB%E5%8F%8A%E6%9C%AA%E6%9D%A5.md "14.findViewById的过去及未来"
[307]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/Jetpack%20Compose%E7%AE%80%E4%BB%8B.md "Jetpack Compose简介"
[308]: https://github.com/CharonChui/AndroidNote/tree/master/Jetpack/ui/material "material"
[309]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/1.MaterialToolbar%E7%AE%80%E4%BB%8B.md "1.MaterialToolbar简介"
[310]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/2.NavigationView%E7%AE%80%E4%BB%8B.md "2.NavigationView简介"
[311]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/3.NestedScrollView%E7%AE%80%E4%BB%8B.md "3.NestedScrollView简介"
[312]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/4.CoordinatorLayout%E7%AE%80%E4%BB%8B.md "4.CoordinatorLayout简介"
[313]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/5.AppBarLayout%E7%AE%80%E4%BB%8B.md "5.AppBarLayout简介"
[314]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/6.CollapsingToolbarLayout%E7%AE%80%E4%BB%8B.md "6.CollapsingToolbarLayout简介"
[315]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/7.Snackbar%E7%AE%80%E4%BB%8B.md "7.Snackbar简介"
[316]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/ui/material/8.TabLayout%E7%AE%80%E4%BB%8B.md "8.TabLayout简介"
[317]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/foundation/1.%E7%AE%80%E4%BB%8B.md "1.简介"
[318]: https://github.com/CharonChui/AndroidNote/blob/master/Jetpack/behavior/1.%E7%AE%80%E4%BB%8B.md "1.简介"
[319]: https://github.com/CharonChui/AndroidNote/blob/master/Gradle%26Maven/Composing%20builds%E7%AE%80%E4%BB%8B.md  "Composing builds简介"
[320]: https://github.com/CharonChui/AndroidNote/blob/master/ImageLoaderLibrary/Coil%E7%AE%80%E4%BB%8B.md "Coil简介"
[321]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/M3U8.md "M3U8"
[322]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/FFmpeg "FFmpeg"
[323]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/1.FFmpeg%E7%AE%80%E4%BB%8B.md "1.FFmpeg简介"
[324]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/2.FFmpeg%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E8%A1%8C.md "2.FFmpeg常用命令行"
[325]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/3.FFmpeg%E5%88%87%E7%89%87.md "3.FFmpeg切片"
[326]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/%E9%9F%B3%E8%A7%86%E9%A2%91%E5%90%8C%E6%AD%A5%E5%8E%9F%E7%90%86.md "音视频同步原理"
[327]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/%E9%9F%B3%E8%A7%86%E9%A2%91%E5%9C%BA%E6%99%AF.md "音视频场景"
[328]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/1.%E9%9F%B3%E8%A7%86%E9%A2%91%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md "1.音视频基础知识"
[329]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/2.%E7%B3%BB%E7%BB%9F%E6%92%AD%E6%94%BE%E5%99%A8MediaPlayer.md "2.系统播放器MediaPlayer"
[330]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/11.%E6%92%AD%E6%94%BE%E7%BB%84%E4%BB%B6%E5%B0%81%E8%A3%85.md "11.播放器组件封装"
[331]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81%E5%8E%9F%E7%90%86.md "视频编码原理"
[332]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenGL/12.FBO.md "12.FBO"
[333]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/4.%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE.md "4.开发环境配置"
[334]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/5.%20FFmpeg%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD.md "5. FFmpeg核心功能"
[335]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81 "音频编码"
[336]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81%E6%A0%BC%E5%BC%8F.md "音频编码格式"
[337]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81/AAC.md "AAC"
[338]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81/PCM.md "PCM"
[339]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E9%9F%B3%E9%A2%91%E7%BC%96%E7%A0%81/WAV.md "WAV"
[340]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/FFmpeg/6.%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E7%AE%80%E4%BB%8B.md  "6.视频播放简介"
[341]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/%E8%A7%86%E9%A2%91%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F/AVI.md "AVI"
[342]: https://github.com/CharonChui/AndroidNote/tree/master/VideoDevelopment/OpenCV "OpenCV"
[343]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/OpenCV/1.OpenCV%E7%AE%80%E4%BB%8B.md "1.OpenCV简介"
[344]: https://github.com/CharonChui/AndroidNote/blob/master/VideoDevelopment/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91/MediaMetadataRetriever.md    "MediaMetadataRetriever"



Developed By
===

 * Charon Chui - <charon.chui@gmail.com>


License
===

    Copyright (C) 2013 Charon Chui <charon.chui@gmail.com>
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
       http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
