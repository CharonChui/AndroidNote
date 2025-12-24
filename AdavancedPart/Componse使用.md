Compose使用
===

传统 Android UI 开发采用命令式模型，即通过命令驱动视图变化：findViewById 查找控件、设置属性、处理交互逻辑。代码经常伴随着多个职责耦合在一起，结构混乱，易错难测。

Compose 则采用声明式模型：界面即状态的函数表达。当状态改变时，对应的 Composable 自动重新组合（Recompose）并刷新界面。这种模式更贴近现代前端（如 React/Vue）的理念。

无需关心视图更新逻辑，只要状态变化，界面自然重绘，大幅降低 UI 层复杂度。


1.3 开发效率提升点

代码量平均减少 40%-60%

无需 ViewHolder、Adapter 逻辑

状态与 UI 同步更新，避免 UI 状态丢失

支持实时预览（@Preview）、热重载、即时调试


我们在公司项目中构建了性能对比 Benchmark（测试设备为 Pixel 6、Android 13）：

2.1 滚动列表对比：RecyclerView vs LazyColumn

指标  RecyclerView    LazyColumn (Compose)    差异
平均帧率
48 fps
58 fps
+20%
内存占用
28 MB
22 MB
-21%
首次绘制耗时
320 ms
210 ms
-34%


2.2 原因解析：Compose 更快的秘密

SlotTable：结构快照树
Compose 编译器会将 Composable 函数转换为组装 SlotTable 的代码。SlotTable 是一种高效的数据结构，存储了 Composable 树的结构快照。当状态发生变化时，Compose 通过对比 SlotTable 的版本，精确地定位变化范围，从而进行最小代价的重组操作（recomposition）。这一过程通过 Composer 对 Slot 表的操作实现，避免了冗余 UI 节点更新。

重组与 Group 管理机制
Compose 使用 Group（startGroup/endGroup）对 Composable 调用进行打包与标识，每个重组区域会通过重新执行对应的 Group 来进行更新，确保仅变更部分被执行。此机制在 RecomposeScopeImpl 中有体现，它能追踪每个状态依赖的作用域，从而提升重组精度。

无需 ViewHolder 回收
传统 RecyclerView 需要手动管理视图缓存与回收，而 Compose 自动处理 Composition 节点生命周期。Compose Compiler 会生成高效的 Slot 操作指令，通过“skip、reuse”策略对 UI 层进行精准控制，避免重复创建与销毁。

Skia 图形引擎与 RenderNode
Compose 绘制层基于 Skia 引擎，使用 DrawModifier 直接对 Canvas 进行渲染。它不会像传统 View 那样层层嵌套测量布局与绘制流程，而是采用测量（MeasurePass）-> 布局（LayoutPass）-> 绘制（DrawPass）的管线逻辑，通过 LayoutNode 驱动 Compose UI 树的变化。同时 Compose Layout 使用 SubcomposeLayout 实现异步测量能力，提高复杂嵌套组件的性能表现。

渲染流程对比
阶段  View System Compose
布局树管理
View/ViewGroup 层级
LayoutNode 节点
渲染方式
Choreographer + RenderThread
FrameClock + Skia 渲染
状态追踪
手动触发 invalidate
Snapshot 自动追踪 + Diff Patch
更新路径
requestLayout → measure/layout
Recomposer + SlotTable 重组

注意：Compose 并非所有场景都一定更快，特别是复杂嵌套、过度组合场景仍需谨慎使用。

### 简介

Jetpack Compose 是用于构建 Android 界面的新款工具包。


Jetpack Compose 是用于构建原生 Android 界面的新工具包。它使用更少的代码、强大的工具和直观的 Kotlin API，可以帮助您简化并加快 Android 界面开发，打造生动而精彩的应用。它可让您更快速、更轻松地构建 Android 界面。


编写更少的代码会影响到所有开发阶段：作为代码撰写者，需要测试和调试的代码会更少，出现 bug 的可能性也更小，您就可以专注于解决手头的问题；作为审核人员或维护人员，您需要阅读、理解、审核和维护的代码就更少。

与使用 Android View 系统（按钮、列表或动画）相比，Compose 可让您使用更少的代码实现更多的功能。无论您需要构建什么内容，现在需要编写的代码都更少了。以下是我们的一些合作伙伴的感想：

“对于相同的 Button 类，代码的体量要小 10 倍。”(Twitter)
“使用 RecyclerView 构建的任何屏幕（我们的大部分屏幕都使用它构建）的大小也显著减小。”(Monzo)
““只需要很少几行代码就可以在应用中创建列表或动画，这一点令我们非常满意。对于每项功能，我们编写的代码行更少了，这让我们能够将更多精力放在为客户提供价值上。”(Cuvva)
编写代码只需要采用 Kotlin，而不必拆分成 Kotlin 和 XML 部分：“当所有代码都使用同一种语言编写并且通常位于同一文件中（而不是在 Kotlin 和 XML 语言之间来回切换）时，跟踪变得更容易”(Monzo)

无论您要构建什么，使用 Compose 编写的代码都很简洁且易于维护。“Compose 的布局系统在概念上更简单，因此可以更轻松地推断。查看复杂组件的代码也更轻松。”(Square)



Compose 使用声明性 API，这意味着您只需描述界面，Compose 会负责完成其余工作。这类 API 十分直观 - 易于探索和使用：“我们的主题层更加直观，也更加清晰。我们能够在单个 Kotlin 文件中完成之前需要在多个 XML 文件中完成的任务，这些 XML 文件负责通过多个分层主题叠加层定义和分配属性。”(Twitter)


Compose 与您所有的现有代码兼容：您可以从 View 调用 Compose 代码，也可以从 Compose 调用 View。大多数常用库（如 Navigation、ViewModel 和 Kotlin 协程）都适用于 Compose，因此您可以随时随地开始采用。“我们集成 Compose 的初衷是实现互操作性，我们发现这件事情已经‘水到渠成’。我们不必考虑浅色模式和深色模式等问题，整个体验无比顺畅。”(Cuvva)


为现有应用设置 Compose

首先，使用 Compose Compiler Gradle 插件配置 Compose 编译器。

然后，将以下定义添加到应用的 build.gradle 文件中：
```
android {
    buildFeatures {
        compose = true
    }
}
```
在 Android BuildFeatures 代码块内将 compose 标志设置为 true 会在 Android Studio 中启用 Compose 功能。


Compose vs HarmonyOS ArkUI 对比分析


Jetpack Compose 和 HarmonyOS ArkUI 均采用声明式 UI 编程范式，面向多设备场景的响应式 UI 构建，二者在理念相通的同时，在架构设计、状态模型、渲染机制等方面有显著区别。


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/compose_vs_arkui.png?raw=true)        


Jetpack Compose 是围绕可组合函数构建的。这些函数可让您以程序化方式定义应用的界面，只需描述应用界面的外观并提供数据依赖项，而不必关注界面的构建过程（初始化元素、将其附加到父项等）。如需创建可组合函数，只需将 @Composable 注解添加到函数名称中即可。


    	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 