Java内存模型
===

Java虚拟机(Java Virtual Machine)在执行Java程序的过程中，会把它管理的内存划分为五个不同的数据区域(Heap Memory、
Stack Memory、Method Area、PC、Native Stack Memory)，这些区域都有各自的用途、创建时间、销毁时间:     
![](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm_memory_model.jpeg)

## Program Counter Register

程序计数器是一块较小的内存空间，严格来说是一个数据结构，用于保存当前正在执行的程序的内存地址，由于Java是支持多线程执行的，所以程序执行的轨迹
不可能一直都是线性执行。当有多个线程交叉执行时，被中断的线程的程序当前执行到哪条内存地址必然要保存下来，以便用于被中断的线程恢复执行时再按照
被中断时的指令地址继续执行下去。为了线程切换后能恢复到正确的执行位置，每个线程都需要有一个独立的程序计数器，各个线程之间计数器互不影响，独立存储，
我们称这类内存区域为“线程私有”的内存,这在某种程度上有点类似于“ThreadLocal”，是线程安全的。可以看作是当前线程所执行的字节码的行号指示器。

从上面的介绍中我们知道程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

注意：程序计数器是唯一一个不会出现OutOfMemoryError的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。



## Heap Memory

所有的对象实例和数组都存放到Heap内存中。Heap内存也称为共享内存。多线程可以共享这里面的数据。 

- 堆内存在JVM启动的时候被加载(初始大小: -Xms)
- 堆内存在程序运行时会增加或减少
- 最小值: -Xmx
- 从结构上来分，可以分为新生代和老年代。而新生代又可以分为Eden空间、From Survivor空间（s0）、To Survivor空间（s1）。 所有新生成的对象首先都是放在新生代的。需要注意，Survivor的两个区是对称的，没先后关系，所以同一个区中可能同时存在从Eden复制过来的对象，和从前一个Survivor复制过来的对象，而复制到老年代的只有从第一个Survivor区过来的对象。而且，Survivor区总有一个是空的。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm_heap_memory.png)

在并发编程中，多个线程之间采取什么机制进行通信（信息交换），什么机制进行数据的同步？在Java语言中，采用的是共享内存模型来实现多线程之间的信息交换和数据同步的。

## Stack Memory

栈总是与线程关联在一起的，每当创建一个线程，JVM就会为该线程创建对应的线程栈，线程stack中包含有关线程调用了哪些方法以达到当前执行点的信息，也可以称之为调用栈，只要线程执行代码，调用栈就会发生改变。在这个栈中又会包含多个栈帧(Stack Frame)，栈是由一个个栈帧组成，这些栈帧是与每个方法关联起来的，每运行一个方法就创建一个栈帧，每个栈帧会含有一些局部变量表、操作栈和方法返回值等信息。

局部变量表主要存放了编译器可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

每当一个方法执行完成时，该栈帧就会弹出栈帧的元素作为这个方法的返回值，并且清除这个栈帧，Java栈的栈顶的栈帧就是当前正在执行的活动栈，也就是当前正在执行的方法，PC寄存器也会指向该地址。只有这个活动的栈帧的本地变量可以被操作栈使用，当在这个栈帧中调用另外一个方法时，与之对应的一个新的栈帧被创建，这个新创建的栈帧被放到Java栈的栈顶，变为当前的活动栈。同样现在只有这个栈的本地变量才能被使用，当这个栈帧中所有指令都完成时，这个栈帧被移除Java栈，刚才的那个栈帧变为活动栈帧，前面栈帧的返回值变为这个栈帧的操作栈的一个操作数。

线程stack中同样会包含该线程调用栈中所有方法执行所需要的本地变量，一个线程只能获取到它自己对应的线程stack。一个线程创建的本地变量对于其他任何线程都是不可见的。即使两个线程执行完全相同的代码，这两个线程仍然会在各自自己对应的线程stack中创建自己的本地变量。
所有基础类型的局部变量(boolean,byte,short,char,int,long,float,double)都被保存在自己的线程stack中。一个线程可以将一个基础变量的副本传递给另一个线程，但是它不能共享原始局部变量本身。    

堆内存包含在Java应用程序中创建的所有对象，而不管创建该对象的线程是什么。这包括基本类型的对象版本（例如Byte，Integer，Long等）。创建对象并将其分配给局部变量，或者将其创建为另一个对象的成员变量都没有关系，该对象仍存储在堆中。
![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java-memory-model-1.png)

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/stack_heap.png)

