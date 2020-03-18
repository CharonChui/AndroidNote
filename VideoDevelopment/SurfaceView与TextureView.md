SurfaceView与TextureView
===


## SurfaceView

在说SurfaceView之前，需要先说一下几个相关的部分。

### `Surface`简介

- `Surface`就是“表面”的意思，可以简单理解为内存中的一段绘图缓冲区。在`SDK`的文档中，对`Surface`的描述是这样的：“`Handle onto a raw buffer that is being managed by the screen compositor`”，
    翻译成中文就是“由屏幕显示内容合成器`(screen compositor)`所管理的原生缓冲器的句柄”，   这句话包括下面两个意思:     

    - 通过`Surface`（因为`Surface`是句柄）就可以获得原生缓冲器以及其中的内容。就像在`C`语言中，可以通过一个文件的句柄，就可以获得文件的内容一样；
    - 原生缓冲器（`rawbuffer`）是用于保存当前窗口的像素数据的。

- 简单的说`Surface`对应了一块屏幕缓冲区，每个`Window`对应一个`Surface`，任何`View`都是画在`Surface`上的，传统的`view`共享一块屏幕缓冲区，所有的绘制必须在`UI`线程中进行
- 我们不能直接操作`Surface`实例，要通过`SurfaceHolder`，在`SurfaceView`中可以通过`getHolder()`方法获取到`SurfaceHolder`实例。
- `Surface`是一个用来画图形的地方，但是我们知道画图都是在一个`Canvas`对象上面进行的，*`Surface`中的`Canvas`成员，是专门用于提供画图的地方，就像黑板一样，其中的原始缓冲区是用来保存数据的地方，
    `Surface`本身的作用类似一个句柄，得到了这个句柄就可以得到其中的`Canvas`、原始缓冲区以及其他方面的内容，所以简单的说`Surface`是用来管理数据的(句柄)*。
    
### `SurfaceView`简介      

- 简单的说`SurfaceView`就是一个有`Surface`的`View`里面内嵌了一个专门用于绘制的`Surface`,`SurfaceView`控制这个`Surface`的格式和尺寸以及绘制位置。

SurfaceView就是在Window上挖一个洞，它就是显示在这个洞里，其他的View是显示在Window上，所以View可以显式在 SurfaceView之上，你也可以添加一些层在SurfaceView之上。

```java
if (mWindow == null) {  ß
    mWindow = new MyWindow(this);  
    mLayout.type = mWindowType;  
    mLayout.gravity = Gravity.LEFT|Gravity.TOP;  
    mSession.addWithoutInputChannel(mWindow, mWindow.mSeq, mLayout,  
    mVisible ? VISIBLE : GONE, mContentInsets);  
}  
```
很明显，每个`SurfaceView`创建的时候都会创建一个`MyWindow`，`new MyWindow(this)`中的`this`正是`SurfaceView`自身，因此将`SurfaceView`和`window`绑定在一起，而前面提到过每个`window`对应一个`Surface`，
所以`SurfaceView`也就内嵌了一个自己的`Surface`，可以认为`SurfaceView`是来控制`Surface`的位置和尺寸。传统`View`及其派生类的更新只能在`UI`线程，然而`UI`线程还同时处理其他交互逻辑，
这就无法保证`view`更新的速度和帧率了，而`SurfaceView`可以用独立的线程来进行绘制，因此可以提供更高的帧率，例如游戏，摄像头取景等场景就比较适合用`SurfaceView`来实现。

