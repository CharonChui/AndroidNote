PAG
---

PAG(Portable Animated Graphics)是腾讯开源的一套完整动效工作流解决方案，主要包含三个部分:    

1. AE导出插件PAGExporter: 把After Effects动效导出为.pag二进制文件。 
2. 桌面预览/性能监测工具PAGViewer
3. 夸端渲染SDK libpag: iOS/Android/macOS/Windows/Web/鸿蒙

广泛应用于视频模板、UI动画、贴纸等场景。     

与Lottie最大的区别是:     

- Lottie是纯矢量Json，只支持AE的部分特性。   
- PAG是二进制形式+矢量/BMP预合成混合导出，能支持AE的全量特性


### PAG的四大核心优势


| 维度 | PAG  | Lottie |
|-------|-------|-------|
| 文件格式  | 二进制 TLV(Tag-Length-Value)  | Json文本 |
| 解码速度 | 比Json快约10倍 | 慢 |
| 文件体积 | 小约50%  | 大 |
| AE特性覆盖 | 全量(矢量+BMP预合成混合) | 仅支持部分矢量 |
| 资源集成 | 单文件可内嵌图片/音频 | 需外挂 |
| 运行时编辑 | 文本、占位图、图层均可替换 | 能力有限 |
| 性能监测 | PAGViewer内置 | 无 |


### 实现原理


- 最上层: PAGView、PAGImageView(UI层)
- Java层: PAGFile、PAGPlayer、PAGSurface，该层再通过JNI进行调用
- C++层(核心): 夸端共享     
    - Codec: 解码
    - Renderer: 合成、渲染
    - RendererCache: 纹理、缓存管理
- TGFX图形库(Tencent Graphics): 封装GPU后端(OpenGL、Metal、Vulkan)
- 平台硬件解码: MediaCodec、VideoToolbox


### PAG文件格式

- 采用可扩展二进制TLV编码(Tag + Length + Value)
- 每个Tag代表一种属性块(图形、关键帧、遮罩、文本、图像等)，tagLevel指明所需最低SDK版本，天然支持向前/向后兼容: 老版本SDK遇到不认识的Tag会跳过而不崩溃。   
- 文件里可以同时承载:     
    - 矢量数据: 形状、文本、遮罩、表达式驱动的变换等(纯矢量导出模式)
    - BMP预合成序列帧: 通过H.264视频轨压缩存储(BMP混合模式)，专门用于那些无法矢量化的复杂AE特效(粒子、第三方插件等)     
- 解码时按图层时间轴拉取需要的帧数据，不必一次性全量解析，首诊渲染很快。   

### 导出模式     

- 矢量导出(Vector Export): 逐层导出AE的shape/text/solid/mask/transform等，运行时保留图层结构，可编辑。   

- BMP预合成导出(Video Sequence): 把一段无法矢量化的合成，渲染成帧序列并用H.264编码，运行时调用MediaCodec硬解。   

- 混合模式: 两者在同一.pag文件里组合--可编辑的文字/占位图保留矢量，特效部分走视频序列。这是PAG能全量支持AE特性+仍能保持运行时可编辑的关键。   



### 渲染流水线

- 解码: Codec把TLV还原为内存中的PAGFile图形树(Composition -> Layer -> Property)
- 组合: PAGPlayer按当前progress计算每一层的变换、关键帧插值，构建一颗渲染树。 
- 渲染(TGFX): TGFX是PAG自研的轻量图形库，把渲染树翻译成GPU指令，通过OpenGL/Metal/Vulkan后端提交渲染
- 输出: 渲染到PAGSurface，它的底层可以是SurfaceTexture/Surface/HardwareBuffer/离屏FBO
- 缓存(RendererCache):    
    - 纹理缓存: 同一帧内相同图层结果复用
    - 视频序列缓存: MediaCodec解码出的帧写入PAGDisckCache
    - 字形缓存: 文本按glyph光栅化后缓存

### PAG诞生背景

PAG方案最早就是诞生在视频编辑场景下，要让动画能够在视频编辑场景下无缝整合使用，需要解决两个问题:   

- 支持离屏渲染绘制、子线程渲染。 Lottie的动画方案之所以无法应用在视频合成中，主要是因为依赖了平台相关的UI框架，开发成本较低，但是也导致了它只能渲染到UI视图上，并且无法在子线程中使用。   

