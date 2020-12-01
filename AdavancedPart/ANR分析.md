# ANR分析

Application Not Responding，字面意思就是应用无响应，稍加解释就是用户的一些操作无法从应用中获取反馈


Android系统中的应用被Activity Manager及Window Manager两个系统服务监控着，Android系统会在如下情况展示出ANR的对话框: 
- Service Timeout:比如前台服务在20s内未执行完成；后台服务超过200没有执行
- BroadcastQueue Timeout：比如前台广播在10s内未执行完成，后台60s
- ContentProvider Timeout：内容提供者,在publish过超时10s
- InputDispatching Timeout: 输入事件分发超时5s，包括按键和触摸事件。



ANR信息输出到traces.txt文件中

traces.txt文件是一个ANR记录文件，用于开发人员调试，目录位于/data/anr中，无需root权限即可通过pull命令获取，下面的命令可以将traces.txt文件拷贝到当前目录下
adb pull /data/anr .


1) Thread基础信息

输出种包含所有的线程，取其中的一条
"Thread-1" prio=5 tid=0x00007fde73872800 nid=0x4a03 waiting for monitor entry [0x000000011cb30000]
   java.lang.Thread.State: BLOCKED (on object monitor)
    at Test.rightLeft(Test.java:48)
    - waiting to lock <0x00000007d56540a0> (a Test$LeftObject)
    - locked <0x00000007d5656180> (a Test$RightObject)
    at Test$2.run(Test.java:68)
    at java.lang.Thread.run(Thread.java:745)
a) "Thread-1" prio=5 tid=0x00007fde73872800 nid=0x4a03 waiting for monitor entry [0x000000011cb30000]

首先描述了线程名是『Thread-1』，然后prio=5表示优先级，tid表示的是线程id，nid表示native层的线程id，他们的值实际都是一个地址，后续给出了对于线程状态的描述，waiting for monitor entry [0x000000011cb30000]这里表示该线程目前处于一个等待进入临界区状态，该临界区的地址是[0x000000011cb30000]
这里对线程的描述多种多样，简单解释下上面出现的几种状态

    waiting on condition（等待某个事件出现）
    waiting for monitor entry（等待进入临界区）
    runnable（正在运行）
    in Object.wait(处于等待状态)

作者：silentleaf
链接：https://www.jianshu.com/p/30c1a5ad63a3
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


















---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