- `Surface`是纵深排序`(Z-ordered)`的，这表明它总在自己所在窗口的后面。
- `Surfaceview`提供了一个可见区域，只有在这个可见区域内的`Surface`部分内容才可见，可见区域外的部分不可见，所以可以认为**`SurfaceView`就是展示`Surface`中数据的地方**,`Surface`就是管理数据的地方，
    `SurfaceView`就是展示数据的地方，只有通过`SurfaceView`才能展现`Surface`中的数据。           
    ![image](https://github.com/CharonChui/Pictures/blob/master/SurfaceView.png?raw=true)   


- `Surface`的排版显示受到视图层级关系的影响，它的兄弟视图结点会在顶端显示。这意味者`Surface`的内容会被它的兄弟视图遮挡，这一特性可以用来放置遮盖物`(overlays)`(例如，文本和按钮等控件)。
    注意，如果`Surface`上面有透明控件，那么它的每次变化都会引起框架重新计算它和顶层控件的透明效果，这会影响性能。`surfaceview`变得可见时，`surface`被创建；`surfaceview`隐藏前，`surface`被销毁。
    这样能节省资源。如果你要查看`surface`被创建和销毁的时机，可以重载`surfaceCreated(SurfaceHolder)`和`surfaceDestroyed(SurfaceHolder)`。
    **`SurfaceView`的核心在于提供了两个线程：`UI`线程和渲染线程**,两个线程通过“双缓冲”机制来达到高效的界面适时更新。而这个双缓冲可以理解为，SurfaceView在更新视图时用到了两张Canvas，一张frontCanvas和一张backCanvas。每次实际显示的是frontCanvas，backCanvas存储的是上一次更改前的视图，当使用lockCanvas（）获取画布时，得到的实际上是backCanvas而不是正在显示的frontCanvas，之后你在获取到的backCanvas上绘制新视图，再unlockCanvasAndPost（canvas）此视图，那么上传的这张canvas将替换原来的frontCanvas作为新的frontCanvas，原来的frontCanvas将切换到后台作为backCanvas。例如，如果你已经先后两次绘制了视图A和B，那么你再调用lockCanvas（）获取视图，获得的将是A而不是正在显示的B，之后你将重绘的C视图上传，那么C将取代B作为新的frontCanvas显示在SurfaceView上，原来的B则转换为backCanvas。()
    不用画布，直接在窗口上进行绘图叫做无缓冲绘图。用了一个画布，将所有内容都先画到画布上，在整体绘制到窗口上，就该叫做单缓冲绘图，那个画布就是一个缓冲区。用了两个画布，一个进行临时的绘图，一个进行最终的绘图，这样就叫做双缓冲。)


SurfaceView的优缺点:     

一般的Activity包含的多个View会组成View hierachy的树形结构，只有最顶层的DectorView才是对WMS可见的，这个DecorView在WMS中有一个对应的WindowState，再SurfaceFlinger中有对应的Layer，而SurfaceView正因为它有自己的Surface，有自己的Window，它在WMS中有对应的WindowState，在SurfaceFlinger中有Layer。虽然在App端它仍在View hierachy中，但在Server端(WMS和SurfaceFlinger)中，它与宿主窗口是分离的。这样的好处是对这个Surface的渲染可以放到单独的线程中去做，渲染时可以有自己的GL context。因为它不会影响主线程对时间的响应。所以它的优点就是可以在独立的线程中绘制，不影响主线程，而且使用双缓冲机制，播放视频时画面更顺畅。但是这也有缺点，因为这个Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移、缩放等动画，它也不能放在其它ViewGroup中，SurfaceView不能嵌套使用，而且不能使用某些View的特性，例如View.setAlpha()。


***从Android7.0开始，SurfaceView的窗口位置与其他View渲染同步更新。 这意味着在屏幕上平移和缩放SurfaceView不会导致渲染失真。***


### `SurfaceHolder`简介

显示一个`Surface`的抽象接口，使你可以控制`Surface`的大小和格式以及在`Surface`上编辑像素，和监视`Surace`的改变。这个接口通常通过`SurfaceView`类实现。
简单的说就是我们无法直接操作`Surface`只能通过`SurfaceHolder`这个接口来获取和操作`Surface`。
`SurfaceHolder`中提供了一些`lockCanvas()`:获取一个`Canvas`对象，并锁定之。所得到的`Canvas`对象，其实就是`Surface`中一个成员。加锁的目的其实就是为了在绘制的过程中，
`Surface`中的数据不会被改变。`lockCanvas`是为了防止同一时刻多个线程对同一`canvas`写入。


*从设计模式的角度来看,`Surface、SurfaceView、SurfaceHolder`实质上就是`MVC(Model-View-Controller)`，`Model`就是模型或者说是数据模型，更简单的可以理解成数据，在这里也就是`Surface`，
`View`就是视图，代表用户交互界面，这里就是`SurfaceView`,`SurfaceHolder`就是`Controller.`*


