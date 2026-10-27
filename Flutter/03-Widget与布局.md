# 03 Widget 与布局

## 3.1 一切皆 Widget

在 Flutter 里，按钮、文字、间距（`Padding`）、居中（`Center`）、手势（`GestureDetector`）、主题（`Theme`）、甚至整个 App（`MaterialApp`）都是 Widget。Widget 通过嵌套组合构成一棵树。

和 Android View 的根本区别：

| | Android View | Flutter Widget |
|---|---|---|
| 本质 | 有状态的重量级对象，负责测量、布局、绘制、事件 | 不可变的轻量“配置”，只描述 UI 长什么样 |
| 更新方式 | 拿到引用调用 `setText()` | 生成新的 Widget 树，框架做 diff |
| 创建成本 | 高，需要复用 | 极低，每帧都可以重新创建 |
| 继承 vs 组合 | 常靠继承扩展（自定义 View） | 几乎全靠组合 |

Widget 只是配置，真正持有状态和参与布局绘制的是 Element 和 RenderObject（原理见第 6 章）。所以“每次 build 都新建 Widget 对象”不是性能问题，这是设计如此。

## 3.2 StatelessWidget 与 StatefulWidget

```dart
// 无状态：输出只取决于构造参数和 InheritedWidget（如 Theme）
class Avatar extends StatelessWidget {
  const Avatar({super.key, required this.url, this.size = 40});
  final String url;
  final double size;

  @override
  Widget build(BuildContext context) {
    return ClipOval(
      child: Image.network(url, width: size, height: size, fit: BoxFit.cover),
    );
  }
}

// 有状态：Widget 和 State 分离。Widget 重建时 State 对象被保留
class LikeButton extends StatefulWidget {
  const LikeButton({super.key, required this.initialCount});
  final int initialCount;

  @override
  State<LikeButton> createState() => _LikeButtonState();
}

class _LikeButtonState extends State<LikeButton> {
  late int _count = widget.initialCount;   // 通过 widget 访问配置
  bool _liked = false;

  void _toggle() {
    setState(() {
      _liked = !_liked;
      _count += _liked ? 1 : -1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return TextButton.icon(
      onPressed: _toggle,
      icon: Icon(_liked ? Icons.favorite : Icons.favorite_border,
          color: _liked ? Colors.red : null),
      label: Text('$_count'),
    );
  }
}
```

### State 生命周期（对照 Activity/Fragment）

```
createState()
   │
initState()                ≈ onCreate：初始化 Controller、订阅、发首次请求。只调一次
   │                         这里不能用 context 去 dependOnInheritedWidget
didChangeDependencies()    依赖的 InheritedWidget（Theme/MediaQuery/Provider）变化时调用
   │
build()  ◄──────────┐      每次重建调用，必须是纯函数、无副作用、快
   │                │
didUpdateWidget(old)│      父组件用新参数重建本组件时调用（State 被复用）
   │                │      ≈ 在这里对比 old.xxx 与 widget.xxx 决定是否重新请求
setState() ─────────┘
   │
deactivate()               从树上移除（可能被重新插入，如 GlobalKey 移动）
   │
dispose()                  ≈ onDestroy：释放 Controller、取消订阅、关闭 Stream
```

App 级前后台生命周期（≈ `ProcessLifecycleOwner`）：

```dart
class _HomeState extends State<Home> {
  late final AppLifecycleListener _listener;

  @override
  void initState() {
    super.initState();
    _listener = AppLifecycleListener(
      onResume: () => debugPrint('回到前台'),
      onPause: () => debugPrint('进入后台'),
    );
  }

  @override
  void dispose() {
    _listener.dispose();
    super.dispose();
  }
  // ...
}
```

页面级的“可见/不可见”（≈ Fragment onResume），用 `RouteAware` + `RouteObserver`，或第三方 `visibility_detector`。

### 异步回调后使用 context 的坑

```dart
Future<void> _submit() async {
  await api.submit();
  if (!mounted) return;           // State 可能已 dispose，必须检查
  Navigator.of(context).pop();
}
```

在 StatelessWidget 或回调中使用 `context.mounted` 判断。

## 3.3 BuildContext 是什么

`BuildContext` 就是当前 Widget 对应的 Element，代表“我在树中的位置”。`Theme.of(context)`、`Navigator.of(context)`、`MediaQuery.of(context)` 都是从当前位置**向上**查找最近的祖先。