- PAG的整套动画方案就是基于C++跨平台架构研发的，一直从最底层的动画插值器，还原到上层的时间轴和图层渲染树系统，虽然开发成本较高，但是所有端共享同一套代码，天然的能保证夸端渲染一致性。最重要的是能直接渲染到离屏纹理上，并完美支持子线程动画渲染。  

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/pag_1.jpeg?raw=true)   


### 为什么性能好

- 二进制TLV解析无字符串解析开销
- GPU合成+TGFX优化的批渲染
- 硬件视频解码处理复杂特效，绕开CPU矢量光栅化瓶颈
- RenderCache + PAGDiskCache减少重复计算与磁盘I/O
- DisplayLink(Android用Choreographer)严格对齐VSYNC,避免丢帧     


### 核心类

- PAGFile: .pag文件的解析产物，PAGComposition子类，可查询/修改文本、图片、图层时间拉伸等。    

- PAGComposition: 合成节点(可嵌套),管理子图层，支持动态增删子图层，合并多个PAG。   
- PAGLayer: 图层基类，派生PAGTextLayer、PAGImageLayer、PAGShapeLayer、PAGSolidLayer。 

- PAGImage: 位图资源封装，用来替换占位图。支持Bitmap、文件路径、纹理ID。

- PAGText: 文本属性，如text/fontSize/fontFamily/fillColor/strokeColor等     

- PAGFont: 注册自定义字体(TTF/OTF)，指定fontFamily与fontStyle    

- PAGPlayer: 渲染驱动核心，绑定PAGComposition和PAGSurface，控制progress/scaleMode/cacheEnabled。自定义渲染时直接用它。   

- PAGSurface: 渲染目标，基于GLSurface/SurfaceTexture

- PAGView: 封装好的TextureView播放控件，内部驱动PAGPlayer，对齐Chorerographer。

- PAGImageView: 基于位图序列的ImageView，省电，适合列表里大量循环小动效。   

- PAGAnimator: 4.0引入的驱动器，自定义渲染时用来与VSYNC对齐。   

- PAGDecoder: 帧解码器，把PAG逐帧解成Bitmap。  

- PAGDiskCache: 磁盘缓存管理，可清缓存、改大小。   



### 运行时替换文字


- 替换文字的前提是AE侧要把文件标记为可编辑

    用PAGExporter导出时，在AE图层面板要把该文本层的Editable属性勾选，这样导出后SDK里才会为这个文本分配一个editableIndex。   

- 通过index替换文字内容

```java
PAGText text = pageFile.getTextData(0); // 取出第0个可编辑文本的当前属性
text.text = "hhhh";
text.fontSize = 40;
text.fillColor = Color.parseColor("#ff3333");
pageFile.replaceText(0, text);  // 回写
```

- 通过涂层面查找替换(防止索引变动)

editableIndex会随导出插件排序变化，业务上更好的方式是使用AE里自定义的图层名:   

```java
PAGLayer[] layers = pagFile.getLayersByEditableIndex(0, PAGLayer.LayerTypeText);
for (PAGLayer layer : layers) {
  if (layer instanceof PAGTextLayer) {
        PAGTextLayer t = (PAGTextLayer) layer;
        if ("title_layer".equals(t.layerName())) {
            t.setText("Ducc");
            t.setFillColor(Color.RED);
            t.setFontSize(48);
        }
    }
}
```

替换后无需重新Load文件，调一次pagView.flush()或者等下一帧就会刷新。   

- 替换图片   

```java
PAGImage pageImage = PAGImage.FromBitmap(bmp); 

// 按index替换
pageFile.replaceImage(0, pageImage);
// 按图层名替换
pageFile.replaceImageByName("name", pageImage);

```

- 换字体

换字体需要先使用PAGFont.RegisterFont()方法来把本地的.ttf注册。然后再通过PAGText.fontFamily方法来使用。   


- 视频模板

视频模板就是在一个PAG文件里堆放N个可替换占位图 + 可替换文字，业务层把用户上传的素材塞进即可。   

这就是PAG在一些视频编辑软件中被大量使用的场景。    



- 合成与图层操作   

