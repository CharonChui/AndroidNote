# 08 iOS 平台开发

Android 开发者做 iOS 端时，Dart 业务代码基本不用改，需要补的是：Xcode 工程配置、签名、Swift 插件、iOS 特有的权限与上架规则。

## 8.1 环境

- 只能在 macOS 上构建 iOS 包。需要 Xcode（3.47 已为 Xcode 27 做准备），以及 Apple 开发者账号（真机调试可用免费 Apple ID，发布需要付费账号）。
- 3.47 起 iOS 最低支持版本为 **iOS 15**（`ios/Podfile` 或 Xcode 中的 Deployment Target）。
- 3.44 起默认用 **Swift Package Manager** 管理插件依赖，不再需要 Ruby/CocoaPods。仍依赖 CocoaPods 的老插件会触发自动回退并给出告警；CocoaPods 已进入维护模式。
- 启用 SwiftPM（老工程迁移）：`flutter config --enable-swift-package-manager`，然后 `flutter run` 会自动改造 Xcode 工程。

```bash
open ios/Runner.xcworkspace    # 用 Xcode 打开（有 Pods 时）或 Runner.xcodeproj
flutter run -d <iPhone 设备 id>
flutter build ipa --release --export-options-plist=ExportOptions.plist
```

## 8.2 工程结构对照

| Android | iOS |
|---|---|
| `AndroidManifest.xml` | `Info.plist` |
| `applicationId` | Bundle Identifier |
| `versionName/versionCode` | `CFBundleShortVersionString` / `CFBundleVersion`（Flutter 由 pubspec 自动注入） |
| 签名 keystore | 证书（Certificate）+ 描述文件（Provisioning Profile），在 Xcode → Signing & Capabilities 配置 |
| 权限声明 `<uses-permission>` | `Info.plist` 中的 `NSCameraUsageDescription` 等用途说明（必须写，否则调用时直接崩溃） |
| `MainActivity` | `AppDelegate.swift` + `SceneDelegate`（UIScene） |
| `res/mipmap` 图标 | `Assets.xcassets/AppIcon` |
| Gradle 依赖 | SwiftPM / CocoaPods |
| Build Variant / Flavor | Scheme + Build Configuration |

## 8.3 UIScene 生命周期迁移（必须处理）

Apple 要求使用最新 SDK 构建的应用采用 `UIScene` 生命周期。按 Flutter 3.47 说明，**用 iOS 27 SDK 构建、未迁移到 UIScene 的应用会启动失败**。

- 如果 `AppDelegate` 没有自定义代码，`flutter run/build` 会自动迁移。
- 如果在 `AppDelegate` 中写过 `application(_:didFinishLaunchingWithOptions:)` 里的 window 操作、URL 回调（`open url`）、推送等逻辑，需要按官方 UISceneDelegate 迁移指南手动移动到 SceneDelegate 对应方法中，并检查依赖旧生命周期的插件是否已更新。

## 8.4 Swift 插件 / Channel

以第 7 章的电量 Channel 为例，iOS 实现：

```swift
// 插件形式（推荐，与 Android 插件结构对称）
import Flutter
import UIKit

public class BatteryPlugin: NSObject, FlutterPlugin, FlutterStreamHandler {
  private var eventSink: FlutterEventSink?

  public static func register(with registrar: FlutterPluginRegistrar) {
    let instance = BatteryPlugin()
    let method = FlutterMethodChannel(name: "com.example/battery",
                                      binaryMessenger: registrar.messenger())
    registrar.addMethodCallDelegate(instance, channel: method)
    let event = FlutterEventChannel(name: "com.example/battery_stream",
                                    binaryMessenger: registrar.messenger())
    event.setStreamHandler(instance)
  }

  public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
    switch call.method {
    case "getLevel":
      UIDevice.current.isBatteryMonitoringEnabled = true
      let level = UIDevice.current.batteryLevel
      if level < 0 {
        result(FlutterError(code: "UNAVAILABLE", message: "Battery level not available", details: nil))
      } else {
        result(Int(level * 100))
      }
    default:
      result(FlutterMethodNotImplemented)
    }
  }

  public func onListen(withArguments arguments: Any?, eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    eventSink = events
    UIDevice.current.isBatteryMonitoringEnabled = true
    NotificationCenter.default.addObserver(self, selector: #selector(onChange),
        name: UIDevice.batteryLevelDidChangeNotification, object: nil)
    return nil
  }

  @objc private func onChange() {
    eventSink?(Int(UIDevice.current.batteryLevel * 100))
  }

  public func onCancel(withArguments arguments: Any?) -> FlutterError? {
    NotificationCenter.default.removeObserver(self)
    eventSink = nil
    return nil
  }
}
```

注意：

