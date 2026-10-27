# 06 Flutter 实现原理

这一章回答几个问题：Flutter 怎么把 Dart 代码变成屏幕上的像素？为什么它能跨平台还接近原生性能？`setState` 之后到底发生了什么？Dart 如何调用到 Kotlin？

## 6.1 整体架构

```
┌──────────────────────────────────────────────────────────────┐
│ Framework（Dart，开源在 flutter/flutter 的 packages/flutter）  │
│   Material / Cupertino           设计语言组件                 │
│   Widgets                        组合、状态、InheritedWidget   │
│   Rendering                      RenderObject：布局与绘制      │
│   Animation / Painting / Gestures                            │
│   Foundation                     基础工具                      │
│   dart:ui                        与引擎交互的最底层 API         │
├──────────────────────────────────────────────────────────────┤
│ Engine（C++）                                                 │
│   Dart Runtime / VM              执行 Dart 代码（JIT 或 AOT）   │
│   Impeller（新）/ Skia（旧）       2D 渲染                      │
│   Text layout（SkParagraph/HarfBuzz/ICU）                     │
│   Platform Channel 消息转发、Isolate 管理、资源加载            │
├──────────────────────────────────────────────────────────────┤
│ Embedder（平台嵌入层，各平台语言实现）                          │
│   Android: Java/Kotlin + JNI   FlutterActivity/FlutterView    │
│   iOS: ObjC/Swift              FlutterViewController           │
│   鸿蒙: ArkTS + NAPI           FlutterAbility/FlutterPage      │
│   负责：提供 Surface、VSync 信号、输入事件、线程、生命周期、    │
│         插件注册、无障碍、输入法                               │
└──────────────────────────────────────────────────────────────┘
                 ↓ 系统只提供“一块画布 + 事件”
┌──────────────────────────────────────────────────────────────┐
│ OS：Android / iOS / OpenHarmony                               │
└──────────────────────────────────────────────────────────────┘
```

**跨平台的本质**：Framework 和 Engine 在各平台上完全相同，只有很薄的 Embedder 不同。对于系统来说，Flutter 页面只是一个 `SurfaceView`/`TextureView`（Android）、`CAMetalLayer`（iOS）、`XComponent`（鸿蒙）。这也是鸿蒙能被快速适配的原因：社区只需要实现一套鸿蒙 Embedder 和插件体系，Framework 层几乎不用改。

## 6.2 三棵树：Widget、Element、RenderObject

```
Widget 树（配置，不可变，每帧可重建）
   │ createElement()
   ▼
Element 树（实例，持久存在，持有 State，是 BuildContext 的真身）
   │ createRenderObject() / updateRenderObject()
   ▼
RenderObject 树（负责布局、绘制、命中测试，重量级，尽量复用）
   │
   ▼
Layer 树（合成层，交给引擎光栅化）
```

用 Android 类比：Widget ≈ 布局 XML 的一个节点描述；RenderObject ≈ 真正的 View；Element ≈ 把二者关联起来、负责 diff 的管理者。

为什么要三棵树？因为声明式 UI 每次状态变化都“重新描述整个界面”，如果每次都重新创建 View 代价太大。Flutter 的方案是：

- 便宜的 Widget 可以随便重建；
- Element 拿新 Widget 与旧 Widget 比较：**`runtimeType` 和 `key` 都相同**（`Widget.canUpdate`）则复用 Element 和 RenderObject，只更新属性；否则销毁子树重建。
- 这就是第 3 章 Key 的作用，也是 `const` Widget 能优化性能的原因（`const` 对象引用相同，Element 直接跳过对它的 rebuild）。

```dart
// Widget.canUpdate 源码（简化）
static bool canUpdate(Widget oldWidget, Widget newWidget) {
  return oldWidget.runtimeType == newWidget.runtimeType
      && oldWidget.key == newWidget.key;
}
```

Element 分两类：

- `ComponentElement`（StatelessElement、StatefulElement）：自己不渲染，只负责调用 `build` 组合子 Widget。
- `RenderObjectElement`：对应一个 RenderObject，例如 `Padding`、`Row`、`RichText`（`Text` 本身是 StatelessWidget，内部 build 出 `RichText`）。

所以你写的 Widget 树比 RenderObject 树大很多，大量 Widget（Container、Text、GestureDetector）只是“组合”，最后真正参与布局绘制的 RenderObject 并不多。