```java
PAGComposition root = PAGComposition.Make(720, 1280);
root.addLayer(pageFile1);
root.addLayer(pageFile2);
pagView.setComposition(root);

// 查询/删除图层

int count = root.numChildren();
PAGLayer child = root.getLayerAt(0);
root.removeLayerAt(0);
```



### 问题点:   

- PAGImageView与PAGView

PAGImageView是基于位图序列的ImageView，省电，适合列表里大量循环小动效。     
而PAGView中有装好的TextureView播放控件，内部驱动PAGPlayer。   
所以对于列表中有大量小动效(点赞、表情)的情况，优先使用PAGImageView + 磁盘缓存，它把动画预解码为位图序列，几乎零CPU。    

- cacheEnabled/cacheScale: 复杂动画开cacheScale < 1用降采样换内存。纯矢量不大时可以关缓存。 

- 硬件解码资源: 含视频序列的PAG会占用MediaCodec实例，同屏有多个PAG动画时需要注意codec上限(一般10 - 16个)，超过会IllegalStateException。   

- 字体内存: PAGFont.registerFont注册时进程级的，频繁注册大字体会OOM，业务上注册一次即可。   

- 不要把PAGView放进recyclerview无节制复用，因为PAGView内部有一个TextureView，而TextureView在滑动列表中有SurfaceTexture的重建成本，建议使用PAGImageView。  

- 内存释放: PAGImage.fromBitmap会引用bitmap，替换完成后如果不再使用，需要把PAGFile引用断掉。   






------

## 矢量 vs BMP

矢量图和位图是图形中最基础的两种图像表达方式。     

PAG之所以这么强，就在于它把两者在同一个文件里混着用。    

#### 两者的本质区别

###### 矢量(Vector)

矢量就是用数学描述画图。    

矢量图不存像素，存的是几何指令和关键帧。     

例如一个红色圆:    
`circle(center=(100,100), r=50, fillColor=#FF0000, strokeWidth=2)`
渲染时，CPU/GPU会按这条指令实时计算出每个像素该是什么颜色。   


它的特点是:    

- 存储的内容是路径(贝塞尔曲线)、描边、填充、变换矩阵、遮罩等。 
- 文件体积非常小
- 可无限缩放，即使放大10被也不糊，重新按数学公式算像素
- 可运行时编辑，改颜色/大小/文字/位置都只是改参数
- 缺点是复杂的特效会算不动，例如一些例子、发光、烟雾等，对于一些第三方AE插件效果，如果用到矢量去模拟，要么会做不到，要么渲染一帧要几十毫秒。   


###### 位图(Bitmap)

位图就是把每个像素的颜色值都直接保存下来。    

比如一个100*100的图，就是10000个RGBA值的数组。   

它的特点是:   

- 存储内容是纯像素颜色矩阵
- 文件体积比较大，因为每个像素都要存，但是可以用PNG、JPEG、H.264压缩
- 缩放会糊，放大就是像素插值，会模糊，缩小会丢信息
- 运行时不可编辑，想改颜色只能逐像素重画
- 有点是什么画面都能表达，不管是烟雾、粒子、3D渲染结果都可以


#### 类比

一个形象的比喻就是:    

- 矢量 = 乐谱。 占的空间小，乐手按照乐谱现场演奏，音质由演奏者决定(放大不糊)
- 位图 = 录音文件。 每一秒的声波都被采样保存下来了，文件很大。

PAG的创新就是: 一首曲子里，能弹的部分用乐谱，演奏不出来的插件音效用录音片段。   


PAG把AE里那些无法矢量化的部分(粒子、插件效果、复杂混合模式)，让AE先渲染成一段帧序列(每一帧都是一张位图)，然后用H.264视频编码压缩到.pag文件里。   

- PAGExporter检测到无法矢量化
- 调AE去渲染成PNG序列： frame_0.png/frame_1.png ...
- 调H.264编码成一段视频
- 作为BMP预合成图层写进.pag
- 运行时，libpag用MediaCodec(Android)/VideoToolbox(iOS)硬件解码这段视频，逐帧纹理上屏。    


例如有一个摇出金币的动效:    

