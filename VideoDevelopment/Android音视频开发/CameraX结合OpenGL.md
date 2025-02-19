CameraX结合OpenGL
===


CameraX 是一个 Jetpack 库，旨在帮助您更轻松地开发相机应用。如果您要开发新应用，我们建议您从 CameraX 开始。它提供了一个一致且易于使用的 API，该 API 适用于绝大多数 Android 设备，并向后兼容 Android 5.0（API 级别 21）。


CameraX 着重于用例，使您可以专注于需要完成的任务，而无需花时间处理不同设备之间的细微差别。CameraX 支持大多数常见的相机用例：

预览：在屏幕上查看图片。接受用于显示预览的 Surface，例如 PreviewView。
图片分析：无缝访问缓冲区中的图片以便在算法中使用，例如将其传递到机器学习套件。为分析（例如机器学习）提供 CPU 可访问的缓冲区。
图片拍摄：保存图片。拍摄并保存照片。
视频拍摄：保存视频和音频。通过 VideoCapture 拍摄视频和音频



版本 1.4.0
2024 年 10 月 30 日

发布了 androidx.camera:camera-*:1.4.0。版本 1.4.0 包含这些提交内容。

自 1.3.0 以来的重要变更

CameraX 1.4.0 包含众多精彩更新！下面是摘要：

主打功能：10 位 HDR：

轻松拍摄出令人惊艳的 HDR 照片和视频。
支持 HLG 和 10 位 HEVC 编码。
享受 10 位 HDR 预览，并查询设备功能。
可在越来越多的设备上与 UltraHDR 图片和 HDR 视频搭配使用。
其他酷炫功能：

Kotlin 扩展：添加了 takePicture 和 awaitInstance 挂起函数。
实时特效：应用水印和对象突出显示等特效。
CameraController API：新增了视频拍摄配置控件。
预览防抖：查询设备功能并启用防抖。
增强了 VideoCapture 功能：可更精细地控制画质，并支持更高分辨率。
CameraX 扩展程序集成：与 VideoCapture 和新的 ImageCapture 功能无缝集成。
Shutter Sound API：轻松查看各个地区的快门提示音要求。
屏幕闪光灯：改进了前置摄像头在弱光环境下的拍照效果。
Camera Extensions Metadata API：支持在 ExtensionMode#AUTO 中调整扩展程序强度和获取当前扩展程序模式通知的 API。如需了解更多 bug 修复，请参阅我们的Beta 版和RC 版公告。




同时选择多个摄像头
从 CameraX 1.3 开始，您还可以同时选择多个摄像头。 例如，您可以对前置和后置摄像头进行绑定，以便从两个视角同时拍摄照片或录制视频。

使用并发摄像头功能时，设备可以同时运行两个不同镜头方向的摄像头，或同时运行两个后置摄像头。以下代码块展示了如何在调用 bindToLifecycle 时设置两个摄像头，以及如何从返回的 ConcurrentCamera 对象中获取两个 Camera 对象。

Kotlin
Java

// Build ConcurrentCameraConfig
val primary = ConcurrentCamera.SingleCameraConfig(
    primaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val secondary = ConcurrentCamera.SingleCameraConfig(
    secondaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val concurrentCamera = cameraProvider.bindToLifecycle(
    listOf(primary, secondary)
)

val primaryCamera = concurrentCamera.cameras[0]
val secondaryCamera = concurrentCamera.cameras[1]






​    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 