常见错误：在创建 `MaterialApp` 的同一个 `build` 里用 `Navigator.of(context)`，此时 context 在 MaterialApp 之上，找不到 Navigator。解决方法是拆出子 Widget 或用 `Builder`。

## 3.4 布局：约束模型（最重要的一节）

Flutter 布局只有一条规则：

> **约束向下传递，尺寸向上传递，父组件决定子组件位置。**
> Constraints go down. Sizes go up. Parent sets position.

1. 父组件给子组件一个约束 `BoxConstraints(minW, maxW, minH, maxH)`。
2. 子组件在约束范围内决定自己的尺寸，告诉父组件。
3. 父组件决定子组件放在哪里（offset）。

对照 Android：约束 ≈ `MeasureSpec`，但 Flutter 是**单遍布局**（每个节点只 layout 一次，O(n)），不像 `RelativeLayout` / 嵌套 `weight` 那样多次 measure。

几个必须理解的现象：

```dart
// 1. 子组件“想要”的尺寸不一定生效：屏幕给 Container 的是 tight 约束（宽高=屏幕）
runApp(Container(width: 100, height: 100, color: Colors.red)); // 结果铺满全屏

// 2. 用 Center/Align 把 tight 约束放松为 loose，子组件才能用自己的尺寸
runApp(Center(child: Container(width: 100, height: 100, color: Colors.red)));

// 3. Column 在主轴方向给子组件的约束是无限高（unbounded）
//    里面直接放 ListView 会报错 "Vertical viewport was given unbounded height"
Column(children: [
  const Text('标题'),
  Expanded(child: ListView(...)),   // 正确：用 Expanded 给出有限高度
]);

// 4. Row 里放长文本会溢出（黄黑条纹 "RenderFlex overflowed"）
Row(children: [
  const Icon(Icons.info),
  Expanded(child: Text(longText, overflow: TextOverflow.ellipsis)),
]);
```

调试布局：DevTools → Widget Inspector，打开 “Show Guidelines”；或 `debugPaintSizeEnabled = true`。

## 3.5 常用布局组件

| Flutter | Android 近似 | 要点 |
|---|---|---|
| `Container` | 带 background 的 FrameLayout | padding/margin/color/decoration/约束的组合体 |
| `SizedBox` | 固定尺寸 View / Space | `SizedBox(height: 8)` 常作间距 |
| `Padding` | padding 属性 | Flutter 里 padding 是一个独立 Widget |
| `Row` / `Column` | 横/竖 LinearLayout | `mainAxisAlignment`、`crossAxisAlignment`、`mainAxisSize` |
| `Expanded` / `Flexible` | `layout_weight` | Expanded 必须占满，Flexible 可小于分配空间 |
| `Spacer` | weight=1 的空 View | |
| `Stack` + `Positioned` | FrameLayout + margin / 绝对定位 | |
| `Align` / `Center` | gravity | |
| `Wrap` | FlexboxLayout(wrap) | 自动换行 |
| `AspectRatio` | ConstraintLayout ratio | |
| `FittedBox` | scaleType | 缩放子组件以适应 |
| `ConstrainedBox` / `LimitedBox` | min/maxWidth | |
| `LayoutBuilder` | onMeasure 拿父约束 | 响应式布局 |
| `SafeArea` | fitsSystemWindows | 避开刘海/状态栏/手势条 |

一个典型的列表 item：

```dart
class FeedItem extends StatelessWidget {
  const FeedItem({super.key, required this.title, required this.cover, required this.author});
  final String title, cover, author;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          ClipRRect(
            borderRadius: BorderRadius.circular(8),
            child: Image.network(cover, width: 120, height: 68, fit: BoxFit.cover),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(title, maxLines: 2, overflow: TextOverflow.ellipsis,
                    style: theme.textTheme.titleMedium),
                const SizedBox(height: 4),
                Text(author, style: theme.textTheme.bodySmall),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 尺寸单位

Flutter 的 `double` 尺寸单位是逻辑像素（logical pixel），等价于 Android 的 dp。字体大小同样是逻辑像素，但会乘以系统字体缩放（`MediaQuery.textScalerOf(context)`），等价于 sp。物理像素 = 逻辑像素 × `devicePixelRatio`。

## 3.6 基础组件

```dart
Text('Hello', style: TextStyle(fontSize: 16, fontWeight: FontWeight.w600, color: Colors.black87));
Text.rich(TextSpan(children: [            // ≈ SpannableString
  const TextSpan(text: '同意'),
  TextSpan(text: '《用户协议》', style: const TextStyle(color: Colors.blue),
      recognizer: TapGestureRecognizer()..onTap = () {}),  // recognizer 需要在 dispose 中释放
]));

