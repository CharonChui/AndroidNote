AndroidRuntime_ART与Dalvik
===
在说Android Runtime之前，我们需要了解什么是运行时环境，还需要了解一些基本知识，即JVM和Dalvik VM的功能。 

## Runtime

用最简单的术语来说，它是操作系统使用的系统，它负责将您用Java之类的高级语言编写的代码转换为CPU/处理器能理解机器代码。

运行时包含在程序运行时执行的软件指令，即使它们实际上并不是该软件代码的一部分也是如此。
CPU或更笼统的术语我们的计算机仅理解机器语言（二进制代码），因此要使其在CPU上运行，必须将代码转换为机器代码，这由翻译器完成。
因此，以下是按顺序生成翻译器的过程: 
- Assemblers
    它可以直接将汇编代码转换为机器代码，因此速度非常快。
- Compilers
    它将代码转换为汇编代码，然后使用汇编程序将代码转换为二进制。使用此编译速度很慢，但是执行速度很快。但是编译器的最大问题是所生成的机器代码取决于平台。换句话说，在一台计算机上运行的代码可能不会在另一台计算机上运行。
- Interpreters
    它在执行代码时翻译代码。由于翻译是在运行时进行的，因此执行速度很慢。
### JVM
为了维持代码的平台独立性，JAVA开发了JVM，即Java虚拟机。它针对每个平台开发了JVM，这意味着JVM依赖于该平台。Java编译器将.java文件转换为.class文件，这称为字节码。该字节码被提供给JVM，该JVM将其转换为机器码。

### Android Runtime

当我们构建应用程序并生成APK时，该APK的一部分是.dex文件。这些文件包含我们应用程序的源代码，包括我们在为软件解释器设计的低级代码（字节码）中使用的所有库。

当用户运行我们的应用程序时，写入的.dex文件中的字节码将由Android Runtime转换为机器码—一组指令，机器可以直接理解并由CPU处理。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/dex_local_code.png)

Android Runtime同样也回管理内存及垃圾回收。



![](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm_dvm.png)



可以使用各种策略将字节码编译为机器代码，所有这些策略都有其取舍。为了了解Android Runtime的工作原理，需要首先了解Dalvik。



## Android Runtime的发展

### Dalvik（<= Android K）

`Dalvik`是`Google`公司自己设计用于`Android`平台的`Java`虚拟机。它可以支持已转换为`.dex`（即`Dalvik Executable`）格式的`Java`应用程序的运行，
`.dex`格式是专为`Dalvik`设计的一种压缩格式，适合内存和处理器速度有限的系统。`Dalvik`经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个`Dalvik`应用作为一个独立的`Linux`进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。
很长时间以来，`Dalvik`虚拟机一直被用户指责为拖慢安卓系统运行速度不如`IOS`的根源。
`2014`年`6`月`25`日，`Android L`正式亮相于召开的谷歌`I/O`大会，`Android L`改动幅度较大，谷歌将直接删除`Dalvik`，代替它的是传闻已久的`ART`。



早期，Android智能手机并不像现在那么强大。大多数手机的RAM很少，有些甚至只有200MB。
难怪第一个被称为Dalvik的Android Runtime的实现正是为了优化此参数：RAM的使用。

因此，它没有在运行它之前将整个应用程序编译为机器代码，而是使用了称为Just In Time编译（简称JIT）的策略。

在这种策略下，编译器可以充当解释器。它在应用程序执行期间（在运行时）编译一小段代码。

而且由于Dalvik仅编译所需的代码并在运行时执行它，因此可以节省大量RAM。

使用Dalvik JIT编译器，每次运行该应用程序时，它会将Dalvik字节码的一部分动态转换为机器代码。随着执行的进行，更多的字节码将被编译和缓存。由于JIT仅编译部分代码，因此它具有较小的内存占用空间，并且在设备上使用的物理空间更少。

但是此策略有一个严重的缺点-因为所有这些都在运行时发生，因此显然会对运行时性能产生负面影响。

最终，引入了一些优化以使Dalvik更具性能。一些经常使用的已编译代码段已被缓存，不再重新编译。但这是非常有限的，因为在最初的日子里内存非常稀缺。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/dalvik_jit.png)

几年来运行良好，但与此同时，手机的性能越来越高，RAM也越来越多。而且由于应用程序也越来越大，因此JIT性能影响变得越来越成问题。

这就是为什么在Android L中引入了新的Android Runtime：ART。



### ART(Android L)

`Android 4.4`提供了一种与`Dalvik`截然不同的运行环境`ART`支持,`ART`源于`google`收购的`Flexycore`的公司。
`ART`模式与`Dalvik`模式最大的不同在于，启用`ART`模式后，系统在安装应用的时候会进行一次预编译，将字节码转换为机器语言存储在本地，这样在运行程序时就不会每次都进行一次编译了，执行效率也大大提升。

