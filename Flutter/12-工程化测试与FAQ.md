# 12 工程化、测试、FAQ 与自测

## 12.1 代码规范与静态检查

`analysis_options.yaml`：

```yaml
include: package:flutter_lints/flutter.yaml   # 或更严格的 very_good_analysis

analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
  errors:
    missing_required_param: error
    missing_return: error

linter:
  rules:
    prefer_const_constructors: true
    prefer_const_declarations: true
    avoid_print: true                # 用 debugPrint 或日志库
    use_build_context_synchronously: true   # async gap 后使用 context 时报警
    cancel_subscriptions: true
    close_sinks: true
    always_declare_return_types: true
```

```bash
dart format .                  # 格式化（≈ ktlint format），CI 中用 dart format --set-exit-if-changed .
flutter analyze                # 静态检查
dart fix --apply               # 自动修复可修复的 lint 和 API 迁移
```

## 12.2 测试

| 类型 | 对应 Android | 运行环境 | 速度 |
|---|---|---|---|
| 单元测试 `test()` | JUnit | Dart VM（本机） | 快 |
| Widget 测试 `testWidgets()` | Robolectric + Espresso | 本机无设备，模拟渲染 | 快 |
| 集成测试 `integration_test` | Espresso / UIAutomator 真机 | 真机/模拟器 | 慢 |
| Golden 测试 | 截图测试（Paparazzi） | 本机 | 快 |

```dart
// test/feed_notifier_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

class MockRepo extends Mock implements FeedRepository {}

void main() {
  test('首次加载返回数据', () async {
    final repo = MockRepo();
    when(() => repo.fetch(page: 1)).thenAnswer((_) async => [Feed(id: '1', title: 't', cover: 'c')]);
    final container = ProviderContainer(overrides: [feedRepoProvider.overrideWithValue(repo)]);
    addTearDown(container.dispose);

    final list = await container.read(feedProvider.future);
    expect(list, hasLength(1));
  });

  testWidgets('点赞后数字加 1', (tester) async {
    await tester.pumpWidget(const MaterialApp(home: Scaffold(body: LikeButton(initialCount: 9))));
    expect(find.text('9'), findsOneWidget);
    await tester.tap(find.byType(TextButton));
    await tester.pump();                       // 触发一帧
    expect(find.text('10'), findsOneWidget);
  });
}
```

```bash
flutter test                                  # 全部
flutter test --coverage                       # 覆盖率 coverage/lcov.info
flutter test integration_test -d <device>     # 集成测试
flutter test --update-goldens                 # 更新 golden 截图
```

Mock 平台 Channel（单元测试中没有原生侧）：

```dart
TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
    .setMockMethodCallHandler(const MethodChannel('com.example/battery'), (call) async {
  if (call.method == 'getLevel') return 80;
  return null;
});
```

## 12.3 多环境与多渠道

```bash
# 编译期常量（≈ BuildConfig 字段）
flutter run --dart-define=ENV=staging --dart-define=API_HOST=https://staging.example.com
flutter build apk --dart-define-from-file=env/prod.json
```

```dart
const env = String.fromEnvironment('ENV', defaultValue: 'dev');
const apiHost = String.fromEnvironment('API_HOST');
```

Flavor（≈ productFlavors，同时配置原生侧的包名、图标、名称）：

```bash
flutter run --flavor staging -t lib/main_staging.dart
```

Android 侧在 `build.gradle.kts` 中定义同名 `productFlavors`；iOS 侧创建同名 Scheme 和 Build Configuration。

## 12.4 错误监控与日志

```dart
void main() {
  // Flutter 框架内部的错误（build/layout/paint 异常）
  FlutterError.onError = (details) {
    FlutterError.presentError(details);
    crashReporter.recordFlutterError(details);
  };
  // 未被捕获的异步错误
  PlatformDispatcher.instance.onError = (error, stack) {
    crashReporter.recordError(error, stack);
    return true;
  };
  // Release 中替换红屏
  ErrorWidget.builder = (details) => const Center(child: Text('页面出错了'));

  runApp(const App());
}
```