- iOS 模拟器上 `batteryLevel` 返回 -1，属于正常现象。
- `FlutterResult` 只能回调一次；耗时任务放 `DispatchQueue.global()`，回调可在任意线程（`eventSink` 需要在主线程调用）。
- 使用 Pigeon 时 `swiftOut` 会生成 Swift 协议，实现方式同 Kotlin。

## 8.5 PlatformView（UiKitView）

```swift
class NativeViewFactory: NSObject, FlutterPlatformViewFactory {
  private let messenger: FlutterBinaryMessenger
  init(messenger: FlutterBinaryMessenger) { self.messenger = messenger }

  func create(withFrame frame: CGRect, viewIdentifier viewId: Int64, arguments args: Any?) -> FlutterPlatformView {
    NativeView(frame: frame, args: args as? [String: Any])
  }
  func createArgsCodec() -> FlutterMessageCodec & NSObjectProtocol {
    FlutterStandardMessageCodec.sharedInstance()
  }
}

class NativeView: NSObject, FlutterPlatformView {
  private let label = UILabel()
  init(frame: CGRect, args: [String: Any]?) {
    label.text = args?["text"] as? String
    super.init()
  }
  func view() -> UIView { label }
}

// 注册
registrar.register(NativeViewFactory(messenger: registrar.messenger()), withId: "com.example/native_view")
```

外接纹理：实现 `FlutterTexture` 协议的 `copyPixelBuffer()` 返回 `CVPixelBuffer`，用 `registrar.textures().register(...)` 拿到 textureId，新帧到来时调用 `textureFrameAvailable(id)`。

## 8.6 Add-to-App（iOS）

```bash
flutter create -t module flutter_module
flutter build ios-framework --output=build/ios   # 产出 xcframework，原生工程直接引用
flutter build swift-package                       # 3.44+：打包为 Swift Package
```

```swift
// AppDelegate 中预热引擎
let flutterEngine = FlutterEngine(name: "main")
flutterEngine.run()
GeneratedPluginRegistrant.register(with: flutterEngine)

// 打开 Flutter 页面
let vc = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
navigationController?.pushViewController(vc, animated: true)
```

多引擎同样使用 `FlutterEngineGroup`。

## 8.7 iOS 特有的坑

| 问题 | 解决 |
|---|---|
| 调用相机/相册/定位直接闪退 | `Info.plist` 缺少对应的 `NS...UsageDescription` |
| 真机 debug 包从桌面图标冷启动打不开 | iOS 14+ 限制 debug 模式（JIT）脱离调试器运行，属正常现象；用 profile/release 包测试 |
| 侧滑返回失效 | Flutter 内部路由用 `CupertinoPageRoute`（或 `MaterialPageRoute` 在 iOS 上会自动使用 Cupertino 转场）；`PopScope(canPop: false)` 会禁用侧滑 |
| 状态栏颜色 | 用 `AnnotatedRegion<SystemUiOverlayStyle>` 或 AppBar 的 `systemOverlayStyle`；`Info.plist` 设置 `UIViewControllerBasedStatusBarAppearance` |
| 120Hz ProMotion | 需在 `Info.plist` 添加 `CADisableMinimumFrameDurationOnPhone = true` 才能跑满 120Hz |
| 审核被拒：动态化 | 不要下发可执行代码；Shorebird 等方案需关注审核条款 |
| 隐私清单 | 自 2024 年起，使用特定 API（UserDefaults、文件时间戳、磁盘空间等）需提供 `PrivacyInfo.xcprivacy`；检查三方插件是否自带 |
| 构建报错 `Module not found` | `flutter clean`，删除 `ios/Pods`、`Podfile.lock`（CocoaPods 时）或 Xcode DerivedData 后重新构建 |
| Intel Mac 开发机 | Flutter 正在停止对 Intel Mac 作为开发主机的支持，尽早切换 Apple Silicon |

## 8.8 iOS 端性能特点

- iOS 只使用 Impeller（Metal），着色器卡顿问题在 iOS 上已基本消失。
- 内存：iOS 对单 App 内存限制更严，超出会被系统直接杀（Jetsam），无 OOM 栈。大图列表务必限制解码尺寸（第 11 章）。
- PlatformView 在 iOS 上性能较好，但同屏多个时会产生多个 overlay，需要控制数量。
- 后台：iOS 对后台执行限制严格，切后台后 Dart 代码很快被挂起，下载、定位等后台任务需要原生侧（`BGTaskScheduler`、`URLSession` 后台会话）实现。

## 动手练习

1. 在 iOS 模拟器上运行第 3～5 章的练习工程，观察 Material 组件在 iOS 上的自动适配（滚动回弹、转场动画）。
2. 给第 7 章的 `native_toast` 插件补充 iOS 实现（可用 `UIAlertController` 模拟 toast）。
3. 用 `Platform.isIOS` / `Theme.of(context).platform` 实现一个按平台显示 Cupertino 或 Material 对话框的工具函数。