- 局部变量可以是原始类型，在这种情况下，它完全保留在线程堆栈中。
- 局部变量也可以是对对象的引用。在这种情况下，引用（局部变量）存储在线程堆栈中，但是对象本身（如果有）存储在堆中。
- 一个对象可能包含方法，而这些方法可能包含局部变量。即使该方法所属的对象存储在堆中，这些局部变量也存储在线程堆栈中。
- 对象的成员变量与对象本身一起存储在堆中。不管成员变量是原始类型时，以及它是对对象的引用时，都是如此。    
- 静态类变量也与类定义一起存储在堆中。    
- 引用对象的所有线程都可以访问堆上的对象。当线程可以访问对象时，它也可以访问该对象的成员变量。如果两个线程同时在同一个对象上调用一个方法，则它们都将有权访问该对象的成员变量，但是每个线程将拥有自己的局部变量副本。

Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError： 

- StackOverFlowError： 若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError异常。
- OutOfMemoryError： 若Java虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

## Method Area

方法区也称”永久代“，它用于存储虚拟机加载的类的信息（名称、修饰符等）、类中的静态变量、类中定义为final类型的常量、类中的属性(Field)信息(属性信息包括属性名、属性类型、属性修饰符(public, private, protected,static,final volatile,transient的某个子集))、类中的方法信息(方法的相关信息包括：方法名， 方法的返回类型(或 void)， 方法参数的数量和类型(有序的)，方法的修饰符(public, private, protected, static, final, synchronized, native, abstract的一个子集))，以及即时编译器编译后的代码(每个方法：字节码，操作数堆栈大小，局部变量大小，局部变量表，异常表；异常表中的每个异常处理程序：起点，终点，处理程序代码的PC偏移，捕获到的异常类的常量池索引)，当在程序中通过Class对象的getName.isInterface等方法来获取信息时，这些数据都来源于方法区。

方法区默认最小值为16MB，最大值为64MB（64位JVM由于指针膨胀，默认是85M），可以通过-XX:PermSize 和 -XX:MaxPermSize 参数限制方法区的大小。它是一片连续的堆空间，永久代的垃圾收集是和老年代(old generation)捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集

方法区是被Java线程锁共享的，不像Java堆中其他部分一样会频繁被GC回收，它存储的信息相对比较稳定，在一定条件下会被GC，当方法区要使用的内存超过其允许的大小时，会抛出OutOfMemory的错误信息。方法区也是堆中的一部分，就是我们通常所说的Java堆中的永久区 Permanet Generation，大小可以通过参数来设置,可以通过-XX:PermSize指定初始值，-XX:MaxPermSize指定最大值。

方法区与堆有很多共性：线程共享、内存不连续、可扩展、可垃圾回收，同样当无法再扩展时会抛出OutOfMemoryError异常。

正因为如此相像，Java虚拟机规范把方法区描述为堆的一个逻辑部分，但目前实际上是与Java堆分开的（Non-Heap）。Non-Heap主要包括的内容如下: 

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm_no_heap.webp)

方法区的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是回收确实是有必要的。

### Constant Pool

常量池也称为运行时常量池(Runtime Constant Pool),用于存放编译期生成的各种字面量和符号引用。

常量池本身是方法区中的一个数据结构。既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

常量池中存储了如字符串、final变量值、类名和方法名常量。常量池在编译期间就被确定，并保存在已编译的.class文件中。一般分为两类：字面量和应用量。字面量就是字符串、final变量等。类名和方法名属于引用量。引用量最常见的是在调用方法的时候，根据方法名找到方法的引用，并以此定为到函数体进行函数代码的执行。引用量包含：类和接口的权限定名、字段的名称和描述符，方法的名称和描述符。

JVM会加载、链接、初始化class文件。一个class文件会把其所有所有符号引用都保留在一个位置，即常量池中。 

每个class文件都会有一个对应constant pool。但是class文件中的常量池显然是不够的，因为需要再JVM上执行。这种情况下，需要runtime constant pool来服务JVM的运行。

Java虚拟机加载的每个类或接口都有其常量池的内部版本，称为运行时常量池(runtime constant pool)。运行时常量池是特定于实现的数据结构，它与类文件中的常量池是一一对应映射的。因此，在最初加载类型之后，该类型的所有符号引用都驻留在该类型的运行时常量池中。包括字符串常量，类和接口名称，字段名称以及类中引用的其他常量。

#### String Constant Pool

在Constant Pool中还有一个单独存放字符串的String Constant Pool。

```java
String str1 = "abcd";
String str2 = new String("abcd");
System.out.println(str1==str2);//false
```
这两种不同的创建方法是有差别的，第一种方式是在常量池中拿对象，第二种方式是直接在堆内存空间创建一个新的对象。
```java
String str1 = "str";
String str2 = "ing";
        
String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象   
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```
尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用StringBuilder或者StringBuffer。