- Release 包使用 `--obfuscate --split-debug-info` 时，崩溃栈需要上传符号文件到崩溃平台，或用 `flutter symbolize -i stack.txt -d build/symbols/app.android-arm64.symbols` 还原。
- Dart 异常不会导致 App 崩溃（只是当前帧/当前操作失败），所以必须主动上报，否则线上问题“静默”发生。
- 原生崩溃（插件、引擎）仍由原生崩溃 SDK 捕获。

## 12.5 CI/CD

```yaml
# 示例流程（以通用 CI 语法描述）
steps:
  - flutter --version               # 用 FVM 或固定镜像锁定版本
  - flutter pub get
  - dart format --set-exit-if-changed .
  - flutter analyze
  - flutter test --coverage
  - flutter build apk --release --split-per-abi --obfuscate --split-debug-info=symbols
  - flutter build ipa --release     # macOS runner
  - flutter build hap --release     # 使用 Flutter OH SDK 的 runner
  - 上传产物与 symbols
```

注意：鸿蒙流水线需要单独的 Flutter OH SDK 与 DevEco 命令行工具（ohpm、hvigor），通常放在单独的构建机/镜像中。

## 12.6 调试技巧

| 需求 | 方法 |
|---|---|
| 断点调试 Dart | IDE 中 Debug 运行；`debugger()` 语句 |
| 调试原生 + Dart | Android Studio 先 `flutter run`，再 Attach Debugger to Android Process（选 Java/Kotlin） |
| 连接已运行的 App | `flutter attach`（Add-to-App 场景，原生启动后附加 Dart 调试） |
| 日志 | `debugPrint`（自动限速、避免丢日志）；Android `adb logcat -s flutter`；鸿蒙 `hdc hilog` |
| 查看布局 | Widget Inspector、`debugDumpRenderTree()` |
| 网络抓包 | dio 中配置代理，或 `LogInterceptor` |
| 性能 | Profile 模式 + DevTools（第 11 章） |

## 12.7 FAQ

**Q：Flutter 页面能直接使用 Android 已有的 View/自定义控件吗？**
可以，通过 PlatformView 嵌入，但有性能代价。纯展示类的建议用 Flutter 重写；视频/渲染类用 Texture。

**Q：已有的 Kotlin 业务逻辑（网络库、数据库、编辑引擎）能复用吗？**
能，通过 Pigeon/MethodChannel 暴露给 Dart。复用的代价是两端序列化和异步调用。复杂而稳定的底层能力（音视频引擎、特效 SDK）适合复用；简单的业务逻辑一般用 Dart 重写更好，因为它们需要在 iOS 和鸿蒙上也能运行。

**Q：Flutter 能做热更新吗？**
官方不支持。可选 Shorebird（Code Push）、服务端驱动 UI、或者关键页面保留 H5。iOS 需遵守审核条款。

**Q：`build` 方法会被调用很多次，会不会很慢？**
Widget 创建本身很便宜，Element 复用使得真正的布局/绘制只发生在变化的部分。慢的是“build 中做了不该做的事”和“重建范围太大”。

**Q：为什么修改代码后热重载没有生效？**
修改的是 `initState`/`main`/全局变量初始化，或原生代码/依赖/资源。按 `R` 热重启或重新运行。

**Q：Flutter 的 Dart 代码是在主线程运行吗？**
是（3.29 起与平台主线程合并）。网络 IO 不会阻塞主线程，但 CPU 密集计算会，需要 Isolate。

**Q：怎么判断一个 pub 包能否在鸿蒙上使用？**
纯 Dart 包可以直接用；含 `android/`、`ios/` 原生代码的插件需要查看社区是否有 ohos 适配版本（第 9 章）。

**Q：三端只维护一套代码，能做到多少？**
UI 和业务逻辑可以做到 90%+ 共用；平台差异集中在插件层、少量 UI 差异和工程配置。鸿蒙需额外维护 SDK 版本差异和插件适配。

**Q：团队里 Android 和 iOS 开发者各自需要掌握什么？**
人人掌握 Dart + Flutter 框架 + 状态管理；各平台开发者负责对应平台的插件、工程配置、发布和平台问题排查；至少一人深入理解渲染原理与性能工具。