## 6.3 一帧是怎么产生的（渲染流水线）

```
VSync 信号（Android Choreographer / iOS CADisplayLink / 鸿蒙 VSync）
     │
     ▼  UI 线程（Dart，自 3.29 起即平台主线程）
 ┌─────────────────────────────────────────────────────┐
 │ 1. Animate     执行动画 ticker 回调（transient callbacks）│
 │ 2. Build       重建所有 dirty Element（setState 标记的） │
 │ 3. Layout      RenderObject 从上到下传约束、从下到上回尺寸 │
 │ 4. Compositing bits 更新哪些节点需要合成层               │
 │ 5. Paint       RenderObject 把绘制命令记录到 Layer        │
 │ 6. Compositing 生成 LayerTree（Scene）提交给引擎          │
 │ 7. Semantics   生成无障碍树                              │
 │ 8. Finalize    unmount 被移除的 Element                  │
 └─────────────────────────────────────────────────────┘
     │ window.render(scene)
     ▼  Raster 线程（C++）
 ┌─────────────────────────────────────────────────────┐
 │ 9. Rasterize   Impeller 把 LayerTree 转为 GPU 指令      │
 │               （Vulkan / Metal / OpenGL ES），提交显示  │
 └─────────────────────────────────────────────────────┘
```

对应代码入口是 `RendererBinding.drawFrame` / `WidgetsBinding.drawFrame`：

```dart
// WidgetsBinding.drawFrame（简化）
void drawFrame() {
  buildOwner!.buildScope(rootElement);   // Build
  super.drawFrame();                     // ↓ RendererBinding.drawFrame
  //   pipelineOwner.flushLayout();
  //   pipelineOwner.flushCompositingBits();
  //   pipelineOwner.flushPaint();
  //   renderView.compositeFrame();       // 提交 Scene 到引擎
  //   pipelineOwner.flushSemantics();
  buildOwner!.finalizeTree();
}
```

`setState` 的完整链路：

```
setState(fn)
 → fn() 修改字段
 → element.markNeedsBuild()      标记 dirty，加入 BuildOwner 的 dirty 列表
 → SchedulerBinding.scheduleFrame()  向引擎请求下一个 VSync
 → VSync 到来 → handleBeginFrame（动画）→ handleDrawFrame
 → buildScope：按深度排序 dirty 列表，逐个 rebuild
 → 新 Widget 与旧 Widget diff（updateChild）→ 更新 RenderObject 属性
 → RenderObject.markNeedsLayout / markNeedsPaint
 → flushLayout / flushPaint → 合成 → 光栅化
```

60Hz 下一帧预算约 16.6ms，120Hz 下约 8.3ms。**UI 线程和 Raster 线程任何一个超时都会掉帧**，这就是 DevTools 性能面板分别显示 UI 和 Raster 两条柱状图的原因（见第 11 章）。

### 布局为什么快

- 单遍布局：每个 RenderObject 在一帧中只 layout 一次，约束下传、尺寸上传，复杂度 O(n)。Android 中嵌套 `weight` 或 `RelativeLayout` 会导致多次 measure（指数级）。
- Relayout Boundary：当一个节点的尺寸不受子节点影响时（例如约束是 tight 的），它成为“重新布局边界”，子树变化不会向上传播。
- Repaint Boundary：`RepaintBoundary` 会创建独立 Layer，子树重绘时不会影响外部，外部重绘也可以直接复用它的缓存。`ListView` 默认给每个 item 包一层 RepaintBoundary。

## 6.4 渲染引擎：Skia → Impeller

| | Skia（旧） | Impeller（新） |
|---|---|---|
| 着色器 | 运行时按需编译（首次出现某种效果时现场编译） | **构建时预编译**全部着色器 |
| 典型问题 | 首次动画/转场卡顿（shader compilation jank），需要 SkSL warmup 缓解 | 基本消除首帧着色器卡顿 |
| 后端 | OpenGL / Metal / Vulkan | Metal（iOS）、Vulkan（Android），OpenGL ES 作为回退 |
| 现状 | iOS 已移除；Android 仅作历史回退 | iOS 唯一渲染器；Android API 29+ 默认启用，不支持 Vulkan 的设备回退到 OpenGL ES；3.47 起桌面默认 |