Java 基本类型的包装类的大部分都实现了常量池技术，即Byte,Short,Integer,Long,Character,Boolean；这5种包装类默认创建了数值[-128，127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。
两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。
```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出true
Integer i11 = 333;
Integer i22 = 333;
System.out.println(i11 == i22);// 输出false
Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出false
```

### Method Area & Constant Pool改动

很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已。对于其他虚拟机（如BEA JRockit、IBM J9 等）来说是不存在永久代的概念的。

在JDK1.7中，将字符串常量池和静态变量从方法区域中移到堆中，其余的运行时常量池仍在方法区域中，即hotspot中的永久生成。 所有的被intern的String被存储在PermGen区.PermGen区使用-XX:MaxPermSize=N来设置最大大小，但是由于应用程序string.intern通常是不可预测和不可控的，因此不好设置这个大小。设置不好的话，常常会引起java.lang.OutOfMemoryError: PermGen space。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java7_method_constant_pool.jpg)

在 JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。string constant pool仍然还是在Heap内存中。runtime constant pool也仍然在方法区。但是方法区的实现已经改成使用Metaspace。 ![](https://raw.githubusercontent.com/CharonChui/Pictures/master/java8_memory_method.jpg)

为什么要移除永久代，改为元空间呢？ 

> Metaspace与PermGen之间最大的区别在于：Metaspace并不在虚拟机中，而是使用本机内存。如果没有使用-XX:MaxMetaspaceSize来设置类的元数据的大小，其最大可利用空间是整个系统内存的可用空间。JVM也可以增加本地内存空间来满足类元数据信息的存储。
> 但是如果没有设置最大值，则可能存在bug导致Metaspace的空间在不停的扩展，会导致机器的内存不足；进而可能出现swap内存被耗尽；最终导致进程直接被系统直接kill掉。
>
> 如果类元数据的空间占用达到MaxMetaspaceSize设置的值，将会触发对象和类加载器的垃圾回收。java.lang.OutOfMemoryError: Metaspace space

从用户角度来看，主要的区别是，默认情况下，Metaspace自动增加其大小（达到基础操作系统所提供的大小），而PermGen始终具有固定的最大大小。您可以使用JVM参数为Metaspace设置固定的最大值，但不能使PermGen自动增加。

在很大程度上，这只是名称的更改。早在引入PermGen时，就没有Java EE或动态类的加载（取消加载），因此，一旦加载了一个类，该类便一直停留在内存中，直到JVM关闭为止，从而实现了永久生成。如今，可以在JVM的生命周期内加载和卸载类，因此对于保留元数据的区域，Metaspace更有意义。

## Native Method Stack

本地方法栈和Java栈所发挥的作用非常相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行Native方法服务。如Java使用c或c++编写的接口服务时，代码在此区运行，本地方法栈也会抛出StackOverflowError和OutOfMemoryError异常。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/RDA.png)
先看一张图，这张图能很清晰的说明JVM内存结构的布局和相应的控制参数：
![](https://raw.githubusercontent.com/CharonChui/Pictures/master/jvm_memory_archi_param.jpeg)

## 主内存和工作内存：

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在JVM中将变量存储到内存和从内存中取出变量这样的底层细节。此处的变量与Java编程里面的变量有所不同步，它包含了实例字段、静态字段和构成数组对象的元素，但不包含局部变量和方法参数，因为后者是线程私有的，不会共享，当然不存在数据竞争问题（如果局部变量是一个reference引用类型，它引用的对象在Java堆中可被各个线程共享，但是reference引用本身在Java栈的局部变量表中，是线程私有的）。为了获得较高的执行效能，Java内存模型并没有限制执行引起使用处理器的特定寄存器或者缓存来和主内存进行交互，也没有限制即时编译器进行调整代码执行顺序这类优化措施。

JMM(Java Memory Model)规定了所有的变量都存储在主内存（Main Memory）中。每个线程还有自己的工作内存（Working Memory）,线程的工作内存中保存了该线程使用到的变量的主内存的副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量（volatile变量仍然有工作内存的拷贝，但是由于它特殊的操作顺序性规定，所以看起来如同直接在主内存中读写访问一般）。不同的线程之间也无法直接访问对方工作内存中的变量，线程之间值的传递都需要通过主内存来完成。

由上述对JVM内存结构的描述中，我们知道了堆和方法区是线程共享的。而局部变量，方法定义参数和异常处理器参数就不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响。

Java线程之间的通信由Java内存模型(JMM)控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

假设线程A与线程B之间如要通信的话，必须要经历下面2个步骤: 

1. 线程A把本地内存A中更新过的共享变量刷新到主内存中去。
2. 线程B到主内存中去读取线程A之前已更新过的共享变量。

## 重排

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。

这里说的重排序可以发生在好几个地方：编译器、运行时、JIT等，比如编译器会觉得把一个变量的写操作放在最后会更有效率，编译后，这个指令就在最后了（前提是只要不改变程序的语义，编译器、执行器就可以这样自由的随意优化），一旦编译器对某个变量的写操作进行优化（放到最后），那么在执行之前，另一个线程将不会看到这个执行结果。

当然了，写入动作可能被移到后面，那也有可能被挪到了前面，这样的“优化”有什么影响呢？这种情况下，其它线程可能会在程序实现“发生”之前，看到这个写入动作（这里怎么理解，指令已经执行了，但是在代码层面还没执行到）。通过内存屏障的功能，我们可以禁止一些不必要、或者会带来负面影响的重排序优化，在内存模型的范围内，实现更高的性能，同时保证程序的正确性。

下面我们来看一个重排序的例子：
```java
Class Reordering {
  int x = 0, y = 0;
  public void writer() {
    x = 1;
    y = 2;
  }
  public void reader() {
    int r1 = y;
    int r2 = x;
  }
}
```
假设这段代码有2个线程并发执行，线程A执行writer方法，线程B执行reader方法，线程B看到y的值为2，因为把y设置成2发生在变量x的写入之后（代码层面），所以能断定线程B这时看到的x就是1吗？

当然不行！因为在writer方法中，可能发生了重排序，y的写入动作可能发在x写入之前，这种情况下，线程B就有可能看到x的值还是0。

在Java内存模型中，描述了在多线程代码中，哪些行为是正确的、合法的，以及多线程之间如何进行通信，代码中变量的读写行为如何反应到内存、CPU缓存的底层细节。


## 问题: 匿名内部类访问局部变量时，为什么这个局部变量必须用final修饰? 
这个问题并不是很严谨，严格来说应该是Java 1.8之前，匿名内部类访问局部变量时，才需要用final修饰。 
我们平时经常会用匿名内部类访问局部变量的情况，编译器都会提示我们要对这个局部变量加final修饰，但是我们却并没有去仔细考虑过这是为什么？

上面说到类和成员变量保存到堆内存中。而局部变量则保存在栈内存中。
假设在main()方法中有一个局部变量a，然后main()方法里面又去创建了一个匿名内部类使用该局部变量a。 
a是在栈内存中的，当main()方法执行结束，a就被清理了。但是你创建的内部类中的方法却可能在main()方法执行完成后再去执行，但是这时候局部变量已经不存在了，那怎么解决这个问题呢？ 
因此实际上是在访问它的副本，而不是访问原始的局部变量。

在Java的参数传递中，当基本类型作为参数传递时，传递的是值的拷贝，无论你怎么改变这个拷贝，原值是不会改变的；当对象作为参数传递时，传递的是对象的引用的拷贝，无论你怎么改变这个新的引用的指向，原来的引用是不会改变的（当然如果你通过这个引用改变了对象的内容，那么改变实实在在发生了）。而当final修饰基本类型变量时，不可更改其值，当final修饰引用变量时，不可更改其指向，只能更改其对象的内容。

在Java中内部类会持有外部类的引用和方法中参数的引用，当反编译class文件后，内部类的class文件的构造函数参数中会传入外部类的对象以及方法内局部变量，不管是基本数据类型还是引用变量，如果重新赋值了，会导致内外指向的对象不一致，所以java就暴力的规定使用final，不能重新赋值。
所以用final修饰实际上就是为了变量值(数据)的一致性。 这里所说的数据一致性，对引用变量来说是引用地址的一致性，对基本类型来说就是值的一致性。


当然在JDK 1.8及以后，看起来似乎编译器取消了这种限制，没有被声明为final的变量或参数也可以在匿名内部类内部被访问了。但实际上是因为Java 8引入了effectively final的概念（A variable or parameter whose value is never changed after it is initialized is effectively final）。对于effectively final（事实上的final），可以省略final关键字，本质不变,所以，实际上是诸如effectively final的变量或参数被Java默认为final类型，所以才不会报错，而上述的根本原因没有任何变化。


 It's about the scope of variables , Because anonymous inner classes appear inside a method , If it wants to access the parameters of the method or the variables defined in the method , Then these parameters and variables must be modified to final. Because although anonymous inner classes are inside methods , But when it's actually compiled , Inner classes are compiled into Outer.Inner, This means that the inner class is at the same level as the method in the outer class , A variable or parameter in a method in an external class is just a local variable of the method , The scope of these variables or parameters is only valid inside this method . Because internal classes and methods are at the same level when compiling , So the variables or parameters in the method are only final, Internal classes can be referenced .

---

参考
---
- [Java (JVM) Memory Types](https://javapapers.com/core-java/java-jvm-memory-types/)
- [Java 8: From PermGen to Metaspace](https://dzone.com/articles/java-8-permgen-metaspace)
- [Interview question Series Part 5: JDK runtime  constant pool, string constant pool, static constant pool, are you  stupid and confused?](https://javamana.com/2020/11/20201113132526144q.html)
- [Where Has the Java PermGen Gone?](https://www.infoq.com/articles/Java-PERMGEN-Removed/)


---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

  
