# 07 Android 平台开发与混合集成

本章是 Android 开发者最能发挥既有经验的部分：写原生插件、嵌原生 View、把 Flutter 接入已有的大型 Android 工程（例如本仓库这种多模块剪辑 App）。

## 7.1 Android 宿主工程

`flutter create` 生成的 `android/` 是一个标准 Gradle 工程（Kotlin DSL）：

```kotlin
// android/settings.gradle.kts（简化）
pluginManagement {
    val flutterSdkPath = /* 从 local.properties 读取 flutter.sdk */
    includeBuild("$flutterSdkPath/packages/flutter_tools/gradle")
}
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "..." apply false
}

// android/app/build.gradle.kts
plugins {
    id("com.android.application")
    id("dev.flutter.flutter-gradle-plugin")   // 负责编译 Dart、打包引擎和 assets
}
android {
    namespace = "com.example.app"
    compileSdk = flutter.compileSdkVersion     // 3.47 默认 36
    defaultConfig {
        minSdk = flutter.minSdkVersion         // 3.47 默认 24，可改成自己的值
        targetSdk = flutter.targetSdkVersion
        versionCode = flutter.versionCode      // 来自 pubspec 的 version: x.y.z+code
        versionName = flutter.versionName
    }
}
flutter { source = "../.." }
```

工具链版本（3.47 官方验证）：Java 17+、AGP 9.1、Gradle 9.3、KGP 2.4。AGP 9 内置了 Kotlin 支持，**不要再手动 apply `kotlin-android` 插件**（3.44 起会冲突）。

`MainActivity`：

```kotlin
class MainActivity : FlutterActivity()   // 或 FlutterFragmentActivity（需要 Fragment 能力的插件，如 local_auth）
```

## 7.2 MethodChannel：调用原生能力

以“获取电量 + 监听电量变化”为例。

Dart 侧：

```dart
class BatteryApi {
  static const _method = MethodChannel('com.example/battery');
  static const _event = EventChannel('com.example/battery_stream');

  Future<int> getLevel() async {
    try {
      final level = await _method.invokeMethod<int>('getLevel');
      return level ?? -1;
    } on PlatformException catch (e) {
      debugPrint('failed: ${e.code} ${e.message}');
      return -1;
    } on MissingPluginException {
      return -1;   // 该平台未实现（例如鸿蒙尚未适配）
    }
  }

  Stream<int> get levelStream => _event.receiveBroadcastStream().map((e) => e as int);
}
```

Android 侧（直接写在 MainActivity 里，适合 App 私有能力）：

```kotlin
class MainActivity : FlutterActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)   // 内部会注册 pubspec 中的插件
        val messenger = flutterEngine.dartExecutor.binaryMessenger

        MethodChannel(messenger, "com.example/battery").setMethodCallHandler { call, result ->
            when (call.method) {
                "getLevel" -> {
                    val bm = getSystemService(BATTERY_SERVICE) as BatteryManager
                    val level = bm.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
                    if (level >= 0) result.success(level)
                    else result.error("UNAVAILABLE", "Battery level not available.", null)
                }
                else -> result.notImplemented()
            }
        }

        EventChannel(messenger, "com.example/battery_stream").setStreamHandler(
            object : EventChannel.StreamHandler {
                private var receiver: BroadcastReceiver? = null
                override fun onListen(arguments: Any?, events: EventChannel.EventSink) {
                    receiver = object : BroadcastReceiver() {
                        override fun onReceive(c: Context, i: Intent) {
                            events.success(i.getIntExtra(BatteryManager.EXTRA_LEVEL, -1))
                        }
                    }
                    registerReceiver(receiver, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
                }
                override fun onCancel(arguments: Any?) {
                    receiver?.let { unregisterReceiver(it) }
                    receiver = null
                }
            })
    }
}
```

规则：

- 每个 `result` **必须且只能回复一次**（success / error / notImplemented 三选一），否则 Dart 端的 Future 永远不完成，或抛 “Reply already submitted”。
- `onMethodCall` 在主线程执行。耗时操作切到后台线程（协程 `Dispatchers.IO`），完成后调用 `result.success`（引擎会把回复投递回 Dart，所以可以在后台线程调用；但 `EventSink.success` 必须在主线程调用）。
- Channel 名称两端必须完全一致，建议用反向域名前缀避免冲突。

