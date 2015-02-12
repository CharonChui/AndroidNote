MAT内存分析
===

`Eclipse Memory Analyzer（MAT）`是著名的跨平台集成开发环境`Eclipse Galileo`版本的33个组成项目中之一，它是一个功能丰富的`JAVA` 堆转储文件分析工具，可以帮助你发现内存漏洞和减少内存消耗。

[内存泄露介绍]{https://raw.githubusercontent.com/CharonChui/AndroidNote/blob/master/Android%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F.md}

[MAT(Memory Analyzer)官网]{http://www.eclipse.org/mat/}    
- 安装：
    - 单机版，解压后直接使用
    - `Eclipse`插件，直接装一个插件,然后`open perspective`打开 `Memory Analysis`
	
- 使用
    - DDMS
	    进入`DDMS`页面，选择要分析的进程,然后点击`Update Heap`按钮。然后在右侧`Heap`页面点击一下`Cause GC`按钮，点击`Cause GC`按钮就是手动触发`Java`垃圾回收。          
		![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mat_1.png)           
		如果想要看某个`Activity`是否发生内存泄露，可以反复多次进入和退出该页面, 然后点击几次`Cause GC`触发垃圾回收，
		看一下上图中`data object`这一栏中的`Total Size`的大小是保持稳定还是有明显的变大趋势，如果有明显的变大趋势就说明这个页面存在内存泄露的问题，需要进行具体分析。
		
		
 
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
