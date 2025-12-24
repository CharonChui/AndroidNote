KMP开发
===

Kotlin Multiplatform（简称 KMP）是 JetBrains 推出的开源跨平台开发框架。

它可以通过共享 Kotlin 编写的业务逻辑代码实现多平台复用。


从应用场景来看，KMP 不仅局限于移动端，它支持 iOS、Android、Web、桌面端（Windows/macOS/Linux）以及服务器端的代码共享。这种扩展性使得开发者能够用同一套代码库构建全平台应用，大幅提升开发效率。



KMP 有三大编译目标，分别是： Kotlin/JVM、Kotlin/Native、Kotlin/JS。通过编译不同的目标文件实现各端的跨平台能力。除此之外，KMP 还实验性地支持 WebAssembly（Kotlin/Wasm）编译目标，不过目前实际应用场景相对较少。


### KMP编译器


我们知道，一个语言的编译需要经过词法分析和语法分析，解析成抽象语法树 AST。
而 KMP 为了将 Kotlin 源代码编译成不同的目标平台代码，就需要将 Kotlin 的编译产物进一步向不同的平台转化。
Kotlin 语言的编译，与向不同的平台转化，明显是不同的职责，需要解耦，所以 KMP 的编译器必然有两个部分，也就是编译器前端（Frontend）与编译器后端（Backend）。
Frontend 会将 AST 进一步转换为 Kotlin IR（Kotlin Intermediate Representation），是 Kotlin 源代码的中间表示形式，Kotlin IR 是编译器前端的输出，也是编译器后端的输入。
Backend 则会吧 Kotlin IR 转换为不同平台的中间表示形式，最终生成目标代码。


    	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 