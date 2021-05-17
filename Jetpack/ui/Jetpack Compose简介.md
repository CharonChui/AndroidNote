# Jetpack Compose简介



Jetpack Compose是用于构建原生Android UI的现代工具包。 Jetpack Compose使用更少的代码，强大的工具和直观的Kotlin API，简化并加速了Android上的UI开发。这是Android Developers 官网对它的描述。



在Android中，UI工具包的历史可追溯到至少10年前。自那时以来，情况发生了很大变化，例如我们使用的设备，用户的期望，以及开发人员对他们所使用的开发工具和语言的期望。

以上只是我们需要新UI工具的一个原因，另外一个重要的原因是`View.java`这个类实在是太大了，有太多的代码，它大到你甚至无法在Githubs上查看该文件，因为它实际上包含了`30000`行代码，这很疯狂，而我们所使用的几乎每一个Android UI 组件都需要继承于View。

GogleAndroid团队的Anna-Chiara表示，他们对已经实现的一些API感到遗憾，因为他们也无法在不破坏功能的情况下收回、修复或扩展这些API，因此现在是一个崭新起点的好时机。

这就是为什么Jetpack Compose 让我们看到了曙光。



作者：依然范特稀西
链接：https://www.jianshu.com/p/19c4e10370ba
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。