Impeller 的设计思路：Flutter 自己掌握所有可能用到的绘制效果（矩形、路径、渐变、模糊、文字等），把它们对应的着色器在编译期全部生成，运行期只做参数填充和 GPU 指令编码，因此帧耗时更稳定、可预测。

调试时可关闭 Impeller 对比（仅用于定位问题）：

```bash
flutter run --no-enable-impeller
```

或在 `AndroidManifest.xml` 中设置 `io.flutter.embedding.android.EnableImpeller` 为 `false`。官方在 2026 roadmap 中计划完成 Android 端到 Impeller 的迁移，长期不要依赖关闭开关。

## 6.5 线程模型

Engine 创建并使用以下 task runner：

| Runner | 作用 | Android 上的对应 |
|---|---|---|
| Platform | 平台消息、插件调用、生命周期、输入事件 | 主线程（Looper） |
| UI | 执行 Dart 代码：build/layout/paint，事件循环 | **Flutter 3.29 起与 Platform 合并为主线程** |
| Raster | 光栅化 LayerTree，提交 GPU | 独立线程 `1.raster` |
| IO | 图片解码、资源加载等 IO 任务 | 独立线程 `1.io` |

**“线程合并”（Great Thread Merge）**：Flutter 3.29 起，Android 和 iOS 上的 Dart 主 Isolate 直接运行在平台主线程上（3.35 起 macOS/Windows 也合并）。影响：

1. 好处：Dart 可以同步调用原生 API（为 FFI 直接调用平台 API、更低延迟的 PlatformView 打基础），减少线程切换。
2. 风险：**Dart 侧的长耗时同步计算会直接阻塞 Android 主线程，可能触发 ANR**（以前只是掉帧）。反过来，原生插件在主线程做耗时操作也会卡住 Dart。所以 CPU 密集任务一定要放 Isolate，原生插件里耗时操作要切到后台线程。
3. 如需临时回退：Android 在 `AndroidManifest.xml` 的 application 下加 `io.flutter.embedding.android.DisableMergedPlatformUIThread` 为 `true`，iOS 在 `Info.plist` 中设置 `FLTEnableMergedPlatformUIThread` 为 `NO`。这是过渡选项，可能被移除。

Raster 线程和 IO 线程仍然独立，所以纯 GPU 压力（大量模糊、阴影、saveLayer）不会阻塞主线程，但会让 Raster 柱状图超标导致掉帧。

## 6.6 Dart 编译与运行：JIT 与 AOT

| 模式 | 编译方式 | 特点 | 用途 |
|---|---|---|---|
| Debug | JIT（Kernel 字节码 + VM 解释/即时编译） | 支持热重载、断言、DevTools 全功能；性能差，**不能用来评估性能** | 开发 |
| Profile | AOT | 接近 Release 性能，保留部分性能分析能力 | 性能分析（必须真机） |
| Release | AOT 编译为机器码 | 最快、最小，关闭断言和调试服务 | 发布 |

Release 构建链路：

```
Dart 源码
  → 前端编译器（CFE）→ Kernel（.dill，中间表示）
  → 全局类型推断 + Tree Shaking（删除未用到的代码）
  → gen_snapshot（AOT 编译器）→ 机器码
  → Android：libapp.so（ELF，含 VM/Isolate snapshot 的 data 与 instructions）
  → iOS：App.framework
  → 鸿蒙：libapp.so（随 HAP 打包）
```

APK 中的关键产物：

- `lib/arm64-v8a/libflutter.so`：引擎（C++）。
- `lib/arm64-v8a/libapp.so`：你的 Dart 代码 AOT 产物。
- `assets/flutter_assets/`：资源、字体、`AssetManifest`、着色器（Impeller）。

Android 的 Flutter release 包**不支持 x86**（32 位），支持 `armeabi-v7a`、`arm64-v8a`、`x86_64`。

因为 Release 是 AOT 机器码，**无法像 JS 那样直接下发 Dart 代码热更新**（苹果政策也不允许下发可执行代码）。动态化方案：Shorebird（Code Push，Dart 解释器补丁）、服务端驱动 UI（如 JSON → Widget）、或 Android 侧替换 `libapp.so`（仅 Android，有合规风险）。

## 6.7 热重载原理

