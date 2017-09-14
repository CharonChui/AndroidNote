AndroidStudio使用教程(第六弹)
===

Debug
---

`Andorid Studio`中进行`debug`:      
- 在`Android Studio`中打开应用程序。    
- 点击状态栏中的`Debug`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_1.png?raw=true)图标。
- 在接下来的选择设备窗口选择相应的设备或创建虚拟机， 点击`OK`即可。     
`Android Studio`在`debug`时会打开`Debug`工具栏， 可以点击`Debug`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_2.png?raw=true)图标打开`Debug`窗口。    

###设置断点 
与`Eclipse`十分相似， 在代码左侧位置点击一下即可， 圆点的颜色变了。   

###Attach the debugger to a running process 
在`debug`时不用每次都去重启应用程序。 我们可以对正在运行的程序进行`debug`:  
- 点击`Attach debugger to Android proccess`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_3.png?raw=true)图标。  
- 设备选择窗口选择想要`debug`的设备。 
- 点击`Debug`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_2.png?raw=true)图标打开`Debug`工具栏。    
- 在`Debug`工具栏中有`Stop Over`, `Stop Into`等图标和快捷键，这些就不仔细说明了, 和`Eclipse`都差不多。     

### View the system log
在`Android DDMS`中和`Debug`工具栏中都可以查看系统`log`日志，
- 在窗口底部栏点击`Android`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_4.png?raw=true) 图标打开`Android DDMS`工具栏。   
- 如果此时`Logcat`窗口中的日志是空得，点击`Restart`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_5.png?raw=true)图标。 
- 如果想要只显示当前某个进程的信息点击`Only Show Logcat from Selected Process`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_6.png?raw=true). 如果当时设备窗口不可见，点击右上角的`Restore Devices View`![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_7.png?raw=true)图标，该图标只有设备窗口不可见时才会显示。    

删除`Project`及`Module`
---

很多人都在问`AndroidStudio`中如何删除`Project`，如何删除`Module`？怎么和`Eclipse`不同啊，找不到`delete`或`remove`选项。       
    - 删除`Project`       
        点击左侧`File`-->`Close project`，关闭当前工程， 然后直接找到工程所在本地文件进行删除(慎重啊), 删除完之后点击最近列表中的该项目就会提示不存在，我们把他从最近项目中移除即可.![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_11.png?raw=true)你会发现，点击`remove`之后没效果，以后估计会解决。      
    - 删除`Module`             
        在该`Module`邮件选择`Open Module Settings`。            
		![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_8.png?raw=true)                
        进入设置页后选中要删除的`Module`点击左上角的删除图标`-`后点击确定。                                
		![Image](https://github.com/CharonChui/Pictures/blob/master/AndroidStudio_6_9.png?raw=true)
		
        	  	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 