`ART`使用`AOT(Ahead Of Time)`(静态编译)而`Dalvik`使用`JIT(Just In Time)`(动态编译)
`JIT`方式会在程序执行时将`Dex bytecode`(`java`字节码)转换为处理器可以理解的本地代码，这种方式会将编译时间计入程序的执行时间，程序执行会显得慢一些。`AOT`方式会在程序执行之前(一般是安装时)就编译好本地代码，因此程序执行时少了编译的过程会显得快一些，但占用更多存储空间，安装时也会更慢。但没针对ART优化的程序反而会运行得更慢，随着`Android L`的普及这个问题迟早会解决。`ART`拥有改进的`GC`(垃圾回收)机制:`GC`时更少的暂停时间、`GC`时并行处理、某些时候`Collector`所需时间更短、减少内存不足时触发GC的次数、减少后台内存占用。   
在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。总体的理念就是空间换时间。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/dvm_art.png)

`AOT`的编译器分两种模式：            

- 在开发机上编译预装应用；
- 在设备上编译新安装的应用,在应用安装时将`dex`字节码翻译成本地机器码。



这种方法极大地提高了运行时性能，因为运行本机机器代码甚至比即时编译快20倍。

ART优点:     

- 系统性能的显著提升。
- 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
- 更长的电池续航能力。
- 支持更低的硬件。

ART缺点:       

- 更大的存储空间占用，可能会增加10%-20%。字节码预先编译成机器码并存储到本地，机器码需要的存储空间更大。
- 更长的应用安装时间，因为下载APK后，整个应用程序都需要转换为机器代码，而且由于所有应用程序都需要重新优化，因此执行系统更新还需要更长的时间。
- Android L中的ART使用的内存比Dalvik多得多。



对于应用程序中经常运行的部分来说，对其进行预编译显然是有回报的，但现实是，用户很少打开应用程序的大多数部分，而对整个应用程序进行预编译几乎也没有回报。

这就是为什么在Android N中，Just In Time编译与称为配置文件引导编译(profile-guided complication)一起被引入到Android Runtime的原因。

### Profile-guided compilation(Android N)

配置文件引导编译是一种策略，可以在运行Android应用程序时不断提高其性能。默认情况下，应用程序使用即时编译策略进行编译，但是当ART检测到某些热点功能时，这意味着它们经常运行，ART可以预编译并缓存这些方法以获得最佳性能。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/art_profile_guide.png)

该策略可为应用程序的关键部分提供最佳性能，同时减少RAM使用量。由于事实证明，对于大多数应用程序来说，通常仅使用10％到20％的代码。

更改ART之后，不再影响应用程序安装和系统更新的速度。应用的关键部分的预编译仅在设备空闲和充电时进行，以最大程度地减少对设备电池的影响。



这种方法的唯一缺点是，为了获取配置文件数据并预编译常用的方法和类，用户必须实际使用应用程序。这意味着该应用程序的一些首次使用可能会有点慢，因为在这种情况下，将仅使用即时编译。

这就是为什么要改善在Android P中的初始用户体验的原因，Google在云中引入了个人资料。

### Profiles in the cloud(Android P)

云中的配置文件背后的主要思想是，大多数人以非常相似的方式使用该应用程序。因此，为了在安装后立即提高性能，我们可以从已经使用过此应用程序的人那里收集配置文件数据。此汇总的配置文件数据用于为应用程序创建一个称为通用核心配置文件的文件。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/profiles_in_cloud.png)

因此，当新用户安装该应用程序时，该文件将与该应用程序一起下载。 ART使用它来预编译大多数用户经常运行的类和方法。这样，新用户在下载应用程序后即可获得更好的性能。

这并不意味着不再使用旧策略。用户运行应用程序后，ART将收集用户特定的配置文件数据并重新编译设备闲置时该特定用户经常使用的代码。

而最好的部分是，我们开发应用程序的开发人员无需执行任何操作即可启用此功能。这一切都发生在Android Runtime中。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_runtime_change.png)



- 最初Android使用Dalvik，它使用即时编译(JIT)来优化RAM(那时内存是非常紧缺的)的使用。
- 为了提高Android L中的性能，引入了使用Ahead of time编译的ART。这样可以实现更好的运行时性能，但会导致更长的安装时间和更多的RAM使用率。
- 因此，在Android N中，JIT被引入到ART中，并且配置文件引导的编译允许为经常运行的部分代码提供更好的性能。
- 为了让用户在Android P中安装应用后立即获得最佳性能，Google在云端引入了配置文件，它通过添加随APK下载的通用核心配置文件来补充以前的优化，并允许ART预编译部分代码最常由以前的应用程序用户运行。





[Android Runtime-How Dalvik and ART works?](https://www.youtube.com/watch?v=0J1bm585UCc&t=27s)


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