- Composition(根合成) 
    - 背景渐变: 矢量(ShapeLayer)
    - 文字: 矢量(TextLayer，可编辑)
    - 用户头像占位图: 矢量(ImageLayer，可替换)
    - 粒子金币特效: BMP预合成(一段H.264视频序列)

  
  
  
### Alpha通道问题

H.264不支持透明通道。     

PAG用了一个聪明的方法: 把视频帧水平或垂直拼成两半:    

- 左半边存RGB颜色、右半边存alpha灰度
- 解码后在shader里合并成RGBA

这样就用标准的H.264编码器得到了带透明度的视频序列。    

iOS的HEV支持原生alpha，新版PAG在部分平台上也会用HEVC直接带alpha编码。   

下面详细介绍一下alpha通道问题:  


H.264是2003年ITU-T定下的视频编码标准，设计目标是高效压缩自然场景视频(电影、监控、直播)。   
它的像素模型是YUV 4：2：0，也就是亮度、色度蓝、色度红。  
三个通道，没有Alpha。   

硬件解码芯片(Adnroid MediaCodec、iOS VideoToolbox)也都是围绕YUV流水线设计的，没有第四通道的位置。   

但是PAG的BMP预合成图层是要叠加在其他图层之上的，没有alpha，边缘就是一个黑色或白色的钜形背景，根本没法用。    


那PAG的思路就是把Alpha藏进RGB通道里:    

既然H.264一帧只能存RGB，那就把一帧原本 width * height 的画面，拼成一张更大的图:   

- 一半区域存原始RGB值

- 另一半区域存Alpha值(用R=G=B=alpha)的灰度图来表示。  

编码时当成一个普通的h.264视频。    

解码时得到一张大图，再用GPU Shader把RGB部分和Alpha部分按像素对齐合成RGBA。 

这个技巧，行业内叫"side-by-side alpha packing"或"alpha-as-luma"，不是PAG首创，（Google的WebM Alpha、Adobe的Animate也用过类似思路），但PAG把它工程化做到了跨端、硬件解码、shader合成一体。

#### 具体排布：两种打包方式

PAG支持两种alpha打包方向，由PAGExporter根据视频宽高比自动选择更省空间的那个：

###### 水平拼接（Left-Right）
```
  原画面 W×H               编码后的 H.264 帧 2W×H
  ┌──────────┐            ┌──────────┬──────────┐
  │   RGBA   │    ===>    │   RGB    │  Alpha   │
  │   一帧    │            │ (原颜色) │ (灰度图) │
  └──────────┘            └──────────┴──────────┘
```

###### 垂直拼接（Top-Bottom）

```
  原画面 W×H               编码后的 H.264 帧 W×2H
  ┌──────────┐            ┌──────────┐
  │   RGBA   │            │   RGB    │
  │   一帧    │    ===>    │ (原颜色) │
  └──────────┘            ├──────────┤
                          │  Alpha   │
                          │ (灰度图) │
                          └──────────┘
```
选择哪种的依据：让打包后的总尺寸对H.264编码器更友好（一般是16的倍数对齐，利于宏块划分），以及让编码压缩率更高。

#### 元信息如何告诉 SDK

.pag文件的TLV里，这段BMP预合成图层的Tag里会带几个关键字段：

``java
VideoCompositionBlock {
    width:          实际原始宽度 W
    height:         实际原始高度 H
    alphaStartX:    Alpha 区域在大图里的起始 x (水平拼接时 = W)
    alphaStartY:    Alpha 区域在大图里的起始 y (垂直拼接时 = H)
    hasAlpha:       bool
    frameRate, duration, ...
}
``
解码时，SDK知道"这段视频真正的画面是W×H，alpha区域从(alphaStartX, alphaStartY) 开始"，后面shader合成就按这两个偏移取样。

#### 编码侧：PAGExporter 怎么做

设计师在AE里导出时，PAGExporter对BMP预合成图层做如下处理:     

1. 逐帧渲染：让AE把这段合成render成一组RGBA位图（每帧W×H，带透明通道）。
2. 拆分通道：对每一帧：
    - 取RGB部分 → 放到大图左半（或上半）；
    - 取Alpha通道 → 复制成灰度图 (A, A, A) → 放到大图右半（或下半）。
