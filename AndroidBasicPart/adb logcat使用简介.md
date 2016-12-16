adb logcat使用简介
===

得有半年没写日记了，今天不太忙，抽空写一个简单的入门，因为很长时间没用了，最近用起来发现已经忘了.....

`android`开发过程中我们经常会用到`log`，虽然平时大多用`studio`，但是如果测试发现一个`crash`时跑过来找你时你会发现用`studio`已经有点迟了。这时候就要打开`adb logcat`.
忘记怎么加参数了怎么办？ 默默的打开`adb logcat --help`你会发现只要你肯努力，你肯定能把所有的事情都搞垮。

`adb logcat --help`


`Usage: logcat [options] [filterspecs]`


`options include:`
---


 -  `-s`              `Set default filter to silent.         
                    Like specifying filterspec '*:s'设置默认的过滤器,我们想要输出包含"System.out"关键字的标签或信息, 就可以使用adb logcat -s System.out 命令;`   
                       
 - `-f <filename>`   `Log to file. Default to stdout  将日志输出到文件,注意这是手机里的文件，使用adb logcat -f /sdcard/log.txt`         
 - `-r [<kbytes>]`  `Rotate log every kbytes. (16 if unspecified). Requires -f 按照每千字节输出日志，需要与-f一起使用`          
 - `-n <count>`      `Sets max number of rotated logs to <count>, default 4 设置日志输出的最大数目`     
- `-v <format>`     `Sets the log print format, where <format> is one of:         
brief process tag thread raw time threadtime long 设置日志的输出格式, 注意只能设置一项，这里格式比较多，我们最后讲`     

- `-c`              `clear (flush) the entire log and exit     清空所有的日志缓存信息`
- `-d`              `dump the log and then exit (don't block)   将缓存的日志输出到屏幕上, 并且不会阻塞;`
- `-t <count>`      `print only the most recent <count> lines (implies -d)  输出最近的几行日志, 输出完退出, 不阻塞;`
  `-g`              `get the size of the log's ring buffer and exit   查看日志缓冲区信息`
  `-b <buffer>`     `Request alternate ring buffer, 'main', 'system', 'radio' 
                  or 'events'. Multiple -b parameters are allowed and the
                  results are interleaved. The default is -b main -b system.加载一个日志缓冲区, 默认是 main`      
  `-B`              `output the log in binary 以二进制形式输出日志`


`filterspecs are a series of `
---

  `<tag>[:priority]`  过滤项格式为标签:日志等级, 默认的日志过滤项是 ` *:I `;
```
where <tag> is a log component tag (or * for all) and priority is:      

  - V    Verbose
  - D    Debug
  - I    Info
  - W    Warn
  - E    Error
  - F    Fatal
  - S    Silent (supress all output)

'*' means '*:d' and <tag> by itself means <tag>:v

If not specified on the commandline, filterspec is set from ANDROID_LOG_TAGS.
If no filterspec is found, filter defaults to '*:I'默认使用的是*:I级别的。

If not specified with -v, format is set from ANDROID_PRINTF_LOG
or defaults to "brief"
```
例如`adb logcat *:E` 是指定只显示`error`级别的`log`， `adb logcat xxx:D *:S` 是指定只显示标签为`xxx`且`debug`级别以上的`Log`。为什么要加上`*:S`呢，`*:S`用于设置所有标记的日志优先级为`S`，这样可以确保仅输出符合条件的日志。

上面基本的参数都介绍完了。下面说一些常用的功能。  

采用grep正则表达式过滤
---

过滤固定字符串 : 只要命令行出现的日志都可以过滤, 不管是不是标签;

`adb logcat | grep -E '^[VDE]/(TAG1|TAG2)'`

`adb logcat | grep Wifi`
`adb logcat | grep -i wifi 过滤字符串忽略大小写`
`adb logcat | grep --color=auto -i myapp #设置匹配字符串颜色。更多设置请查看 grep 帮助。`

在同时输出到屏幕和文件tee
---  

想要把日志保存到文件,如果采用IO重定向,就无法输出到屏幕, 针对这个问题可以采用tee命令

`adb logcat | grep -E '^[VDE]/(TAG1|TAG2)' | tee my.log`

`adb logcat -v time | tee a.log`



持续输出日志到文件中    
---

`>`输出 : `>` 后面跟着要输出的日志文件, 可以将`logcat`日志输出到文件中

`adb logcat > test.log` //将日志保存到文件test.log

`-d -f <log>` 组合命令：可以将日志保存到手机上的指定位置，对不能一直用电脑连着手机收集日志的场景非常有用。
`adb logcat -d -f /sdcard/mylog.txt`



最后说一下上面的`-v`的格式问题:  
`-v <format>`设置日志输入格式控制输出字段，默认的是`brief`格式:   

- `brief` — 显示优先级/标记和原始进程的PID (默认格式)
- `process` — 仅显示进程PID
- `tag` — 仅显示优先级/标记
- `thread` — 仅显示进程：线程和优先级/标记
- `raw` — 显示原始的日志信息，没有其他的元数据字段
- `time` — 显示日期，调用时间，优先级/标记，PID
- `long` —显示所有的元数据字段并且用空行分隔消息内容
- `adb logcat -v thread`   //使用 thread 输出格式
***注意***`-v` 选项中只能指定一种格式。






---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
