AndroidStudio使用教程(第一弹)
===

`Android Studio`是一套面世不久的`IDE`（即集成开发环境），免费向谷歌及`Android`的开发人员发放。`Android Studio`以`IntelliJ IDEA`为基础,旨在取代`Eclipse`和`ADT`（`Android`开发者工具）为开发者提供更好的开发工具。              
运行相应速度、智能提示、布局文件适时多屏预览等都比`Eclipse`要强，但也不能说全部都是有点现在`Studio`中无法在一个窗口管理多个`Project`，每个`Project`都要打开一个窗口，或者是`close`当前的后再打开别的。

当但是毕竟是预览版，所以只是暂时试用了下，并没有过多接触，开发中还是使用`Eclipse`。           
经过一年多的沉淀，如果已到0.8.4版本，最近准备在工作用正式开始使用，所以看了下官网的教程。准备开始了。

- 安装                  
    这个我就不多说了，大家都知道，官网下载安装即可。安装完成后界面和`Eclipse`有些类似，然后就新建一个`Project`，完成之后会发现一直在提示下载，
	这是在下载`Gradle`，大约二三十M的大小，由于伟大的防火墙，所以可能需要很长时间，这里就不教大家了，对程序猿来说不是难题，大家都会科学上网。
	
- 区别              
    - 此`Project`非彼`Project`, `Android Studio`的目录结构(`Project`)代表一个`Workspace`，一个`Workspace`里面可以有多个`Module`，这里`Module`可以理解成`Eclipse`中的一个`Project`.
	`Project`代表一个完整的`Android app`，而`modules`则是`app`的一个组件，并且这个组件可以单独`build,test,debug`。`modules`可以分为下面几种：      
	    - Java library modules    
        - Android library modules: 包含android相关代码和资源，最后生成AAR(Android ARchive)包    
        - Android application modules       
		
    - 结构发生了变化，在`src`目录下有一个`main`的分组同时包含了`java`和`res`.
	    ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_1.png?raw=true)        
	    如图：`MyApplication`就是`Project`，而`app`就是`Module`.
	
- 设置           
   进入后你会发现字体或样式等不符合你的习惯。            
   `Windows`下点击左上角`File` -> `Settings`进入设置页面(`Mac`下为 `Android Studio` -> `Preferences`)，在搜索框搜`Font`找到`Colors&Font`下的`Font`选项，我们会发现无法修改右侧字体大小。这里修改必须
   要通过新建`Theme`进行修改的，点击`Save as`输入一个名字后，就可以修改字体了。
   	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2.png?raw=true)

	这里可能有些人会发现我的主题是黑色的，和`IO`大会演示的一样，但是安装后默认是白色的，有些刺眼。这里可以通过设置页面中修改`Theme`来改变,
	默认是`Intellij`, 改为`Darcula`就是黑色的了.
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_3.png?raw=true)
	很酷有木有.
	
- 运行            
    设置好字体后，当然要走你了。             
	运行和`Eclipse`中比较像，点击绿色的箭头。 可以通过箭头左边的下拉菜单选择不同的`Module`,快捷键是`Shift+F10`                              
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_4.png?raw=true)

    `AndroidStudio`默认安装会启动模拟器，如果想让安装到真机上可以配置一下。在下拉菜单中选择`Edit Configurations`选择提示或者是`USB`设备。
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_5.png?raw=true)	
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_6.png?raw=true)	
	
- 常用快捷键介绍            
    `AndroidStudio`中可以将快捷键设置成`Eclipse`中的快捷键。具体方法为在设置页面搜索`keymap`然后选择为`Eclipse`就可以了.
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_7.png?raw=true)	
	
	强迫症的人伤不起，非想用默认的快捷键。               
	这里我整理下下个人常用的几个快捷键。 每个人的习惯不同，大家各取所需              
	`Ctrl+S`                   开个玩笑，这个键算是彻底废掉了， 因为`AndroidStudio`与`Eclipse`不同，他是自动保存的，所以我们再也不需要`Ctrl+S`了.          
	`Ctrl+空格`                代码提示                    (同`Eclipse`中`Alt+/`)         
	`Ctrl+Shjft+N`             查找文件                    (同`Eclipse`中`Ctrl+Shift+R`)              
	`Ctrl+F12`                 显示当前文件的结构          (同`Eclipse`中`Ctrl+0`)             
	`Ctrl+Alt+L`               格式化                      (同`Eclipse`中`Ctrl+Shift+F`)             
	`Ctrl+Alt+O`               优化导入的包                (同`Eclipse`中`Ctrl+Shift+O`)           
	`Ctrl+Q`                   查看文档                    (同`Eclipse`中`F2`)              
	`Alt+F7`                   搜寻                        (同`Eclipse`中`File Search`)           
	`Alt+Insert`               新建文件或生成代码(`GetSet`)(同`Eclipse`中`Alt+Shift+S后的操作`)             
	`Alt+Enter`                快速修复                    (同`Eclipse`中`F1`)           
	`Ctrl+P`                   显示方法参数                         
	`Shift+F10`                运行项目                               
	`Ctrl+Shift+Backspace`     跳转到上次修改的地方           
	`Ctrl+E`                   显示最近编辑列表             
	`Ctrl+[或]`                跳转到大括号开头或结尾            
	`Ctrl+Alt+T`               把代码包在一起(加try catch等)          
	`Alt+↑或↓`                 在方法间移动            
	`Ctrl+Shift+F7`            高亮本文件中选中的代码            
	`Alt+←或→`                 切换已打开的文件视图               
	`Shift+F6`                 重命名               
	`Ctrl+B`                   快速打开该类或方法                  
	`F2`                       快速定位到文件错误或警告位置              
	`Ctrl+X`                   剪切当前行                
	`Ctrl+Y`                   删除当前行             
	`Ctrl+D`                   复制当前行                            
	`F4`                       进入源码             
	`Alt+1`                    开关`Project`视图      
	`Ctrl+_F4`                 关闭当前窗口            
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 