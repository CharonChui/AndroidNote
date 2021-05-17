# Jetpack简介

JetPack是Google推出的一些库的集合。是Android基础支持库SDK以外的部分。包含了组件、工具、架构方案等...开发者可以自主按需选择接入具体的哪个库。

## 背景 

### Support库

早之前的Android更新迭代是，所有的功能更新都是跟随着每一个特定的Android版本所发布的。
例如: 
- Fragment是在Android 3.0更新的。
- Material Design组件是在Android 5.0上更新的
但是由于Android用户庞大，每一次Android更新都无法覆盖所有用户的，同时因为手机厂商众多，但支持有限，也无法为自己生产的所有设备保持迭代到最新的Android版本，所以用户所持有的设备上Android版本是层次不齐的。
从技术的角度来说，因为用户的设备版本不一致，导致Android工程师在维护项目的时候，会遇到很多难以解决的问题。为了解决这些由于Android版本不一致而出现的兼容性问题，Google推出了Support库。

Support库是针对Framework API的补充，Framework API跟随每一个Android版本所发布，和设备具有强关联性，Support API是开发者自主集成的，最终会包含在我们所发布的应用中，这样我们就可以在最新的Android版本上进行应用的开发，同时使用Support API解决各种潜在的兼容性问题，帮助开发者在不同Android版本的设备上实现行为一致的工作代码。

### Support 库的弊端
最早的Support库发布于2011年，版本号为：android.support.v4，也就是我们所熟知的v4库，2013年在v4的基础上，Android团队发布了v7库，版本号为：android.support.v7，之后还发布了用于特定场景的v8、v13、v14、v17。    
如果是前几年刚开始学习Android的同学们，一定都对这些奇怪的数字很疑惑，4、7、8、13、14、17 到底都是什么意思？   
拿第一代支持库v4举例，最初本意是指：该支持库最低可以支持到API 4(Android 1.4)的设备，v7表示最低支持 API 7(Android 2.1)的设备，但随着Android版本的持续更新，API 4以及API 7的设备早就淘汰了。在2017年7月Google将所有支持库的最低API支持版本提高到了API 14(Android 4.0)，但由于包名无法修改，所以还是沿用之前的v4、v7命名标准，所以就出现了Support库第一个无法解决的问题：版本增长混乱。

与此同时Support库还面临一个非常严峻的问题：架构设计本身导致的严重依赖问题。最早的Support库是v4，v7是基于v4进行的补充，因为v4、v7太过庞大，功能集中，所以如果想要开发新的支持库，也只能在v4、v7的基础上进行二次开发，比方说我们后期常用的，RecyclerView、CardView等等。

这样就会产生很严重的重复依赖的问题，在无论是使用官方库，还是第三方库的时候，我们都需要保持整个项目中Support库版本一致，我相信很多人都在这个问题上踩过坑，虽然还是有办法解决这个问题，但无形中增加了很多工作量。

我们都知道“组合优于继承”这句话，但Support库在最初的架构设计上，却采用了重继承轻组合的方式，我猜这可能是因为开发Support库的人和开发Framework API的是同一批人有关，Framework API里有种各种继承逻辑，例如我们常用的 Activity、Fragment、View。

虽然在后期Google尝试拆分Support库，例如推出了独立的支持库：support:design、support:customtabs等，但并不能从根源解决依赖的问题。

### Android X
从Goole IO 2017开始。Google开始推出Architecture Component,ORM库Room,用户生命周期管理的ViewModel/LiveData.
Goole IO 2018将Support lib更名为androidx.将许多Google认为是正确的方案和实践集中起来。可以说AndroidX的出现就是为了解决长久以来Support库混乱的问题，你也可以把AndroidX理解为更强大的Support库。
AndroidX将原有的Support库拆分为85个大大小小的支持库，抛弃了之前与API最低支持相关联的版本命名规范，重置为1.0.0，并且每一个库在之后都会按照严格的语义版本控制规则进行版本控制。
同时通过组合依赖的方式，我们可以选择自己需要的组件库，而不是像Support一样全部依赖，一定程度上也减小了应用的体积。
很重要的一点，就是它不会随着特定的Android版本而更新，它是由开发者自主控制，同时包含在我们所发布的应用程序中。


以上种种，现在统称为JetPack.
Jetpack的出现是为了彻底解决这两个致命的问题:   
1. Support 库版本增长混乱
2. Support 库重复依赖
如果Jetpack仅仅是针对Support库的重构，那它并没有了不起的，因为这只是Google解决了它自身因为历史原因所产生的代码问题。
更重要的是Jetpack为大家提供了一系列的最佳实践，包含：架构、数据处理、后台任务处理、动画、UI 各个方面，无需纠结于各种因为Android本身而出现的问题，而是让我们把更多的精力放在业务需求的实现上，这才是Jetpack真正了不起的地方。其最核心的出发点就是帮助开发者快速构建出稳定、高性能、测试友好同时向后兼容的APP。