## TextureView

因为上面所说的SurfaceView不在主窗口中，它没法做动画没法使用一些View的特性方法，所以在Android 4.0中引入了TextureView，它是一个结合了View和SurfaceTexture的View对象。它不会在WMS中单独创建窗口，而是作为View hierachy中的一个普通view，因此它可以和其他普通View一样进行平移、旋转等动画。但是TextureView必须在硬件加速的窗口中，它显示的内容流数据可以来自App进程或者远程进程。    

TextureView重载了draw()方法，其中主要SurfaceTexture中收到的图像数据作为纹理更新到对应的HardwareLayer中。

SurfaceTexture.OnFrameAvailableListener用于通知TextureView内容流有新图像到来。SurfaceTextureListener接口用于让TextureView的使用者知道SurfaceTexture已准备好，这样就可以把SurfaceTexture交给相应的内容源。Surface为BufferQueue的Producer接口实现类，使生产者可以通过它的软件或硬件渲染接口为SurfaceTexture内部的BufferQueue提供graphic buffer。

SurfaceTexture可以用作非直接输出的内容流，这样就提供二次处理的机会。与SurfaceView直接输出相比，这样会有若干帧的延迟。同时，由于它本身管理BufferQueue，因此内存消耗也会稍微大一些。
TextureView是一个可以把内容流作为外部纹理输出在上面的View, 它本身需要是一个硬件加速层。


### SurfaceTexture

SurfaceTexture是Surface和OpenGL ES(GLES)纹理的组合。SurfaceTexture用于提供输出到GLES 纹理的Surface。

SurfaceTexture 包含一个应用是其使用方的BufferQueue。当生产方将新的缓冲区排入队列时，onFrameAvailable() 回调会通知应用。然后，应用调用updateTexImage()，这会释放先前占有的缓冲区，从队列中获取新缓冲区并执行EGL调用，从而使GLES可将此缓冲区作为外部纹理使用。

## SurfaceView vs TextureView

与 SurfaceView 相比，TextureView 具有更出色的 Alpha 版和旋转处理能力，但在视频上以分层方式合成界面元素时，SurfaceView 具有性能方面的优势。当客户端使用 SurfaceView 呈现内容时，SurfaceView 会为客户端提供单独的合成层。如果设备支持，SurfaceFlinger 会将单独的层合成为硬件叠加层。当客户端使用 TextureView 呈现内容时，界面工具包会使用 GPU 将 TextureView 的内容合成到 View 层次结构中。对内容进行的更新可能会导致其他 View 元素重绘，例如，如果其他 View 位于 TextureView 上方。View 呈现完成后，SurfaceFlinger 会合成应用界面层和所有其他层，以便每个可见像素合成两次。

***注意：受 DRM 保护的视频只能在叠加平面上呈现。支持受保护内容的视频播放器必须使用 SurfaceView 进行实现。***

| 项目        |  SurfaceView   |  TextureView  |
| --------   | -----:  | :----:  |
| 内存      |  低   |   高     |
| 耗电        |   低   |   高   |
| 绘制        |    及时    |  1-3帧延迟  |
| 动画和截图        |    不支持    |  支持  |



- 在android 7.0上系统surfaceview的性能比TextureView更有优势，支持对象的内容位置和包含的应用内容同步更新，平移、缩放不会产生黑边。 - 在7.0以下系统如果使用场景有动画效果，可以选择性使用TextureView
- 由于失效(invalidation)和缓冲的特性，TextureView增加了额外1~3帧的延迟显示画面更新
- TextureView总是使用GL合成，而SurfaceView可以使用硬件overlay后端，可以占用更少的内存
- TextureView的内部缓冲队列导致比SurfaceView使用更多的内存
- SurfaceView： 内部自己持有surface，surface 创建、销毁、大小改变时系统来处理的，通过surfaceHolder 的callback回调通知。当画布创建好时，可以将surface绑定到MediaPlayer中。SurfaceView如果为用户可见的时候，创建SurfaceView的SurfaceHolder用于显示视频流解析的帧图片，如果发现SurfaceView变为用户不可见的时候，则立即销毁SurfaceView的SurfaceHolder，以达到节约系统资源的目的




    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 