## 12.8 面试自测题

基础：

1. `StatelessWidget` 与 `StatefulWidget` 的区别？`State` 为什么要和 Widget 分开？
2. `setState` 调用后到画面更新经历了哪些步骤？
3. 解释“约束向下、尺寸向上、父决定位置”。为什么 `Column` 里直接放 `ListView` 会报错？
4. `final` 和 `const` 的区别？为什么 `const` Widget 能提升性能？
5. `Expanded` 和 `Flexible` 的区别？
6. `BuildContext` 是什么？为什么在 `MaterialApp` 同级 `build` 中 `Navigator.of(context)` 找不到？
7. Key 的作用？什么时候必须使用 `ValueKey`？

原理：

8. Widget、Element、RenderObject 三棵树的职责与关系？`Widget.canUpdate` 的判断条件？
9. Flutter 为什么能跨平台？它与 React Native 在渲染上的根本区别？
10. UI 线程和 Raster 线程分别负责什么？如何判断卡顿发生在哪一个？3.29 的线程合并带来什么影响？
11. Impeller 解决了 Skia 的什么问题？
12. Debug/Profile/Release 三种模式的编译方式与用途？
13. 热重载的原理与限制？
14. MethodChannel 的通信过程？`StandardMessageCodec` 支持哪些类型？`result` 回复多次会怎样？
15. PlatformView 的几种实现模式和各自优缺点？什么时候用 Texture？

实战：

16. Isolate 和线程的区别？什么时候用 `Isolate.run`？
17. 一个长列表滑动卡顿，你的排查步骤？
18. 列表图片导致内存暴涨，怎么解决？
19. 在已有 Android App 中接入 Flutter，打开 Flutter 页面白屏 1 秒，怎么优化？
20. 原生 A → Flutter B → 原生 C → Flutter D 的混合栈怎么设计？
21. 一个使用了 `camera`、`shared_preferences` 和自研 Channel 的 App 要支持鸿蒙，需要做哪些工作？
22. 如何在一套代码中同时兼容官方 Flutter 与 Flutter OH？

## 12.9 综合练手项目

做一个“迷你短视频 App”，三端运行（Android 必做，iOS/鸿蒙选做）：

| 功能 | 涉及知识 |
|---|---|
| 首页 feed 流：吸顶 Tab、下拉刷新、上拉加载、图片缓存 | 第 3、5、11 章 |
| 视频详情：`video_player` 播放、点赞、评论输入框 | 第 3、10 章 |
| 登录：手机号 + 验证码，登录态拦截 | 第 4、5 章 |
| 我的：深色模式切换、缓存大小、清理缓存 | 第 5 章 |
| 原生能力：Pigeon 获取视频元信息、原生 Toast | 第 7、8、9 章 |
| 工程化：Riverpod/Bloc 分层、单测、Widget 测试、flavor、错误上报 | 第 4、12 章 |
| 性能：Profile 下列表 60fps，图片限制解码尺寸，无内存泄漏 | 第 11 章 |
| 混合：把“我的”页面以 Add-to-App 方式接入一个原生 Android 壳 | 第 7 章 |

验收标准：`flutter analyze` 无告警；核心逻辑有测试；在一台低端 Android 真机 Profile 模式下滑动 feed 流无明显掉帧；能回答 12.8 中的全部问题。

## 12.10 参考资料

- 官方文档：<https://docs.flutter.dev>（含 “Flutter for Android developers”）
- Dart 语言：<https://dart.dev/language>
- API 文档：<https://api.flutter.dev>
- 包仓库：<https://pub.dev>（国内镜像 `pub.flutter-io.cn`）
- 渲染原理：Flutter 官方 “Inside Flutter” 与 “Flutter architectural overview”
- 性能：<https://docs.flutter.dev/perf>
- Impeller：<https://docs.flutter.dev/perf/impeller>
- 发布说明与 Breaking changes：<https://docs.flutter.dev/release>
- Flutter OH（鸿蒙）：<https://gitcode.com/openharmony-sig/flutter_flutter>
- 鸿蒙适配三方库：<https://gitcode.com/openharmony-sig/flutter_packages>
