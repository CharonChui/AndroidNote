Android Studio你可能不知道的操作
===

今天看在`Youtube`看视频，看到`Reto Meier`在讲解`Studio`，
一查才知道他现在是`Studio`的开发人员。
想起刚开始学`Android`时买的他写的书`Professional Android 4 Application Development`，
当时很多内容没看懂。不过看了这个视频才发现大神写代码如此之快…

现在天天用着大神开发的工具，没有理由不去好好学习下。

工欲善其事必先利其器，一个好的开发工具可以让你事半功倍。`Android Studio`是一个非常好的开发工具，但是虽然都在用，你可能还是了解的不全面，今天就来说一下一些你可能不知道的功能。

熟练使用快捷键是非常有必要的:     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/use_shortcut.gif?raw=true)  


- 自动导入              
经常听到同事抱怨，`Studio`怎么没有`Eclipse`那种批量导包啊，那么多类要到，费劲了。其实不用一个个导的。
使用`Command+Shift+A(Windows或Linux是Ctrl+Shift+A)`快速的查找设置命令。我们输入`auto import`后将`Add unambiguous imports on fly`选项开启就好了，很爽有木有？你的是不是也没开啊？              
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/auto_import.gif?raw=true)         
你可以快速打开一些设置，例如你想在`finder`中查看该文件，直接输入`find`就好了。
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/action_find.png?raw=true)        

- 在自动完成代码时使用`Tab`键来替换已存在的方法和参数。           
经常如我们想要修改一个已经存在的方法时，我们移动到对象的`.`的提示区域，如果使用`Command+Space`来补充代码选择后按`enter`的话，会把之前已经存在的代码往后移动，这样还要再去删除，很不方便。但是如果我们直接使用`Tag`键，那就会直接替换当前已经存在的方法。
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/tab_tip.gif?raw=true)  

- 内容选择技巧         
使用`Control+↑或↓`能在方法间移动。使用`Shift+↑或↓`来选择至上一行或者至下一行代码的代码？那么使用`option++↑或↓(Windows或Linux是alt++↑或↓)`呢？它能选择或者取消选择和你鼠标所在位置的代码区域。同时使用`option+shift++↑或↓(Windows或Linux是alt+shift++↑或↓)可以交换选中代码块与上一行的位置。这样我们就不需要剪切和粘贴了。

- 使用模板补全代码      
你可以在代码后加用后缀的方式补充代码块，也可以用`Command+J(Windows是Ctrl+J)`来提示。      
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/command_j.gif?raw=true)        
对于一些更多的模式，在代码自动完成时也支持生成对应的模板，例如使用`Toast`的快捷键可以很方便的生成一个新的`toast`对象，你只需要指定文字就可以了。     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/toast_autocomp.gif?raw=true)         
用`Command+Shift+A`然后输入`live template`然后打开对应的页面，可以看到目前已经存在的模板。当然你也可以添加新的模板。     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/live_templete.png?raw=true)  

- 在`Evaluating Expressions`中为`Object`对象指定显示内容        
如果我们在`debug`的时候查看断点处的变量值或者`evaluating expressions`，你会发现`objects`会显示他们的`toString()`值。如果你的变量是`String`类型或者基础类型那不会有问题，但是大多数其他对象，这样没有什么意义。    
尤其是在集合对象时，你看的是一个`CallsName:HashValue`的列表。而为了需要看清数据，我们需要知道每个对象的内容。     
你当然可以去对每个对象类型指定索要显示的内容。在对应的对象上邮件`View as`然后创建你想要显示的内容就可以了。     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/debug_eva.gif?raw=true)  

- 结构性的搜索、替换、和检查       
`Command+shift+A`然后输入`structural `后选择`search structurally`或者`replace structurally`，然后可以对应的结构性搜索的模板，完成之后所有符合该模板的代码都会提示`warning`。     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/structural_search.gif?raw=true)        
更有用的是可以使用快速修复来替换一些代码，例如一些废弃的代码或者你在`review`时发现的其他团队成员提交的一些普遍的错误。

    	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 