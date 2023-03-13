# crash分析


## Java Crash流程

1、首先发生crash所在进程，在创建之初便准备好了defaultUncaughtHandler，用来处理Uncaught Exception，并输出当前crash的基本信息；
2、调用当前进程中的AMP.handleApplicationCrash；经过binder ipc机制，传递到system_server进程；
3、接下来，进入system_server进程，调用binder服务端执行AMS.handleApplicationCrash；
4、从mProcessNames查找到目标进程的ProcessRecord对象；并将进程crash信息输出到目录/data/system/dropbox；
5、执行makeAppCrashingLocked：

创建当前用户下的crash应用的error receiver，并忽略当前应用的广播；
停止当前进程中所有activity中的WMS的冻结屏幕消息，并执行相关一些屏幕相关操作；

6、再执行handleAppCrashLocked方法：

当1分钟内同一进程连续crash两次时，且非persistent进程，则直接结束该应用所有activity，并杀死该进程以及同一个进程组下的所有进程。然后再恢复栈顶第一个非finishing状态的activity;
当1分钟内同一进程连续crash两次时，且persistent进程，则只执行恢复栈顶第一个非finishing状态的activity;
当1分钟内同一进程未发生连续crash两次时，则执行结束栈顶正在运行activity的流程。

7、通过mUiHandler发送消息SHOW_ERROR_MSG，弹出crash对话框；
8、到此，system_server进程执行完成。回到crash进程开始执行杀掉当前进程的操作；
9、当crash进程被杀，通过binder死亡通知，告知system_server进程来执行appDiedLocked()；
10、最后，执行清理应用相关的四大组件信息。



- 剩余内存: /proc/meminfo，当系统可用内存小于MemTotal的10%时，非常容易发生OOM和大量GC。
- PSS和RSS通过/proc/self/smap
- 虚拟内存: 获取大小/proc/self/status，获取具体的分布/proc/self/maps。

如果应用堆内存和设备内存比较充足，但还出现内存分配失败，则可能跟资源泄露有关。
- 获取fd的限制数量:/proc/self/limits。一般单个进程允许打开的最大句柄个数为1024，如果超过800需将所有fd和文件名输出日志进行排查。
- 获取线程数大小：/proc/self/status一个线程一般占2MB的虚拟内存，线程数超过400个比较危险，需要将所有tid和线程名输出到日志进行排查。



Native Crash

    崩溃过程：native crash 时操作系统会向进程发送信号，崩溃信息会写入到 data/tombstones 下，并在 logcat 输出崩溃日志
    定位：so 库剥离调试信息的话，只有相对位置没有具体行号，可以使用 NDK 提供的 addr2line 或 ndk-stack 来定位
    addr2line：根据有调试信息的 so 和相对位置定位实际的代码处
    ndk-stack：可以分析 tombstone 文件，得到实际的代码调用栈



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

ANR排查流程
1、Log获取
1、抓取bugreport
adb shell bugreport > bugreport.txt
2、直接导出/data/anr/traces.txt文件
adb pull /data/anr/traces.txt trace.txt
2、搜索“ANR in”处log关键点解读


发生时间（可能会延时10-20s）


pid：当pid=0，说明在ANR之前，进程就被LMK杀死或出现了Crash，所以无法接受到系统的广播或者按键消息，因此会出现ANR


cpu负载Load: 7.58 / 6.21 / 4.83
代表此时一分钟有平均有7.58个进程在等待
1、5、15分钟内系统的平均负荷
当系统负荷持续大于1.0，必须将值降下来
当系统负荷达到5.0，表面系统有很严重的问题


cpu使用率
CPU usage from 18101ms to 0ms ago
28% 2085/system_server: 18% user + 10% kernel / faults: 8689 minor 24 major
11% 752/android.hardware.sensors@1.0-service: 4% user + 6.9% kernel / faults: 2 minor
9.8% 780/surfaceflinger: 6.2% user + 3.5% kernel / faults: 143 minor 4 major


上述表示Top进程的cpu占用情况。
注意
如果CPU使用量很少，说明主线程可能阻塞。
3、在bugreport.txt中根据pid和发生时间搜索到阻塞的log处
----- pid 10494 at 2019-11-18 15:28:29 -----
4、往下翻找到“main”线程则可看到对应的阻塞log
"main" prio=5 tid=1 Sleeping
| group="main" sCount=1 dsCount=0 flags=1 obj=0x746bf7f0 self=0xe7c8f000
| sysTid=10494 nice=-4 cgrp=default sched=0/0 handle=0xeb6784a4
| state=S schedstat=( 5119636327 325064933 4204 ) utm=460 stm=51 core=4 HZ=100
| stack=0xff575000-0xff577000 stackSize=8MB
| held mutexes=
上述关键字段的含义如下所示：

tid：线程号
sysTid：主进程线程号和进程号相同
Waiting/Sleeping：各种线程状态
nice：nice值越小，则优先级越高，-17~16
schedstat：Running、Runable时间(ns)与Switch次数
utm：该线程在用户态的执行时间(jiffies)
stm：该线程在内核态的执行时间(jiffies)
sCount：该线程被挂起的次数
dsCount：该线程被调试器挂起的次数
self：线程本身的地址
