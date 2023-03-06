# JVM垃圾回收机制

[Java内存模型](https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.md)如下图所示，**堆和方法区是所有线程共有的**，而虚拟机栈，本地方法栈和程序计数器则是线程私有的。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm.png)

### 堆（Heap）

Java堆是垃圾收集器管理的主要区域，因此也被称作GC堆（Garbage Collected Heap)。Java的自动内存管理主要是针对对象内存的回收和对象
内存的分配。同时，Java自动内存管理最核心的功能是堆内存中对象的分配与回收。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，
所以Java堆还可以细分为：新生代和老年代，再细致一点有：Eden空间、From Survivor、To Survivor空间等。进一步划分的目的是更好地回收
内存，或者更快地分配内存。

**堆空间的基本结构：**

![](http://raw.githubusercontent.com/CharonChui/Pictures/master/java_heap.png)

在垃圾回收的时候，我们往往将堆内存分成**新生代和老生代（大小比例1：2）**，新生代中由Eden和Survivor0，Survivor1组成，**三者的比例是8：1：1**，
新生代的回收机制采用**复制算法**，大部分情况，对象都会首先在Eden区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入s0或者s1，并且对象的
年龄还会加1(Eden区->Survivor区后对象的初始年龄变为1)，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄
阈值，可以通过参数`-XX:MaxTenuringThreshold`来设置。老生代采用的回收算法是**标记整理算法。**

#### 对象优先在eden区分配

目前主流的垃圾收集器都会采用分代回收算法，因此需要将堆内存分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

大多数情况下，对象在新生代中eden区分配。当eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC.

这里说一下Minor GC和Full GC有什么不同呢？

- 新生代GC（Minor GC）:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。
- 老年代GC（Major GC/Full GC）:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上。

#### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。这样做主要是为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率。

#### 长期存活的对象将进入老年代

既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对
象一个对象年龄（Age）计数器。

如果对象在Eden出生并经过第一次Minor GC后仍然能够存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1.对象在
Survivor中每熬过一次MinorGC,年龄就增加1岁，当它的年龄增加到一定程度（一般都会说默认为 15 岁，其实默认晋升年龄并不都是15，这个是要区分垃圾收集器的，CMS就是6），
就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`来设置。


## 如何判断对象是垃圾

### 引用计数算法

在`JDK1.2`之前，使用的是引用计数器算法，即当这个类被加载到内存以后，就会产生方法区，
堆栈、程序计数器等一系列信息，当创建对象的时候，为这个对象在堆栈空间中分配对象，
同时会产生一个引用计数器，同时引用计数器+1，当有新的引用的时候，引用计数器继续+1，
而当其中一个引用销毁的时候，引用计数器-1，当引用计数器被减为零的时候，
标志着这个对象已经没有引用了，可以回收了！
这种算法在JDK1.2之前的版本被广泛使用，但是随着业务的发展，很快出现了一个问题，那就是互相引用的问题:   

```java
ObjA.obj = ObjB
ObjB.obj - ObjA
```
这样的代码会产生如下引用情形`objA`指向`objB`，而`objB`又指向`objA`，这样当其他所有的引用都消失了之后，
`objA`和`objB`还有一个相互的引用，也就是说两个对象的引用计数器各为1，
而实际上这两个对象都已经没有额外的引用，已经是垃圾了。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/yinyongjishu.jpg)


### 根搜索算法

根搜索算法是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，
从一个节点`GC ROOT`开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，
当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。           
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/genshousuo.jpg)

目前java中可作为GC Root的对象有:    

- 虚拟机栈中引用的对象（本地变量表）
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中引用的对象（Native对象）

垃圾回收算法
---

而收集后的垃圾是通过什么算法来回收的呢：   

- 标记-清除算法
- 复制算法
- 标记整理算法

那我们就继续分析下这三种算法：  

### 标记-清除算法(Mark-Sweep)        
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/biaoji_qingchu.png)	

原理：从根集合节点进行扫描，标记出所有的存活对象，最后扫描整个内存空间并清除没有标记的对象（即死亡对象）    

适用场合：

- 存活对象较多的情况下比较高效 
- 适用于年老代（即旧生代） 

缺点: 

- 标记清除算法带来的一个问题是会存在大量的空间碎片，因为回收后的空间是不连续的，这样给大对象分配内存的时候可能会提前触发full gc。 

### 标记复制算法(mark-copy)           
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/fuzhisuanfa.png)

原理：思路也很简单，将内存对半分，总是保留一块空着（上图中的右侧），将左侧存活的对象（浅灰色区域）复制到右侧，然后左侧全部清空。避免了内存碎片
问题，但是内存浪费很严重，相当于只能使用50%的内存。从根集合节点进行扫描，标记出所有的存活对象，并将这些存活的对象复制到一块儿新的内存
（图中下边的那一块儿内存）上去，之后将原来的那一块儿内存（图中上边的那一块儿内存）全部回收掉 

适用场合：

- 存活对象较少的情况下比较高效 
- 扫描了整个空间一次（标记存活对象并复制移动） 
- 适用于年轻代（即新生代）：基本上98%的对象是”朝生夕死”的，存活下来的会很少 

缺点：

- 需要一块儿空的内存空间 
- 需要复制移动对象 

### 标记-整理算法(Mark-Compact)              
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/biaoji_zhengli.png)

原理：从根集合节点进行扫描，标记出所有的存活对象，最后扫描整个内存空间并清除没有标记的对象（即死亡对象）（可以发现前边这些就是标记-清除算法的原理），
清除完之后，将所有的存活对象左移到一起。避免了上述两种算法的缺点，将垃圾对象清理掉后，同时将剩下的存活对象进行整理挪动（类似于 windows 的磁盘碎片整理），
保证它们占用的空间连续，这样就避免了内存碎片问题，但是整理过程也会降低GC的效率。

适用场合：

- 用于年老代（即旧生代） 

缺点：

- 需要移动对象，若对象非常多而且标记回收后的内存非常不完整，可能移动这个动作也会耗费一定时间 
- 扫描了整个空间两次（第一次：标记存活对象；第二次：清除没有标记的对象） 

优点：

- 不会产生内存碎片 

### 分代收集算法(Generational Collection)

上述三种算法，每种都有各自的优缺点，都不完美。在现代JVM中，往往是综合使用的，经过大量实际分析，发现内存中的对象，大致可以分为两类：
有些生命周期很短，比如一些局部变量/临时对象，而另一些则会存活很久，典型的比如websocket长连接中的connection对象，如下图：

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/62226d097ac54148804119b4c239c802.png)

纵向y轴可以理解分配内存的字节数，横向x轴理解为随着时间流逝（伴随着 GC），可以发现大部分对象其实相当短命，很少有对象能在GC后活下来。
因此诞生了分代的思想，以Hotspot为例（JDK 7）：

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/e4ff361d409b6939e6da49a06b6dc677.png)

将内存分成了三大块：年青代（Young Genaration），老年代（Old Generation）, 永久代（Permanent Generation），其中Young Genaration更是又细分为eden，
S0，S1三个区。

结合我们经常使用的一些jvm调优参数后，一些参数能影响的各区域内存大小值，示意图如下：

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/d4e6583cd0ecfa60622526e7a4f13634.png)

注：jdk8开始，用MetaSpace区取代了Perm区（永久代），所以相应的jvm参数变成-XX:MetaspaceSize及-XX:MaxMetaspaceSize。

以Hotspot为例，我们来分析下GC的主要过程：

刚开始时，对象分配在eden区，s0（即：from）及s1（即：to）区，几乎是空着。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/8dc1423cc9f85658a946e204dc85ec18.png)

随着应用的运行，越来越多的对象被分配到eden区。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/c9138916377192a8a2590aca3e888049.png)

当eden区放不下时，就会发生minor GC（也被称为young GC），第1步当然是要先标识出不可达垃圾对象（即：下图中的黄色块），然后将可达对象，
移动到s0区（即：4个淡蓝色的方块挪到s0区），然后将黄色的垃圾块清理掉，这一轮过后，eden区就成空的了。

注：这里其实已经综合运用了“【标记 - 清理 eden】 + 【标记 - 复制 eden->s0】”算法。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/adc376cad0670c6993da8f32f3a88aba.png)

随着时间推移，eden如果又满了，再次触发minor GC，同样还是先做标记，这时eden和s0区可能都有垃圾对象了（下图中的黄色块），注意：这时s1（即：to）区是空的，
s0区和eden区的存活对象，将直接搬到s1区。然后将eden和s0区的垃圾清理掉，这一轮minor GC后，eden和s0区就变成了空的了。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/f21d18a0736d0976c4ecf14e82236cb2.png)

继续，随着对象的不断分配，eden区可能又满了，这时会重复刚才的minor GC过程，不过要注意的是，这时候s0是空的，所以s0与s1的角色其实会互换，
即：存活的对象，会从eden和s1区，向s0区移动。然后再把eden和s1区中的垃圾清除，这一轮完成后，eden与s1区变成空的，如下图。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/f765eb0f46635ae093516f35fb1eee66.png)

对于那些比较“长寿”的对象一直在s0与s1中挪来挪去，一来很占地方，而且也会造成一定开销，降低gc效率，于是有了“代龄 (age)”及“晋升”。

对象在年青代的3个区 (edge,s0,s1) 之间，每次从1个区移到另1区，年龄+1，在young区达到一定的年龄阈值后，将晋升到老年代。下图中是8，即：挪动8次后，
如果还活着，下次minor GC时，将移动到Tenured区。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/2705a4ba41ed37bd535adab5b91ffb8f.png)

下图是晋升的主要过程：对象先分配在年青代，经过多次Young GC后，如果对象还活着，晋升到老年代。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/6d24d96eb137f805c867736a750c2bd9.png)

如果老年代，最终也放满了，就会发生major GC（即 Full GC），由于老年代的的对象通常会比较多，因为标记 - 清理 - 整理（压缩）的耗时通常会比较长，
会让应用出现卡顿的现象，这也是为什么很多应用要优化，尽量避免或减少Full GC的原因。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/ebad989a70f78a40c561df9943234441.png)

注：上面的过程主要来自oracle官网的资料，但是有一个细节官网没有提到，如果分配的新对象比较大，eden区放不下，但是old区可以放下时，会直接分配到
old区（即没有晋升这一过程，直接到老年代了）。

下图引自阿里出品的《码出高效 -Java 开发手册》一书，梳理了 GC 的主要过程。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/e663bd3043c6b3465edc1e7313671d69.png)


##  垃圾回收器

不算最新出现的神器ZGC，历史上出现过7种经典的垃圾回收器。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/ab1c7be2fa4b5d180ffa1f43bf9dfce7.png)

这些回收器都是基于分代的，把G1除外，按回收的分代划分，横线以上的3种：Serial ,ParNew, Parellel  Scavenge都是回收年青代的，
横线以下的3种：CMS，Serial Old, Parallel Old都是回收老年代的。



垃圾收集算法是内存回收的理论基础，而垃圾收集器就是内存回收的具体实现。
下面介绍一下`HotSpot(JDK 7)`虚拟机提供的几种垃圾收集器，用户可以根据自己的需求组合出各个年代使用的收集器。	

### Serial/Serial Old
`Serial/Serial Old`收集器是最基本最古老的收集器，它是一个单线程收集器，并且在它进行垃圾收集时，
必须暂停所有用户线程。`Serial`收集器是针对新生代的收集器，采用的是`Copying`算法，`Serial Old`收集器是针对老年代的收集器，
采用的是`Mark-Compact`算法。它的优点是实现简单高效，但是缺点是会给用户带来停顿。

### ParNew	

ParNew收集器是Serial收集器新生代的多线程实现，注意在进行垃圾回收的时候依然会stop the world，只是相比较Serial收集器而言它会运行多条进程进行垃圾回收。

ParNew收集器在单CPU的环境中绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不
能百分之百的保证能超越Serial收集器。当然，随着可以使用的CPU的数量增加，它对于GC时系统资源的利用还是很有好处的。它默认开启的收集线程数与CPU的数
量相同，在CPU非常多（譬如32个，现在CPU动辄4核加超线程，服务器超过32个逻辑CPU的情况越来越多了）的环境下，可以使用-XX:ParallelGCThreads参数
来限制垃圾收集的线程数。

### Parallel Scavenge	
Parallel是采用复制算法的多线程新生代垃圾回收器，似乎和ParNew收集器有很多的相似的地方。但是Parallel  Scanvenge收集器的一个特点是它所关注的
目标是吞吐量(Throughput)。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间 / (运行用户代码时间 +  垃圾收集时间)。
停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能够提升用户的体验；而高吞吐量则可以最高效率地利用CPU时间，尽快地完成程序的运算任务，
主要适合在后台运算而不需要太多交互的任务。

### CMS	

全称：Concurrent Mark Sweep，从名字上看，就能猜出它是并发多线程的。这是JDK 7中广泛使用的收集器，有必要多说一下，借一张网友的图说话：

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/e8/4e/e8b152cf510b06544c2a13bfb4fc564e.png)

相对Serial Old收集器或Parallel Old 收集器而言，这个明显要复杂多了，从名字(Mark Swep)就可以看出，CMS收集器是基于标记清除算法实现的。
它的收集过程分为四个步骤：

1. 初始标记(initial mark) 
2. 并发标记(concurrent mark) 
3. 重新标记(remark) 
4. 并发清除(concurrent sweep) 

分为4个阶段：

- Inital Mark 初始标记
    主要是标记GC Root开始的下级（注：仅下一级）对象，这个过程会STW，但是跟GC Root直接关联的下级对象不会很多，因此这个过程其实很快。

- Concurrent Mark 并发标记
    根据上一步的结果，继续向下标识所有关联的对象，直到这条链上的最尽头。这个过程是多线程的，虽然耗时理论上会比较长，但是其它工作线程并不会阻塞，没有STW。

- Remark 再标志
    为啥还要再标记一次？因为第2步并没有阻塞其它工作线程，其它线程在标识过程中，很有可能会产生新的垃圾。
    试想下，高铁上的垃圾清理员，从车厢一头开始吆喝“有需要扔垃圾的乘客，请把垃圾扔一下”，一边工作一边向前走，等走到车厢另一头时，刚才走过的位置上，
    可能又有乘客产生了新的空瓶垃圾。所以，要完全把这个车厢清理干净的话，她应该喊一下：所有乘客不要再扔垃圾了（STW），然后把新产生的垃圾收走。
    当然，因为刚才已经把收过一遍垃圾，所以这次收集新产生的垃圾，用不了多长时间（即：STW 时间不会很长）。

- Concurrent Sweep
    并行清理，这里使用多线程以“Mark Sweep- 标记清理”算法，把垃圾清掉，其它工作线程仍然能继续支行，不会造成卡顿。

等等，刚才我们不是提到过“标记清理”法，会留下很多内存碎片吗？确实，但是也没办法，如果换成“Mark Compact标记 -  整理”法，把垃圾清理后，
剩下的对象也顺便排整理，会导致这些对象的内存地址发生变化，别忘了，此时其它线程还在工作，如果引用的对象地址变了，就天下大乱了。

另外，由于这一步是并行处理，并不阻塞其它线程，所以还有一个副使用，在清理的过程中，仍然可能会有新垃圾对象产生，只能等到下一轮GC，才会被清理掉。
不过由于CMS收集器是基于标记清除算法实现的，会导致有大量的空间碎片产生，在为大对象分配内存的时候，往往会出现老年代还有很大的空间剩余，但是无法找
到足够大的连续空间来分配当前对象，不得不提前开启一次Full GC。

为了解决这个问题，CMS收集器默认提供了一个-XX:+UseCMSCompactAtFullCollection收集开关参数（默认就是开启的)，用于在CMS收集器进行Full GC完
开启内存碎片的合并整理过程，内存整理的过程是无法并发的，这样内存碎片问题倒是没有了，不过停顿时间不得不变长。虚拟机设计者还提供了另外一个参
数-XX:CMSFullGCsBeforeCompaction参数用于设置执行多少次不压缩的FULL GC后跟着来一次带压缩的（默认值为0，表示每次进入Full GC时都进行碎片整理）。

不幸的是，它作为老年代的收集器，却无法与jdk1.4中已经存在的新生代收集器Parallel  Scavenge配合工作，所以在jdk1.5中使用cms来收集老年代的时候，
新生代只能选择ParNew或Serial收集器中的一个。ParNew收集器是使用-XX:+UseConcMarkSweepGC选项启用CMS收集器之后的默认新生代收集器，也可以使用
-XX:+UseParNewGC选项来强制指定它。

虽然仍不完美，但是从这4步的处理过程来看，以往收集器中最让人诟病的长时间STW，通过上述设计，被分解成二次短暂的STW，所以从总体效果上看，应用在GC期间
卡顿的情况会大大改善，这也是CMS一度十分流行的重要原因。

### G1

G1的全称是Garbage-First，为什么叫这个名字，呆会儿会详细说明。鉴于CMS的一些不足之外，比如:老年代内存碎片化，STW时间虽然已经改善了很多，
但是仍然有提升空间。G1就横空出世了，它对于heap区的内存划思路很新颖，有点算法中分治法“分而治之”的味道。G1收集器是一款面向服务端应用的垃圾收集器。
HotSpot团队赋予它的使命是在未来替换掉JDK1.5中发布的CMS收集器。与其他GC收集器相比，G1具备如下特点：

1. 并行与并发：G1能更充分的利用CPU，多核环境下的硬件优势来缩短stop the world的停顿时间。 
2. 分代收集：和其他收集器一样，分代的概念在G1中依然存在，不过G1不需要其他的垃圾回收器的配合就可以独自管理整个GC堆。 
3. 空间整合：G1收集器有利于程序长时间运行，分配大对象时不会无法得到连续的空间而提前触发一次GC。 
4. 可预测的非停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，能让使用者明确指定在一个长度为M毫秒的时间片段内，
    消耗在垃圾收集上的时间不得超过N毫秒。 

在使用G1收集器时，Java堆的内存布局和其他收集器有很大的差别，它将这个Java堆分为多个大小相等的独立区域，虽然还保留新生代和老年代的概念，但是新生
代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。

虽然G1看起来有很多优点，实际上CMS还是主流。


如下图，G1将heap内存区，划分为一个个大小相等（1-32M，2的n次方）、内存连续的Region区域，每个region都对应Eden、Survivor、Old、Humongous
四种角色之一，但是region与region之间不要求连续。

注：Humongous，简称H区是专用于存放超大对象的区域，通常>= 1/2 Region Size，且只有Full GC阶段，才会回收H区，避免了频繁扫描、复制 / 移动大对象。

所有的垃圾回收，都是基于1个个region的。JVM内部知道，哪些region的对象最少（即：该区域最空），总是会优先收集这些 region（因为对象少，内存相对较空，肯定快），这也是 Garbage-First  得名的由来，G 即是 Garbage 的缩写， 1 即 First。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/f1a1bdecf4e56a4440707ce073d73f61.png)

**G1 Young GC**

young GC 前：

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/28b9077c545675b934d64ee643e30a9a.png)

young GC 后：

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/be46c92240c7fd6dfcc6c11a48e4edd9.png)

理论上讲，只要有一个 Empty Region（空区域），就可以进行垃圾回收。

![一文看懂JVM内存布局及GC原理](https://raw.githubusercontent.com/CharonChui/Pictures/master/28ca063660f168b462ed8466faa77e68.png)

由于 region 与 region 之间并不要求连续，而使用 G1 的场景通常是大内存，比如 64G 甚至更大，为了提高扫描根对象和标记的效率，G1 使用了二个新的辅助存储结构：

Remembered Sets：简称 RSets，用于根据每个 region 里的对象，是从哪指向过来的（即：谁引用了我），每个 Region 都有独立的 RSets。（Other Region -> Self Region）。

Collection Sets ：简称 CSets，记录了等待回收的 Region 集合，GC 时这些 Region 中的对象会被回收（copied or moved）。

RSets 的引入，在 YGC 时，将年青代 Region 的 RSets 做为根对象，可以避免扫描老年代的 region，能大大减轻  GC 的负担。注：在老年代收集 Mixed GC 时，RSets 记录了 Old->Old 的引用，也可以避免扫描所有 Old 区。



### ZGC （截止目前史上最好的 GC 收集器）

在 G1 的基础上，做了很多改进（JDK 11 开始引入）

#### 动态调整大小的 Region

G1 中每个 Region 的大小是固定的，创建和销毁 Region，可以动态调整大小，内存使用更高效。

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/4f/03/4fd12cf96cf020e56d071161fa56b603.png)

#### 不分代，干掉了 RSets

G1 中每个 Region 需要借助额外的 RSets 来记录“谁引用了我”，占用了额外的内存空间，每次对象移动时，RSets 也需要更新，会产生开销。

注：ZGC 没有为止，没有实现分代机制，每次都是并发的对所有 region 进行回收，不象 G1 是增量回收，所以用不着  RSets。不分代的带来的可能性能下降，会用下面马上提到的 Colored Pointer && Load Barrier  来优化。

#### 带颜色的指针 Colored Pointer

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/0e/5a/0e071b9c1124d0b7b09150d960c2925a.png)

这里的指针类似 java 中的引用，意为对某块虚拟内存的引用。ZGC 采用了 64 位指针（注：目前只支持 linux 64 位系统），将 42-45 这 4 个 bit 位置赋予了不同的含义，即所谓的颜色标志位，也换为指针的 metadata。

finalizable 位：仅 finalizer（类比 c++ 中的析构函数）可访问；

remap 位：指向对象当前（最新）的内存地址，参考下面提到的 relocation；

marked0 && marked1 位：用于标志可达对象；

这 4 个标志位，同一时刻只会有 1 个位置是 1。每当指针对应的内存数据发生变化，比如内存被移动，颜色会发生变化。

####  读屏障 Load Barrier

传统 GC 做标记时，为了防止其它线程在标记期间修改对象，通常会简单的 STW。而 ZGC 有了 Colored Pointer  后，引入了所谓的读屏障，当指针引用的内存正被移动时，指针上的颜色就会变化，ZGC 会先把指针更新成最新状态，然后再返回。（大家可以回想下  java 中的[ volatile 关键字](https://www.cnblogs.com/yjmyzz/p/6994796.html)，有异曲同工之妙），这样仅读取该指针时可能会略有开销，而不用将整个 heap STW。

#### 重定位 relocation

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/0e/dd/0ee7dc65b940fe54d0b67a217174d2dd.png)

如上图，在标记过程中，先从 Roots 对象找到了直接关联的下级对象 1，2，4。

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/fd/e2/fd2797de4c1ba3b3f8e3b6b871d773e2.png)

然后继续向下层标记，找到了 5，8 对象， 此时已经可以判定 3，6，7 为垃圾对象。

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/95/28/9524b9346adfa6f78283b73b1b201728.png)

如果按常规思路，一般会将 8 从最右侧的 Region 移动或复制到中间的 Region，然后再将中间 Region 的 3  干掉，最后再对中间 Region 做压缩 compact 整理。但 ZGC 做得更高明，它直接将 4，5 复制到了一个空的新 Region  就完事了，然后中间的 2 个 Region 直接废弃，或理解为“释放”，做为下次回收的“新”Region。这样的好处是避免了中间 Region 的 compact 整理过程。

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/7b/22/7b18ada0e14c1fb65eabd73d7760a422.png)

最后，指针重新调整为正确的指向（即：remap），而且上一阶段的 remap 与下一阶段的 mark 是混在一起处理的，相对更高效。

Remap 的流程图如下：

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/8f/c7/8f02e87ca3f90da08edb414e174fd8c7.png)

####  多重映射 Multi-Mapping

这个优化，说实话没完全看懂，只能谈下自己的理解（如果有误，欢迎指正)。虚拟内存与实际物理内存，OS 会维护一个映射关系，才能正常使用。如下图：

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/4c/c4/4ce9d8741ffd7bcde179ed56ac56abc4.png)

zgc 的 64 位颜色指针，在解除映射关系时，代价较高（需要屏蔽额外的 42-45 的颜色标志位）。考虑到这 4 个标志位，同 1  时刻，只会有 1 位置成 1（如下图），另外 finalizable 标志位，永远不希望被解除映射绑定（可不用考虑映射问题）。

所以剩下 3 种颜色的虚拟内存，可以都映射到同 1 段物理内存。即映射复用，或者更通俗点讲，本来 3 种不同颜色的指针，哪怕 0-41 位完全相同，也需要映射到 3 段不同的物理内存，现在只需要映射到同 1 段物理内存即可。

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/ac/49/ac0d5ac0f5f32a7c1c23bde7f513b649.png)

![一文看懂JVM内存布局及GC原理](https://static001.infoq.cn/resource/image/44/2c/445295bc8132bf5aa33fd5aa6cc3c02c.png)

####  支持[ NUMA 架构](https://baike.baidu.com/item/NUMA/6906025?fr=aladdin)

NUMA 是一种多核服务器的架构，简单来讲，一个多核服务器（比如 2core），每个 cpu 都有属于自己的存储器，会比访问另一个核的存储器会慢很多（类似于就近访问更快）。

相对之前的 GC 算法，ZGC 首次支持了 NUMA 架构，申请堆内存时，判断当前线程属是哪个 CPU 在执行，然后就近申请该 CPU 能使用的内存。

**小结**：革命性的 ZGC 经过上述一堆优化后，每次 GC 总体卡顿时间按官方说法 <10ms。注：启用 zgc，需要设置 -XX:+UnlockExperimentalVMOptions -XX:+UseZGC。



# 	与GC相关的常用参数 

除了上面提及的一些参数，下面补充一些和GC相关的常用参数：

- -Xmx: 设置堆内存的最大值。 
- -Xms: 设置堆内存的初始值。 
- -Xmn: 设置新生代的大小。 
- -Xss: 设置栈的大小。 
- -PretenureSizeThreshold: 直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配。 
- -MaxTenuringThrehold: 晋升到老年代的对象年龄。每个对象在坚持过一次Minor GC之后，年龄就会加1，当超过这个参数值时就进入老年代。 
- -UseAdaptiveSizePolicy: 在这种模式下，新生代的大小、eden和survivor的比例、晋升老年代的对象年龄等参数会被自动调整，以达到在堆大小、
    吞吐量和停顿时间之间的平衡点。在手工调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量 (GCTimeRatio) 和
    停顿时间 (MaxGCPauseMills)，让虚拟机自己完成调优工作。 
- -SurvivorRattio: 新生代Eden区域与Survivor区域的容量比值，默认为8，代表Eden: Suvivor= 8: 1。 
- -XX:ParallelGCThreads：设置用于垃圾回收的线程数。通常情况下可以和CPU数量相等。但在CPU数量比较多的情况下，设置相对较小的数值也是合理的。 
- -XX:MaxGCPauseMills：设置最大垃圾收集停顿时间。它的值是一个大于0的整数。收集器在工作时，会调整Java堆大小或者其他一些参数，尽可能地把
    停顿时间控制在MaxGCPauseMills以内。 
- -XX:GCTimeRatio:设置吞吐量大小，它的值是一个0-100之间的整数。假设GCTimeRatio的值为n，那么系统将花费不超过1/(1+n)的时间用于垃圾收集。 



参考: 

- [一文看懂 JVM 内存布局及 GC 原理](https://www.infoq.cn/article/3WyReTKqrHIvtw4frmr3)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 