Image.asset('assets/images/logo.png');    // 本地资源，按 2.0x/3.0x 目录自动选分辨率
Image.network(url);                        // 网络图（有内存缓存，无磁盘缓存）
Icon(Icons.home, size: 24);

ElevatedButton(onPressed: () {}, child: const Text('确定'));
TextButton(...); OutlinedButton(...); IconButton(icon: const Icon(Icons.close), onPressed: () {});
// onPressed 传 null 就是禁用态

// 输入框
final _controller = TextEditingController();   // 在 State 中创建，dispose 中释放
TextField(
  controller: _controller,
  decoration: const InputDecoration(hintText: '请输入', border: OutlineInputBorder()),
  keyboardType: TextInputType.phone,
  onChanged: (v) {},
);

// 装饰
Container(
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(12),
    boxShadow: const [BoxShadow(blurRadius: 8, color: Colors.black12)],
    gradient: const LinearGradient(colors: [Colors.blue, Colors.purple]),
  ),
);
```

## 3.7 列表与滚动

```dart
// ≈ RecyclerView：按需构建，滑出屏幕的 item 被销毁
ListView.builder(
  itemCount: items.length,
  itemExtent: 88,                 // 固定高度时务必设置，滚动性能大幅提升
  itemBuilder: (context, index) => FeedItem(...),
);

ListView.separated(itemCount: n, itemBuilder: ..., separatorBuilder: (_, __) => const Divider());

GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 2, mainAxisSpacing: 8, crossAxisSpacing: 8, childAspectRatio: 0.75),
  itemBuilder: ...,
);

// 下拉刷新
RefreshIndicator(onRefresh: _reload, child: ListView.builder(...));

// 上拉加载更多：监听 ScrollController
_scroll.addListener(() {
  if (_scroll.position.pixels > _scroll.position.maxScrollExtent - 300) _loadMore();
});
```

复杂页面（吸顶头部 + 多种列表混排，≈ CoordinatorLayout + 多类型 RecyclerView）使用 Sliver：

```dart
CustomScrollView(
  slivers: [
    const SliverAppBar(pinned: true, expandedHeight: 200,
        flexibleSpace: FlexibleSpaceBar(title: Text('详情'))),
    SliverToBoxAdapter(child: Header()),
    SliverPersistentHeader(pinned: true, delegate: _TabBarDelegate()),  // 吸顶 Tab
    SliverList.builder(itemCount: n, itemBuilder: (c, i) => Item(i)),
    SliverGrid.builder(...),
  ],
);
```

注意：`ListView(children: [...])`、`Column` + `SingleChildScrollView` 会一次性构建全部子项，只适合少量内容。长列表必须用 `.builder`。不要用 `shrinkWrap: true` 嵌套列表来“解决报错”，它会让懒加载失效。

## 3.8 Key

Key 决定重建时 Element 与 Widget 如何对应（diff 时的身份标识），类似 `DiffUtil` 的 `areItemsTheSame`。

- 默认按“类型 + 位置”复用。列表里删除/重排**有状态**的子项时，状态会错位，需要 `ValueKey(item.id)`。
- `GlobalKey`：全局唯一，可以跨位置保留状态、获取 `State` 或 `RenderObject`（如 `formKey.currentState!.validate()`）。开销较大，少用。

```dart
ListView(children: [for (final t in todos) TodoTile(key: ValueKey(t.id), todo: t)]);
```

## 3.9 手势与事件

```dart
GestureDetector(
  onTap: () {},
  onLongPress: () {},
  onPanUpdate: (d) => setState(() => _offset += d.delta),
  behavior: HitTestBehavior.opaque,   // 空白区域也响应点击（默认只有绘制区域响应）
  child: ...,
);