## 7.3 Pigeon：类型安全的 Channel（推荐）

手写 MethodChannel 容易出现方法名拼错、参数类型对不上。Pigeon 用 Dart 定义接口，自动生成 Dart / Kotlin / Swift（以及 C++/ObjC）代码。

```dart
// pigeons/video_api.dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(PigeonOptions(
  dartOut: 'lib/src/video_api.g.dart',
  kotlinOut: 'android/app/src/main/kotlin/com/example/VideoApi.g.kt',
  kotlinOptions: KotlinOptions(package: 'com.example'),
  swiftOut: 'ios/Runner/VideoApi.g.swift',
))

class VideoInfo {
  VideoInfo({required this.path, required this.durationMs, this.width, this.height});
  String path;
  int durationMs;
  int? width;
  int? height;
}

@HostApi()            // Dart 调原生
abstract class VideoHostApi {
  @async
  VideoInfo probe(String path);
  @async
  String export(String draftId, int quality);
}

@FlutterApi()         // 原生调 Dart
abstract class VideoFlutterApi {
  void onExportProgress(String draftId, double progress);
}
```

```bash
dart run pigeon --input pigeons/video_api.dart
```

Kotlin 侧实现生成的接口：

```kotlin
class VideoHostApiImpl(private val context: Context) : VideoHostApi {
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

    override fun probe(path: String, callback: (Result<VideoInfo>) -> Unit) {
        scope.launch {
            val info = runCatching {
                withContext(Dispatchers.IO) {
                    val r = MediaMetadataRetriever()
                    try {
                        r.setDataSource(path)
                        VideoInfo(
                            path = path,
                            durationMs = r.extractMetadata(METADATA_KEY_DURATION)?.toLong() ?: 0,
                            width = r.extractMetadata(METADATA_KEY_VIDEO_WIDTH)?.toLong(),
                            height = r.extractMetadata(METADATA_KEY_VIDEO_HEIGHT)?.toLong(),
                        )
                    } finally {
                        r.release()   // AutoCloseable 需要 API 29，minSdk 24 时手动 release
                    }
                }
            }
            callback(info)   // 失败时 Dart 侧收到 PlatformException
        }
    }

    override fun export(draftId: String, quality: Long, callback: (Result<String>) -> Unit) { /* ... */ }
}

// 注册
VideoHostApi.setUp(flutterEngine.dartExecutor.binaryMessenger, VideoHostApiImpl(this))
```

注意 Dart 的 `int` 在 Kotlin 侧是 `Long`。

## 7.4 编写独立插件（可复用、可发布）

```bash
flutter create --template=plugin --platforms=android,ios --org com.example native_toast
```

```
native_toast/
├── lib/native_toast.dart                     # Dart API
├── android/src/main/kotlin/.../NativeToastPlugin.kt
├── ios/Classes/NativeToastPlugin.swift       # 3.44 起默认带 Package.swift（SwiftPM）
├── example/                                   # 示例 App，用来调试
└── pubspec.yaml
```

```yaml
# pubspec.yaml 中声明各平台实现类
flutter:
  plugin:
    platforms:
      android:
        package: com.example.native_toast
        pluginClass: NativeToastPlugin
      ios:
        pluginClass: NativeToastPlugin
      ohos:                                   # 鸿蒙实现（Flutter OH，见第 9 章）
        pluginClass: NativeToastPlugin
```

Android 插件类：