1. 修改代码保存后，工具链只重新编译**发生变化的库**为增量 Kernel 文件。
2. 通过 VM Service 把增量 Kernel 推送到设备上运行中的 Dart VM。
3. VM 重新加载这些库：替换类和函数的定义，**保留堆上已有的对象和状态**。
4. Framework 调用 `reassemble()`，对整棵树执行一次 rebuild，界面使用新代码重新 build。

限制（需要 Hot Restart 或重新运行）：

- 修改 `main()` 或 `initState()` 里的逻辑（已执行过不会重跑）。
- 修改全局变量/静态字段的初始化器（已初始化过）。
- 修改枚举为类、修改泛型声明等结构性改动。
- 修改原生代码（Kotlin/Swift/ArkTS）、`pubspec.yaml` 依赖、资源 —— 需要重新 `flutter run`。

热重载只在 JIT（Debug）模式下可用。

## 6.8 Platform Channel 原理

Dart 不能直接调用 Java/Kotlin。所有 Dart ↔ 原生通信都通过引擎转发的**二进制消息**完成：

```
Dart: MethodChannel('app/battery').invokeMethod('getLevel', args)
  │ MethodCodec（StandardMethodCodec）把方法名+参数编码为 ByteData
  ▼
BinaryMessenger.send(channelName, ByteData)
  │ dart:ui → Engine（C++）PlatformMessage
  ▼
Embedder: Android 通过 JNI 回调 FlutterJNI.handlePlatformMessage
  │ DartMessenger 根据 channelName 找到注册的 Handler
  ▼
Kotlin: MethodCallHandler.onMethodCall(call, result)
  │ result.success(value) → 编码 → 原路返回
  ▼
Dart: Future 完成，拿到返回值
```

三种 Channel：

| Channel | 用途 |
|---|---|
| `MethodChannel` | 方法调用，一问一答（最常用） |
| `EventChannel` | 原生 → Dart 的持续事件流（传感器、下载进度、网络变化），Dart 端是 Stream |
| `BasicMessageChannel` | 双向收发任意消息，可自定义 Codec |

`StandardMessageCodec` 支持的类型：`null`、`bool`、`int`、`double`、`String`、`Uint8List`/`Int32List`/`Float64List` 等、`List`、`Map`。对应 Android 的 `null`、`Boolean`、`Integer/Long`、`Double`、`String`、`byte[]`…、`ArrayList`、`HashMap`。自定义对象需要先转 Map，或使用 Pigeon 生成类型安全代码。

性能特点：

- 每次调用都有编码/解码 + 线程切换（Dart 主线程与平台线程合并后少了一次切换）开销，单次通常在 0.1ms 量级，**不适合每帧高频调用或传输大块数据**（例如逐帧传视频数据）。
- 大数据：用 `Uint8List` 传二进制避免逐元素编码；或用 FFI 共享内存；视频/相机预览用 Texture（外接纹理），数据不经过 Dart。
- 原生侧默认在平台主线程收到调用，耗时操作要自己切线程，回调 `result` 可以在任意线程（Android 引擎会切回主线程）；也可用 `TaskQueue` 让 handler 在后台线程执行。

**FFI（`dart:ffi`）** 是另一条路：Dart 直接调用 C 函数，没有消息编解码，同步调用，开销在纳秒/微秒量级，适合音视频处理、加密、图像算法等已有 C/C++ 库的场景。配合 `ffigen` 从头文件自动生成绑定，配合 `jnigen`/`swiftgen`（实验）还能直接生成调用 Java/Kotlin/ObjC 的绑定。

## 6.9 PlatformView 与外接纹理原理

当需要在 Flutter 页面中嵌入原生控件（地图、WebView、广告 SDK、原生播放器）时：

**PlatformView**：把原生 View 的内容合成到 Flutter 画面中。Android 有几种模式：

| 模式 | 原理 | 优缺点 |
|---|---|---|
| Virtual Display（VD，旧） | 原生 View 渲染到 VirtualDisplay → 纹理 → Flutter 绘制 | 性能尚可，但输入法、无障碍、部分手势有问题 |
| Hybrid Composition（HC） | 原生 View 真实地加入 Android View 层级，Flutter 内容被拆分为多个 Surface 叠在其上下 | 兼容性最好；但 Flutter 帧需要与平台线程同步，性能下降明显，低端机卡 |
| Texture Layer Hybrid Composition（TLHC，默认） | 原生 View 绘制到 `SurfaceTexture`，作为纹理参与 Flutter 合成 | 性能好；`SurfaceView` 类控件可能有问题时自动回退 HC |
| Hybrid Composition++（HCPP，3.44 新增，需手动开启） | 使用 Vulkan hardware buffer swapchain + `SurfaceControl` 事务，由系统合成器同步原生 View 与 Flutter 图层 | 滚动性能和触摸更好、`SurfaceView` 支持更可靠；有 Android API 版本与硬件要求，不满足时不生效 |

