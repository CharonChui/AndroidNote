AndroidStudio使用教程(第四弹)
===
   
讲解到这里我感觉有必要说明一下`Gradle`。       
`Gradle`是一个基于`Apache Ant`和`Apache Maven`概念的项目自动化建构工具。它使用一种基于`Groovy`的特定领域语言来声明项目设置，而不是传统的`XML`.      
更多介绍请直接参考[Gradle](http://www.gradle.org/)或`Google`搜索。

以下是为什么Android Studio选择Gradle的主要原因：   
- 使用领域专用语言（Domain Specific Language）来描述和处理构建逻辑。（以下简称DSL）
- 基于Groovy。DSL可以混合各种声明元素，用代码操控这些DSL元素达到逻辑自定义。
- 支持已有的Maven或者Ivy仓库基础建设
- 非常灵活，允许使用best practices，并不强制让你遵照它的原则来。
- 其它插件时可以暴露自己的DSL和API来让Gradle构建文件使用。
- 允许IDE集成，是很好的API工具
		
	
	![Image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/AndroidStudio_3_12.png?raw=true)	                    
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 