3. 拼帧入视频：所有帧按顺序喂给H.264编码器（PAGExporter在macOS/Windows里调的是各自平台的SDK。   
4. 写入TLV：把H.264码流作为VideoCompositionBlock的payload写进.pag，同时记录
  hasAlpha=true、alphaStartX/Y。

  ▎ 小细节：alpha 灰度图压缩率很高（大片平坦区域），H.264
  ▎ 对这种内容压得特别好，因此"2×面积"实际体积增加往往只有 20%~40%，远比 PNG 序列小。

#### 解码侧：运行时怎么合回来

这是最关键的一步，也是PAG性能好的原因。

###### 硬件解码拿到大图
```java
// 伪代码
MediaCodec codec = MediaCodec.createDecoderByType("video/avc");
codec.configure(format, surface, null, 0);
// surface 是 SurfaceTexture，解码后一帧直接以 OES 纹理形式上屏
```
Android上MediaCodec的输出surface是SurfaceTexture，对应一个GL_TEXTURE_EXTERNAL_OES纹理。这个纹理尺寸就是打包后的大图尺寸（2W×H 或 W×2H）。

###### 自定义Fragment Shader采样两次

重点是这个片段着色器（PAG源码里对应YUVBuffer.cpp/VideoComposition的渲染pipeline），简化后长这样:    
```glsl
  #extension GL_OES_EGL_image_external : require
  precision mediump float;

  uniform samplerExternalOES uTexture; // MediaCodec 输出的 OES 纹理
  varying vec2 vTexCoord;              // 当前像素在"原始画面"中的归一化坐标 (0~1)

  // 从 CPU 传进来的：alpha 区域起点占大图的比例
  uniform vec2 uAlphaStart;   // 水平打包时 = (0.5, 0.0)
  uniform vec2 uScale;        // 原画面在大图里占的归一化尺寸，水平打包时 = (0.5, 1.0)

  void main() {
      // 1) 采样 RGB：在大图的左半（或上半）
      vec2 rgbCoord = vTexCoord * uScale;
      vec3 rgb = texture2D(uTexture, rgbCoord).rgb;

      // 2) 采样 Alpha：相同的局部坐标，但整体偏移到右半（或下半）
      vec2 alphaCoord = rgbCoord + uAlphaStart;
      float alpha = texture2D(uTexture, alphaCoord).r; // 灰度图 R=G=B，取 R 即可

      // 3) 合并，并做 premultiplied alpha
      gl_FragColor = vec4(rgb * alpha, alpha);
  }
```
 三个关键点：

 1. 一个像素做两次纹理采样（同一张大纹理的不同区域），代价非常低，GPU 本来就是为这个干的。
2. 取灰度图的 R 分量当 alpha：因为编码时 R=G=B=alpha，解码回来可能因为 YUV
  色度子采样略有误差，但灰度视觉上无感。
3. 预乘 alpha（premultiplied）：rgb * alpha 这一步把输出变成预乘格式，方便后续和其他图层做标准blending：finalColor = src + dst * (1 - src.a)。这是 PAG 渲染树统一的颜色空间约定。

###### 渲染目标

Shader输出的RGBA会渲染到一个中间FBO（帧缓冲对象），它在PAG的渲染树里就是一个普通的"带透明度的图层纹理"，和矢量图层、其它BMP图层一起进入最终合成阶段，通过标准OpenGL blending叠在一起。

这就是为什么PAG能做到**"BMP 图层和矢量图层无缝叠加"**——从合成器的视角看它们都是带alpha的RGBA纹理，根本不关心上游是矢量画的还是视频解出来的。




###### 演进：iOS/macOS 上用 HEVC 原生 alpha

HEVC（H.265）从 2019 年起支持 alpha auxiliary picture（Apple 叫 HEVC with
  Alpha），可以在编码里直接带透明通道。PAG 在 iOS 11+、macOS 上已经用这个替代 side-by-side方案，好处:    

- 文件小 20%+（不用冗余存一份灰度图）；
- 解码直接得到 RGBA，省一次 shader 采样；
- Apple VideoToolbox 原生硬解。

Android端因为MediaCodec对HEVC-Alpha支持度参差不齐（部分芯片厂商不支持），目前仍以 H.264 + side-by-side 为主。



  
  
  
  
  