iOS 只有一种实现：原生 `UIView` 真实加入视图层级，Flutter 内容拆分为多个 overlay layer，性能较好但多个 PlatformView 叠加时开销会增加。鸿蒙通过 ArkUI 的 `PlatformView` + 自定义组件实现（见第 9 章）。

**外接纹理（Texture）**：原生侧把画面写到一个 `SurfaceTexture`（Android）/ `CVPixelBuffer`（iOS），Flutter 通过 `Texture(textureId: id)` 直接把它作为 GPU 纹理绘制。没有原生 View，不处理触摸，适合视频播放、相机预览、原生渲染引擎输出（`video_player` 和 `camera` 就是这样实现的）。对剪辑类 App，原生特效引擎的预览画面接到 Flutter 最适合用这种方式。

## 6.10 Android 上的启动流程

```
Application.onCreate
  → FlutterLoader.startInitialization()  解压/定位资源，加载 libflutter.so
FlutterActivity.onCreate
  → FlutterActivityAndFragmentDelegate.onAttach
  → 创建 FlutterEngine（或从 FlutterEngineCache 取预热好的引擎）
      → FlutterJNI.attachToNative 创建 C++ Shell：线程、Dart VM、Impeller 上下文
  → GeneratedPluginRegistrant.registerWith(engine)   注册所有插件
  → 创建 FlutterView（FlutterSurfaceView / FlutterTextureView）
  → DartExecutor.executeDartEntrypoint("main")    加载 libapp.so，运行 main()
  → runApp → 第一帧 build/layout/paint
  → 第一帧光栅化完成 → onFlutterUiDisplayed → 移除启动页
```

启动耗时主要在：加载 so、Dart VM 初始化、第一帧构建。混合开发中“打开 Flutter 页面白屏/慢”通常是因为每次都新建 FlutterEngine，解决方法是引擎预热（第 7 章）。

## 6.11 内存模型

- Dart 堆：分代 GC。新生代（scavenger，半空间复制，非常快，适合大量短命 Widget 对象）+ 老生代（并发标记-清除/整理）。GC 发生在主 Isolate 空闲时（利用帧间空闲），对帧率影响很小。
- 原生内存：图片解码后的位图（`ui.Image`）、GPU 纹理、引擎缓存不在 Dart 堆中，Android Studio Profiler 里体现为 Native/Graphics 内存。图片是 Flutter 内存问题的头号来源（第 11 章）。
- 每个 FlutterEngine 实例额外占用约数十 MB（官方数据：首个引擎约 40MB 量级，后续通过 `FlutterEngineGroup` 派生的引擎约 180KB），所以多引擎场景要用 `FlutterEngineGroup`。

## 6.12 小结

| 问题 | 答案 |
|---|---|
| 为什么跨平台一致 | 自绘，只依赖平台提供的画布和事件 |
| 为什么性能好 | AOT 机器码 + 单遍布局 + Widget/Element/RenderObject 分离复用 + GPU 直接渲染 + 预编译着色器 |
| 为什么包大 | 内置引擎、字体渲染、ICU 数据 |
| 为什么混排原生控件麻烦 | 原生 View 不在 Flutter 的渲染树里，需要 PlatformView 合成 |
| 为什么不能热更新 | Release 是 AOT 机器码 |
| 为什么 build 要快 | 它在主线程每帧执行，3.29 后还会阻塞平台主线程 |

## 动手练习

1. 在 Android Studio 打开 Flutter SDK 源码（Cmd+点击 `setState`），沿着 `markNeedsBuild → scheduleFrame → drawFrame` 读一遍。
2. 打开 DevTools → Performance，录制一段滚动，找到 Build、Layout、Paint 各阶段耗时。
3. 用 `debugPrintRebuildDirtyWidgets = true` 观察点击按钮后哪些 Widget 被重建。
4. 在 Android 上 `adb shell ps -T -p <pid>` 查看 Flutter 进程的 `1.raster`、`1.io` 线程。