```kotlin
class NativeToastPlugin : FlutterPlugin, MethodChannel.MethodCallHandler, ActivityAware {
    private lateinit var channel: MethodChannel
    private lateinit var appContext: Context
    private var activity: Activity? = null

    override fun onAttachedToEngine(binding: FlutterPlugin.FlutterPluginBinding) {
        appContext = binding.applicationContext
        channel = MethodChannel(binding.binaryMessenger, "native_toast")
        channel.setMethodCallHandler(this)
    }

    override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
        when (call.method) {
            "show" -> {
                val msg = call.argument<String>("msg").orEmpty()
                Toast.makeText(appContext, msg, Toast.LENGTH_SHORT).show()
                result.success(null)
            }
            else -> result.notImplemented()
        }
    }

    override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {
        channel.setMethodCallHandler(null)
    }

    // 需要 Activity（弹权限、startActivityForResult）时实现 ActivityAware
    override fun onAttachedToActivity(binding: ActivityPluginBinding) {
        activity = binding.activity
        // binding.addActivityResultListener { requestCode, resultCode, data -> ... }
        // binding.addRequestPermissionsResultListener { ... }
    }
    override fun onDetachedFromActivityForConfigChanges() { activity = null }
    override fun onReattachedToActivityForConfigChanges(binding: ActivityPluginBinding) { activity = binding.activity }
    override fun onDetachedFromActivity() { activity = null }
}
```

要点：

- `onAttachedToEngine` 时可能还没有 Activity（例如后台引擎），需要 Activity 的逻辑放到 `ActivityAware` 回调里。
- 旧版 `PluginRegistry.Registrar` / `registerWith` 已在 3.29 移除，网上的老代码不要照抄。
- 插件内依赖的 Android 库写在插件的 `android/build.gradle` 里，与宿主依赖冲突时由宿主 Gradle 统一解析。
- 联邦插件（federated plugin）：大型插件把 `xxx_platform_interface`、`xxx_android`、`xxx_ios`、`xxx_ohos` 拆成多个包，鸿蒙适配通常以这种方式追加（见第 9 章）。

## 7.5 PlatformView：在 Flutter 中嵌原生 View

Dart 侧：

```dart
Widget build(BuildContext context) {
  const viewType = 'com.example/native_player';
  final params = <String, dynamic>{'url': url};
  if (defaultTargetPlatform == TargetPlatform.android) {
    return AndroidView(            // 默认 TLHC，必要时回退 HC
      viewType: viewType,
      creationParams: params,
      creationParamsCodec: const StandardMessageCodec(),
      onPlatformViewCreated: (id) { /* 用 id 建立该实例专属的 MethodChannel */ },
    );
  }
  return UiKitView(viewType: viewType, creationParams: params,
      creationParamsCodec: const StandardMessageCodec());
}
```

如果要强制使用 Hybrid Composition（例如 `SurfaceView` 内容显示异常），使用 `PlatformViewLink` + `PlatformViewsService.initExpensiveAndroidView`。

Android 侧：

```kotlin
class NativePlayerFactory(private val messenger: BinaryMessenger) :
    PlatformViewFactory(StandardMessageCodec.INSTANCE) {
    override fun create(context: Context, viewId: Int, args: Any?): PlatformView {
        val params = args as Map<*, *>
        return NativePlayerView(context, viewId, params["url"] as String, messenger)
    }
}

class NativePlayerView(context: Context, id: Int, url: String, messenger: BinaryMessenger) : PlatformView {
    private val player = MyPlayerView(context).apply { setDataSource(url) }
    override fun getView(): View = player
    override fun dispose() { player.release() }   // Flutter 侧 Widget 移除时调用，务必释放
}

// 注册（在插件 onAttachedToEngine 或 configureFlutterEngine 中）
flutterEngine.platformViewsController.registry
    .registerViewFactory("com.example/native_player", NativePlayerFactory(messenger))
```

开启 HCPP（3.44+，可选）：

```xml
<meta-data android:name="io.flutter.embedding.android.EnableHcpp" android:value="true" />
```

PlatformView 的选择建议：

- 只需要显示画面（视频、相机、原生渲染引擎预览）→ 用**外接纹理 `Texture`**，性能最好。
- 需要原生控件的交互（地图、WebView、广告）→ PlatformView。
- 同屏 PlatformView 数量尽量少，不要放进长列表里每个 item 一个。

### 外接纹理示例（原生渲染 → Flutter 显示）

```kotlin
// 插件中
val entry = binding.textureRegistry.createSurfaceProducer()   // 新 API，兼容 Impeller
val surface: Surface = entry.surface
entry.setSize(width, height)
nativeEngine.setOutputSurface(surface)    // 原生播放器/特效引擎往这个 Surface 渲染
result.success(entry.id())                // 把 textureId 返回 Dart
// 释放：entry.release()
```

