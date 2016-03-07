JVM垃圾回收机制
===

引用计数算法
---

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

根搜索算法
---

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

而手机后的垃圾是通过什么算法来回收的呢：   

- 标记-清除算法
- 复制算法
- 标记整理算法

那我们就继续分析下这三种算法：  

- 标记-清除算法(Mark-Sweep)        
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/biaoji_qingchu.jpg)
    标记-清除算法采用从根集合进行扫描，对存活的对象对象标记，标记完毕后，再扫描整个空间中未被标记  的对象，进行回收，如上图所示。
    标记-清除算法不需要进行对象的移动，并且仅对不存活的对象进行处理，在存活对象比较多的情况下极为高效，但由于标记-清除算法直接回收不存活的对象，因此会造成内存碎片！

- 复制算法(Copying)           
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/fuzhisuanfa.jpg)

    复制算法采用从根集合扫描，并将存活对象复制到一块新的，没有使用过的空间中，这种算法当控件存活的对象比较少时，极为高效，但是带来的成本是需要一块内存交换空间用于进行对象的移动。
 
- 标记-整理算法(Mark-Compact)              
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/biaoji_zhengli.jpg)
 
    整理算法采用标记-清除算法一样的方式进行对象的标记，但在清除时不同，在回收不存活的对象占用的空间后，会将所有的存活对象往左端空闲空间移动，并更新对应的指针。标记-整理算法是在标记-清除算法的基础上，又进行了对象的移动，因此成本更高，但是却解决了内存碎片的问题。

- 分代收集算法(Generational Collection)
    分代收集算法是目前大部分`JVM`的垃圾收集器采用的算法。它的核心思想是根据对象存活的生命周期将内存划分为若干个不同的区域。
	一般情况下将堆区划分为老年代（`Tenured Generation`）和新生代（`Young Generation`），老年代的特点是每次垃圾收集时只有少量对象需要被回收，
	而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。
　　目前大部分垃圾收集器对于新生代都采取复制算法，因为新生代中每次垃圾回收都要回收大部分对象，也就是说需要复制的操作次数较少，
    但是实际中并不是按照1：1的比例来划分新生代的空间的，一般来说是将新生代划分为一块较大的`Eden`空间和两块较小的`Survivor`空间，
	每次使用`Eden`空间和其中的一块`Survivor`空间，当进行回收时，将`Eden`和`Survivor`中还存活的对象复制到另一块`Survivor`空间中，
	然后清理掉`Eden`和刚才使用过的`Survivor`空间。

　　而由于老年代的特点是每次回收都只回收少量对象，一般使用的是标记-整理算法。
　　注意，在堆区之外还有一个代就是永久代（`Permanet Generation`），它用来存储`class`类、常量、方法描述等。对永久代的回收主要回收两部分内容：
    废弃常量和无用的类。         
	![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/xinshengdai.jpg)
	对象的内存分配，往大方向上讲就是在堆上分配，对象主要分配在新生代的`Eden Space`和`From Space`，
	少数情况下会直接分配在老年代。如果新生代的`Eden Space`和`From Space`的空间不足，则会发起一次`GC`，如果进行了`GC`之后，`Eden Space`和`From Space`
	能够容纳该对象就放在`Eden Space`和`From Space`。在`GC`的过程中，会将`Eden Space`和`From  Space`中的存活对象移动到`To Space`，
	然后将`Eden Space`和`From Space`进行清理。如果在清理的过程中，`To Space`无法足够来存储某个对象，就会将该对象移动到老年代中。
	在进行了`GC之`后，使用的便是`Eden space`和`To Space`了，下次`GC`时会将存活对象复制到`From Space`，如此反复循环。
	当对象在`Survivor`区躲过一次`GC`的话，其对象年龄便会加1，默认情况下，如果对象年龄达到15岁，就会移动到老年代中。
	
垃圾收集器
---

垃圾收集算法是内存回收的理论基础，而垃圾收集器就是内存回收的具体实现。
下面介绍一下`HotSpot(JDK 7)`虚拟机提供的几种垃圾收集器，用户可以根据自己的需求组合出各个年代使用的收集器。	
- Serial/Serial Old
    `Serial/Serial Old`收集器是最基本最古老的收集器，它是一个单线程收集器，并且在它进行垃圾收集时，
	必须暂停所有用户线程。`Serial`收集器是针对新生代的收集器，采用的是`Copying`算法，`Serial Old`收集器是针对老年代的收集器，
	采用的是`Mark-Compact`算法。它的优点是实现简单高效，但是缺点是会给用户带来停顿。
	
- ParNew
    `ParNew`收集器是`Serial`收集器的多线程版本，使用多个线程进行垃圾收集。	
	
- Parallel Scavenge	
    `Parallel Scavenge`收集器是一个新生代的多线程收集器（并行收集器），它在回收期间不需要暂停其他用户线程，其采用的是`Copying`算法，
	该收集器与前两个收集器有所不同，它主要是为了达到一个可控的吞吐量。

- Parallel Old
    `Parallel Old`是`Parallel Scavenge`收集器的老年代版本（并行收集器），使用多线程和`Mark-Compact`算法。	
	
- CMS
    `CMS(Current Mark Sweep)`收集器是一种以获取最短回收停顿时间为目标的收集器，它是一种并发收集器，采用的是`Mark-Sweep`算法。	
	
- G1
    `G1`收集器是当今收集器技术发展最前沿的成果，它是一款面向服务端应用的收集器，它能充分利用多`CPU`、多核环境。因此它是一款并行与并发收集器，
	并且它能建立可预测的停顿时间模型。	
	
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 