Jetpack相当于Google把自己的Android生态重新整理了一番。确立了Android未来的版图和大方向。

## 组成部分

前面讲到过，JetPack是一系列库和工具的集合，它更多是Google的一个提出的一个概念，或者说态度。
并非所有的东西都是每年在IO大会上新推出的，它也包含了对现有基础库的整理和扩展。在大部分项目中其实我们都有用到JetPack的内容，也许你只是不知道而已。让我们以上帝视角来看看整个JetPack除了你熟悉的部分，还有哪些是你不熟悉但是听过的内容。看看他们都能做些什么事情。
![](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_jetpack.png)

[Jetpack完整组件列表](https://developer.android.com/jetpack/androidx/explorer)

### Foundation(基础组件)：
基础组件提供了横向功能，例如向后兼容性、测试以及Kotlin语言的支持。它包含如下组件库: 
- Android KTX：Android KTX 是一组 Kotlin 扩展程序，它优化了供Kotlin使用的Jetpack和Android平台的API。以更简洁、更愉悦、更惯用的方式使用Kotlin进行Android开发。
- AppCompat：提供了一系列以AppCompat开头的API，以便兼容低版本的Android开发。Jetpack基础中的AppCompat库包含v7库中的所有组件([支持库软件包](https://developer.android.com/topic/libraries/support-library/packages#v7-appcompat))。 其中包括AppCompat，Cardview，GridLayout，MediaRouter，Palette，RecyclerView，Renderscript，Preferences，Leanback，Vector Drawable，Design，Custom选项卡等。此外，该库为材质设计用户界面提供了实现支持，这使得AppCompat对 开发人员。 以下是android应用程序的一些关键领域，这些领域很难构建，但是可以使用AppCompat库轻松进行设计： 一般都是为了兼容 Android L以下版本，来提供Material Design的效果: 
    - Toolbar
    - ContextCompat
    - AppCompatDialog
- annotation:注解，提升代码可读性，内置了Android中常用的注解
- Multidex(多Dex处理)：为方法数超过 64K 的应用启用多 dex 文件。Security(安全)：按照安全最佳做法读写加密文件和共享偏好设置。
- Test(测试)：用于单元和运行时界面测试的 Android 测试框架。

### Architecture(架构组件)
架构组件可帮助开发者设计稳健、可测试且易维护的应用。它包含如下组件库: 
- Data Binding(数据绑定):数据绑定库是一种支持库，借助该库，可以使用声明式将布局中的界面组件绑定到应用中的数据源。
- Lifecycles:方便管理Activity和Fragment生命周期，帮助开发者书写更轻量、易于维护的代码。
- ViewModel：以生命周期感知的方式存储和管理与UI相关的数据。
- LiveData:是一个可观察的数据持有者类。与常规observable不同，LiveData是有生命周期感知的。
- Navigation:处理应用内导航所需的一切。
- Paging：帮助开发者一次加载和显示小块数据。按需加载部分数据可减少网络带宽和系统资源的使用。
- Room：Room持久性库在SQLite上提供了一个抽象层，帮助开发者更友好、流畅的访问SQLite数据库。
- WorkManager：即使应用程序退出或设备重新启动，也可以轻松地调度预期将要运行的可延迟异步任务。
- hilt:基于Dagger的Android依赖注入框架绑定View和Model
- startup:自动处理依赖初始化
- datastore:Preferences的替代类，支持异步、更加安全



### Behavior(行为组件)
行为组件可帮助开发者的应用与标准Android服务（如通知、权限、分享和Google助理）相集成。它包含如下组件库: 
- CameraX：帮助开发者简化相机应用的开发工作。它提供一致且易于使用的 API 界面，适用于大多数 Android 设备，并可向后兼容至 Android 5.0（API 级别 21）。
- DownloadManager下载管理器:可处理长时间运行的HTTP下载，并在出现故障或在连接更改和系统重新启动后重试下载。
- Media & playback(媒体&播放)：用于媒体播放和路由（包括 Google Cast）的向后兼容 API。
- Notifications(通知)：提供向后兼容的通知 API，支持 Wear 和 Auto。
- Permissions(权限)：用于检查和请求应用权限的兼容性 API。
- Preferences(偏好设置)：提供了用户能够改变应用的功能和行为能力。
- Sharing(共享)：提供适合应用操作栏的共享操作。
- Slices(切片)：创建可在应用外部显示应用数据的灵活界面元素。

### UI(界面组件)
大多数的UI组件其实都包含在基础组件中的appcompat中，这里是一些独立组件库存在的UI组件，界面组件可提供各类view和辅助程序，让应用不仅简单易用，还能带来愉悦体验。它包含如下组件库: 
- drawerlayout:抽屉布局
- recyclerview:可复用的滑动列表
- constraintlayout:约束布局
- compose*: Jetpack compose声明式UI
- coordinatorlayout:顶层布局继承自Framelayout，可以实现子View之间的联动交互效果
- swiperefreshlayout:下拉刷新布局
- viewpager2:分页布局
- Material Design Components * : MD组件
- Animation & Transitions(动画&过度)：提供各类内置动画，也可以自定义动画效果。
- Emoji(表情符号)：使用户在未更新系统版本的情况下也可以使用表情符号。
    -  EmojiTextView
    -  EmojiEditTExt
    -  EmojiButton
- Fragment：组件化界面的基本单位。
- Layout(布局)：xml书写的界面布局或者使用Compose完成的界面。
    用户界面结构（如应用程序的活动）由Layout定义。 它定义了View和ViewGroup对象。 可以通过两种方式创建View和ViewGroup：通过以XML声明UI元素或通过编写代码（即以编程方式）。 Jetpack的这一部分涵盖了一些最常见的布局，例如LinearLayout，RelativeLayout和全新的ConstraintLayout。 而且，官方的Jetpack布局文档提供了一些指导，以使用RecyclerView创建项目列表以及使用CardView创建卡布局。 用户可以看到一个视图。 EditView，TextView和Button是View的示例。 另一方面，ViewGroup是一个容器对象，它定义了View的布局结构，因此它是不可见的。 ViewGroup的示例是LinearLayout，RelativeLayout和ConstraintLayout。 
- Palette(调色板)：从调色板中提取出有用的信息。


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/jetpack_compose.png?raw=true)

[Android Jetpack Compose](https://developer.android.com/jetpack/compose)是2019 Google/IO大会上推出的一种声明式的UI开发框架，经过一年左右的演进，现在到了alpha阶段。Jetpack Compose是用于构建原生界面的新款Android工具包。它可简化并加快Android上的界面开发。使用更少的代码、强大的工具和直观的KotlinAPI，快速让应用生动而精彩，从此不再需要写xml，使用声明式的Compose函数来构建页面UI。

Compose由androidx中的6个Maven组ID构成。每个组都包含一套特定用途的功能，并各有专属的版本说明。下表介绍了各个组及指向其版本说明的链接: 
- compose.animation:在Jetpack Compose应用中构建动画，丰富用户的体验。
- compose.compiler:借助Kotlin编译器插件，转换@Composable functions（可组合函数）并启用优化功能。
- compose.foundation:使用现成可用的构建块编写Jetpack Compose应用，还可扩展Foundation以构建您自己的设计系统元素。
- compose.material:使用现成可用的Material Design组件构建Jetpack Compose UI。这是更高层级的Compose入口点，旨在提供与www.material.io上描述的组件一致的组件。
- compose.runtime:Compose的编程模型和状态管理的基本构建块，以及Compose编译器插件针对的核心运行时。
- compose.ui:与设备互动所需的Compose UI的基本组件，包括布局、绘图和输入。
```kotlin
class MainActivity : AppCompatActivity() {
    overridefun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
             Text("Hello, Android技术杂货铺")
        }
    }
}
```
废弃部分: 
- asynclayoutinflater:异步生成UI
- localbroadcastmanager:本地广播
- viewpager:使用vviewpager2
- cardview:使用MaterialCardView替代


## Jetpack的意义
Jetpack里目前包含的内容，未来也会是Google大力维护和扩展的内容。对应开发者来说也是值得去学习使用的且相对无后顾之忧的。JetPack里没有的，除开一些优秀的第三方库，未来应该也会慢慢被新的API替代，逐渐边缘化，直至打上Deprecate注解。

以当下的环境来说，要开发出一个完全摆脱JetPack的APP是很难做到的。但是反过来讲JetPack也远远没有到成熟的地步，目前也还存在亟待解决的问题，未来可以做的事情还有很多。

关于使用的话，并不是所有库都建议使用，因为目前还有很多库在alpha版本。但是作为学习还是很有必要的，能给你日常的开发中多提供一些思路，这些是无可厚非的。





## Jetpack库列表及集成

[Jetpack各库版本及gradle集成使用](https://developer.android.com/jetpack/androidx/releases/activity)



## 参考
- [Jetpack Compose初体验 ](https://easyliu-ly.github.io/2020/12/12/android_jetpack/compose/)