```dart
final textureId = await channel.invokeMethod<int>('createPreview');
AspectRatio(aspectRatio: 16 / 9, child: Texture(textureId: textureId!));
```

Impeller 下推荐使用 `createSurfaceProducer()`，旧的 `createSurfaceTexture()` 在 Vulkan 后端下性能和兼容性较差。`SurfaceProducer` 在应用切后台时 Surface 可能被销毁，需要实现 `SurfaceProducer.Callback` 在 `onSurfaceAvailable` 时重新绑定。

## 7.6 Add-to-App：把 Flutter 集成到已有 Android 工程

大多数公司是“已有原生 App，部分页面用 Flutter 写”。两种集成方式：

| 方式 | 做法 | 优点 | 缺点 |
|---|---|---|---|
| 源码依赖 | `settings.gradle` 中 include Flutter module | 改 Dart 后直接运行，调试方便 | 所有 Android 开发者都要装 Flutter SDK；宿主 Gradle/AGP 版本要满足 Flutter 要求 |
| AAR 依赖（推荐大团队） | `flutter build aar` 产物发布到 Maven | 原生同学无感，不需要 Flutter 环境；与宿主工具链解耦 | Flutter 改动需要重新打 AAR |

### 1. 创建 Flutter module

```bash
cd ..   # 与宿主工程平级
flutter create -t module --org com.baidu.tz flutter_module
```

module 下的 `.android/` 是自动生成的临时宿主，**不要手动修改，不要提交**。

### 2a. 源码依赖

```groovy
// 宿主 settings.gradle
apply from: "../flutter_module/.android/include_flutter.groovy"
// 或新版写法
// include(":flutter"); project(":flutter").projectDir = file("../flutter_module/.android/Flutter")

// app/build.gradle
dependencies { implementation project(':flutter') }
```

### 2b. AAR 依赖

```bash
cd flutter_module
flutter build aar --build-number=1.0.0   # 产出 debug/profile/release 三套 AAR + 本地 maven 仓库
```

```groovy
repositories {
    maven { url '/path/to/flutter_module/build/host/outputs/repo' }   // 或公司 Maven
    maven { url "https://storage.googleapis.com/download.flutter.io" } // 引擎 AAR（国内用镜像）
}
dependencies {
    debugImplementation 'com.baidu.tz.flutter_module:flutter_debug:1.0.0'
    releaseImplementation 'com.baidu.tz.flutter_module:flutter_release:1.0.0'
}
```

### 3. 宿主中打开 Flutter 页面 + 引擎预热

```kotlin
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        // 预热：提前创建引擎并执行 Dart 入口，打开页面时无白屏（需要约数十 MB 内存）
        val engine = FlutterEngine(this).apply {
            navigationChannel.setInitialRoute("/home")
            dartExecutor.executeDartEntrypoint(DartExecutor.DartEntrypoint.createDefault())
        }
        FlutterEngineCache.getInstance().put("main", engine)
    }
}

// 打开页面
startActivity(
    FlutterActivity.withCachedEngine("main")
        .backgroundMode(FlutterActivityLaunchConfigs.BackgroundMode.opaque)
        .build(this)
)

// 作为 Fragment 嵌入原生页面（例如一个 Tab）
val fragment = FlutterFragment.withCachedEngine("main").build<FlutterFragment>()
supportFragmentManager.beginTransaction().add(R.id.container, fragment).commit()
```

`AndroidManifest.xml` 声明 `FlutterActivity`：

```xml
<activity
    android:name="io.flutter.embedding.android.FlutterActivity"
    android:theme="@style/AppTheme"
    android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
    android:hardwareAccelerated="true"
    android:windowSoftInputMode="adjustResize" />
```

`configChanges` 必须包含这些项，否则旋转屏幕等配置变化会导致 Activity 重建，Flutter 状态丢失。

### 4. 多个 Flutter 页面与原生页面交替（混合栈）

问题：原生 A → Flutter B → 原生 C → Flutter D。如果共用一个缓存引擎，B 和 D 抢同一个 FlutterView；如果每个页面新建引擎，内存暴涨、状态不共享。

