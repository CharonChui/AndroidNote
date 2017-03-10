AndroidStudio使用教程(第二弹)
===

- 迁移`Eclipse`工程到`Android Studio`            

    官方文档中说`Android Studio`可以兼容`Eclipse`的现有工程，但需要做一些操作：              
	
    - `Eclipse`进行项目构建           
	    首先升级`ADT`到最新版本, 好像是22之后，选择需要从`Eclipse`导出的工程，右键选择`Export`并选择`Android`下的`Generate Gradle Build Files`, 
		运行完成之后你会发现在项目目录中多了一个`build.gradle`, 这就是`Android Studio`所识别的文件。      
		PS：官方文档中说明如果没有`Grade build`文件，也是可以将项目导入到`Android Studio`中，它会用现有的`Ant build`文件进行配置.
		但为了更好地使用之后的功能和充分使用构建变量，
		还是强烈地建议先从`ADT`插件中生成`Gradle`文件再导入`Android Studio`.
    
	- 导入            
	    在`Android Studio`中选择`Import Project`,并选择刚才工程目录下的`build.gradle`即可。           

		有些时候会发现导入之后在运行按钮左边显示不出`Module`来，可能是你导入之前的`SDK`版本不同导致的，只要在`build.gradle`中配置相应的`SDK`版本就可以了。           
		```java
		android {
			compileSdkVersion 19
			buildToolsVersion "21.1.1"
			...
			}
		```
	
- 创建工程     
    创建工程和`Eclipse`流程基本差不多，大家一看就明白了，这里就不说了。
	
- 使用`Android`项目视图
    这里纯粹看个人爱好，不过对于标准的`AndroidStudio`工程，一般我们常用的部分，都在`Android`项目视图中显示出来了。      
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_1.png?raw=true)             
	效果如下图：     
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_2.png?raw=true)               
	
- 使用布局编辑器             

    布局编辑器的适时和多屏幕预览的确是一个亮点。     
	
	- 点击布局页面右侧的`Preview`按钮，可以进行预览。  
	
	    想预览多屏幕效果时可以在预览界面设备的下拉菜单上选择`Preview All Screen Sizes`.     
	    ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_3.png?raw=true)       
		
	- 选择主题
	
	    想给应用设置一个主题，可以点击`Theme`图标![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_4.png?raw=true), 
		就会显示出选择对话框。       
		![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_5.png?raw=true)
	
	- 国际化
	    对于国际化的适配选择国际化图标![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_6.png?raw=true),
		然后在弹出的列表中选择需要进行国际化的国家进行适配即可。

- 常用功能
    有些人进来之后可能找不到`DDMS`了.      
	![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/AndroidStudio_2_7.png?raw=true)	   
	上图中的三个图标分别为, `AVD Manager`、`SDK Manager`、`DDMS`
	

		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 