InkWell(onTap: () {}, child: ...);    // 带 Material 水波纹，需要祖先有 Material
```

事件机制：Flutter 先做 hit test 找到命中路径上的所有 RenderObject，再由**手势竞技场（Gesture Arena）**决定哪个识别器胜出（例如嵌套滚动时谁来响应）。它不是 Android 的 `dispatchTouchEvent/onInterceptTouchEvent` 那套分发拦截模型。需要原始指针事件用 `Listener`。

## 3.10 动画

```dart
// 1. 隐式动画：改属性自动过渡（≈ animateLayoutChanges），优先使用
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  width: _expanded ? 300 : 100,
  curve: Curves.easeOut,
);
AnimatedOpacity(...); AnimatedSwitcher(...); AnimatedCrossFade(...);
TweenAnimationBuilder<double>(tween: Tween(begin: 0, end: 1), duration: ..., builder: ...);

// 2. 显式动画：AnimationController ≈ ValueAnimator
class _SpinState extends State<Spin> with SingleTickerProviderStateMixin {
  late final AnimationController _c =
      AnimationController(vsync: this, duration: const Duration(seconds: 1))..repeat();

  @override
  void dispose() { _c.dispose(); super.dispose(); }   // 必须释放

  @override
  Widget build(BuildContext context) =>
      RotationTransition(turns: _c, child: const Icon(Icons.refresh));
}

// 3. 页面共享元素：Hero(tag: id, child: ...)
// 4. 复杂矢量动画：lottie / rive 包
```

`vsync: this` 让动画跟随屏幕刷新信号，页面不可见时自动暂停 ticker，节省电量。

## 3.11 自定义绘制（≈ 自定义 View 的 onDraw）

```dart
class RingPainter extends CustomPainter {
  RingPainter(this.progress);
  final double progress;

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 6
      ..strokeCap = StrokeCap.round
      ..color = Colors.blue;
    final rect = Offset.zero & size;
    canvas.drawArc(rect.deflate(3), -1.5708, 6.2832 * progress, false, paint);
  }

  @override
  bool shouldRepaint(RingPainter old) => old.progress != progress;  // 决定是否重绘
}

CustomPaint(size: const Size(48, 48), painter: RingPainter(0.6));
```

`Canvas` 的 API 与 Android `Canvas` 高度相似（drawPath、drawRRect、saveLayer、clipPath 等），迁移成本很低。

## 3.12 Material 与 Cupertino

- `material.dart`：Material Design 3 组件（Scaffold、AppBar、NavigationBar、Dialog、BottomSheet、SnackBar）。
- `cupertino.dart`：iOS 风格组件（CupertinoNavigationBar、CupertinoSwitch、CupertinoActionSheet）。
- Flutter 3.47 起，Material/Cupertino 已拆分为独立包 `material_ui` / `cupertino_ui`，可通过 `dart fix --apply --code=migrate_design_widgets` 迁移。SDK 内置版本计划在后续稳定版中弃用，新工程建议直接依赖独立包。迁移期间两种 import 都能用，本教程示例仍写 `package:flutter/material.dart`。

大部分国内 App 有自己的设计规范，实践中通常基于 Material 的基础组件封装一套自己的组件库，而不是直接使用 Material 默认样式。

```dart
// 对话框 / 底部弹层 / 提示
final ok = await showDialog<bool>(
  context: context,
  builder: (ctx) => AlertDialog(
    title: const Text('删除？'),
    actions: [
      TextButton(onPressed: () => Navigator.pop(ctx, false), child: const Text('取消')),
      TextButton(onPressed: () => Navigator.pop(ctx, true), child: const Text('删除')),
    ],
  ),
);
showModalBottomSheet(context: context, builder: (_) => const SheetContent());
ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('已保存')));
```

## 动手练习

1. 实现一个“好看视频 feed 流”：顶部吸顶 Tab，下面是 `ListView.builder`，item 为左图右文，支持下拉刷新和上拉加载（数据先用本地 mock）。
2. 故意在 `Column` 里直接放 `ListView`，阅读报错信息，再用 `Expanded` 修复。
3. 用 `CustomPainter` 实现一个圆形进度条，用 `AnimationController` 驱动它从 0 到 100%。
4. 写一个带删除按钮、每项有勾选状态的列表，先不加 Key，删除第一项观察勾选状态错位，再加 `ValueKey` 修复。