方案：

- **FlutterEngineGroup**（官方）：从一个引擎组派生多个轻量引擎，共享代码和 GPU 上下文，每个额外引擎约 180KB。缺点是各引擎的 Dart 内存（Isolate）不共享，状态需要通过原生层或持久化同步。

```kotlin
val group = FlutterEngineGroup(context)
val engine = group.createAndRunEngine(
    FlutterEngineGroup.Options(context)
        .setDartEntrypoint(DartExecutor.DartEntrypoint(FlutterInjector.instance().flutterLoader().findAppBundlePath(), "main"))
        .setInitialRoute("/detail?id=1")
)
```

- 单引擎 + 混合栈框架（如阿里 `flutter_boost`）：所有 Flutter 页面共享一个引擎，切换时把 FlutterView 重新 attach 到当前 Activity。社区方案，升级 Flutter 版本时要关注其兼容性。
- 尽量减少混合跳转：让一个完整业务流程都在 Flutter 内完成，原生只负责入口。

### 5. 宿主工程的兼容性检查

接入前对照宿主工程检查：

| 项 | 本仓库现状 | Flutter 3.47 要求 | 处理 |
|---|---|---|---|
| AGP | 7.4.2 | 官方验证 AGP 9.1 | 源码依赖需升级宿主 AGP；或改用 AAR 方式 |
| Gradle | 7.5 | 官方验证 9.3 | 同上 |
| Kotlin | 1.9.21 | 官方验证 KGP 2.4 | AAR 中插件编译出的 Kotlin 元数据版本可能高于宿主编译器能读取的版本，需验证 |
| Java | 17 | 17+ | 满足 |
| minSdk | 24 | 默认 24 | 满足 |
| ABI | — | 不支持 32 位 x86 | 检查 `abiFilters` |

老宿主工程不方便升级工具链时，可以选择一个与宿主工具链匹配的较老 Flutter 版本（用 FVM 锁定），并采用 AAR 方式隔离。具体每个 Flutter 版本支持的最低 AGP/Gradle，以 `flutter build` 时打印的告警和官方 release notes 为准。

## 7.7 Android 侧常见问题

| 问题 | 原因与解决 |
|---|---|
| 启动白屏/黑屏 | 配置 `LaunchTheme` 的 `windowBackground`；Android 12+ 使用 SplashScreen API；Add-to-App 使用引擎预热 |
| `MissingPluginException` | 插件未注册：新增插件后需要完全重新运行（热重载无效）；Add-to-App 自建引擎时需调用 `GeneratedPluginRegistrant.registerWith(engine)`；后台 Isolate 未初始化 messenger |
| release 包闪退，debug 正常 | R8 混淆了插件中反射用到的类，补充 proguard 规则；或 `abiFilters` 去掉了 arm64 |
| 软键盘遮挡输入框 | `windowSoftInputMode="adjustResize"`，Flutter 侧 `Scaffold(resizeToAvoidBottomInset: true)` |
| Android 15 edge-to-edge 强制全面屏 | targetSdk 35+ 默认 edge-to-edge，Flutter 内容会延伸到状态栏和导航栏下方，用 `SafeArea` / `MediaQuery.paddingOf` 处理 |
| 多 Activity/进程 | Flutter 引擎绑定进程，多进程中每个进程需要各自初始化；尽量在主进程使用 |
| 插件 Gradle 冲突 | 宿主 `resolutionStrategy` 统一版本；AGP 9 下插件仍 apply KGP 导致构建失败，需升级插件 |
| 包体积增加 | 见第 11 章：`--split-per-abi`、`--obfuscate --split-debug-info`、AAB |

## 动手练习

1. 用 Pigeon 封装 `MediaMetadataRetriever`，在 Flutter 中展示视频时长和分辨率。
2. 写一个插件：原生用 `MediaPlayer`/ExoPlayer 往 `SurfaceProducer` 渲染，Flutter 用 `Texture` 显示，实现播放/暂停。
3. 新建一个原生 Android 工程，用 AAR 方式接入 Flutter module，实现原生页面 → Flutter 页面传参，Flutter 页面返回结果给原生。
