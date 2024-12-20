Gradle 支持使用 Groovy DSL 或 Kotlin DSL 来编写脚本。所以在学习具体怎么写脚本时，我们肯定会考虑到底是使用 Kotlin 来写还是 Groovy 来写。

不一定说你是 Kotlin Android 开发者就一定要用 Kotlin 来写 Gradle，我们得判断哪种写法更适合项目、更适合开发团队人群（学习成本）。

所以下面来学习一下这两种语言的差异。

1. Groovy 和 Kotlin 的差异     

1.1 语言差异

Groovy是一种基于 JVM 的面向对象的编程语言，它可以作为常规编程语言，但主要是作为脚本的语言（为了解决 Java 在写脚本时过于死板）。      

它是一个动态语言，可以不指定变量类型。它的特性是支持闭包，闭包的本质很简单，简单的说就是定义一个匿名作用域，这个作用域内部可以封装函数和变量，外部不可以访问这个作用域内部的东西，但是可以通过调用这个作用域来完成一些任务。
     
Kotlin则是Java的优化版，在解决Kotlin很多痛点的情况下，不引入过多的新概念。它具有强大的类型推断系统，使得语言有良好的动态性，其次是其语言招牌 —— 语法糖，Kotlin 的代码可以写的非常简洁。这使得 Kotlin 不仅做为常规编程语言能大放异彩，作为脚本语言也深受很多开发者喜爱。

它们共同特点就是基于JVM，可以和 Java 互操作。Gradle 能提供的东西， Kotlin 也能通过提供（闭包）。在功能上，两者能做的事情都是一样的。此外一些简单的差异有：

groovy 字符串可以使用单引号，而 kotlin 则必须为双引号。    
groovy 在方法调用时可以省略扩号，而 kotlin 不可省略。    
groovy 分配属性时可以省略 = 赋值运算符，而 kotlin 不可省略。   
groovy 是动态语言，不用导包，而 kotlin 则需要。



为什么要用Kotlin DSL写gradle脚本?     

撇开其他方面，就单从提高程序员生产效率方面就有很多优点：

- 脚本代码自动补全
- 跳转查看源码
- 动态显示注释
- 支持重构(Refactoring)
…
怎么样，要是你经历过groovy那令人蛋疼的体验，kotlin会让你爽的起飞，接下来让我们开始吧。

从Groovy到Kotlin

让我们使用Android Studio 新建一个Android项目，AS默认会为我们生成3个gradle脚本文件。

- settings.gradle (属于 project)
- build.gradle (属于 project)
- build.gradle (属于 module)

我们的目的就是转换这3个文件

- 第一步: 修改groovy语法到严格格式

groovy既支持双引号""也支持单引号''，而kotlin只支持双引号，所以首先将所有的单引号改为双引号。

例如 include ':app' -> include ":app"

groovy方法调用可以不使用() 但是kotlin方法调用必须使用(),所以将所有方法调用改为()方式。

例如:     

```java
implementation "androidx.appcompat:appcompat:1.0.2"
改为

 implementation ("androidx.appcompat:appcompat:1.0.2")
groovy 属性赋值可以不使用=，但是kotlin属性赋值需要使用=，所以将所有属性赋值添加=。
```

例如:     
```
applicationId "com.ss007.gradlewithkotlin"
改为

applicationId = "com.ss007.gradlewithkotlin"
```

完成以上几步，准备工作就完成了。




1.2 文件差异
两者编写 Gradle 的文件是有差异的：

用 Groovy 写的 Gradle 文件是 .gradle 后缀
用 Kotlin 写的 Gradle 文件是 .gradle.kts 为后缀
两者的主要区别是：

代码提示和编译检查
.kts 内所有都是基于kotlin代码规范的，所以强类型语言的好处就是编译没通过的情况下根本无法运行。此外，IDE 集成后可以提供自动补全代码的能力
.gradle 则不会有代码提示和编译检查
源代码、文档查看
.gradle 被编译后是 JVM 字节码，有时候无法查看其源码
.kts 的 DSL 是通过扩展函数实现的（可以看这篇：Kotlin DSL 学习），IDE 支持下可以导航到源代码、文档或重构部分
对于写脚本的人来说，两者的差异不大，因为 Gradle 的 DSL 是 Groovy 提供的，后来的 Kotlin 并没有另起炉灶，而是写了一套 Kotlin 版的。所以两者在代码上也就只有所用语言的差异了，概念啥的都是一样的。

作为一名 Kotlin Android 开发者，我之后基本上是使用 Kotlin DSL 来学习写 Gradle 脚本，但是就跟我上面说的一样，了解其中一个后，要搞懂另外一个成本是很低的。


2. 基本命令
2.1 Project 、 Task 和 Action 介绍
Gradle 主要是围绕着 Project（项目）、Task（任务）、Action（行为）这几个概念进行的。它们的作用分别是：

project：每次 build 可以由一个或多个 project 组成。Gradle 为每个 build.gradle 创建一个相应的 project 领域对象，在编写Gradle脚本时，我们实际上是在操作诸如 project 这样的 Gradle 领域对象。
若要创建多 project 的项目，我们需要在 根工程（root目录）下面新建 settings.gradle 文件，将所有的子 project 都写进去（include）。在 Android 中，每个 Module 都是一个子 project。
task：每个 project 可以由一个或多个 task 组成。它代表更加细化的构建任务，例如：签名、编译一些java文件等。
action：每个 task 可以由一个或多个 action 组成，它有 doFirst{} 和 doLast{} 两种类型
————————————————

                            






























