弹幕

---



[OpenDanmaku](https://github.com/linsea/OpenDanmaku)

[bilibili Android开源弹幕引擎.烈焰弹幕使](https://github.com/Bilibili/DanmakuFlameMaster)



![image](https://github.com/CharonChui/Pictures/blob/master/danmaku.gif?raw=true)

### 弹幕目前使用场景



1. 直播:通过直播间im可以实时的去显示用户新发的弹幕
2. 短视频:目前没有im，可以通过单独的接口来请求弹幕，实时性有缺陷



### 播放器上弹幕实现原理

   布局上分为: 最下层的播放器视图、中间层的弹幕视图层、最上层的操作视图层。

- 自定义View显示弹幕:VideoView上面盖上一层全透明的DanmakuView，该DanmakuView去不断的渲染显示弹幕。可能会有卡顿。

- SurfaceView显示弹幕:内存及耗电量低、绘制及时，但是不支持动画和截图。    

  打个比方可以是两个SurfaceView，下面的一个SurfaceView显示视频画面，上面的一层透明SurfaceView用于绘制弹幕。          

- TextureView显示弹幕: 内存和耗电稍高、绘制会有1-3帧的延迟、但是支持动画和截图。   

  一般的Activity包含的多个View会组成View hierachy的树形结构，只有最顶层的DectorView才是对WMS可见的，这个DecorView在WMS中有一个对应的WindowState，再SurfaceFlinger中有对应的Layer，而SurfaceView正因为它有自己的Surface，有自己的Window，它在WMS中有对应的WindowState，在SurfaceFlinger中有Layer。虽然在App端它仍在View hierachy中，但在Server端(WMS和SurfaceFlinger)中，它与宿主窗口是分离的。这样的好处是对这个Surface的渲染可以放到单独的线程中去做，渲染时可以有自己的GL context。因为它不会影响主线程对时间的响应。所以它的优点就是可以在独立的线程中绘制，不影响主线程，而且使用双缓冲机制，播放视频时画面更顺畅。但是这也有缺点，因为这个Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移、缩放等动画，它也不能放在其它ViewGroup中，SurfaceView不能嵌套使用，而且不能使用某些View的特性，例如View.setAlpha()。

  ***从Android7.0开始，SurfaceView的窗口位置与其他View渲染同步更新。 这意味着在屏幕上平移和缩放SurfaceView不会导致渲染失真。***



### 需要考虑的问题



1. 弹幕的大小及位置是否可以用户随意调整

2. 弹幕内移动的item出现的角度是否可按类型不同控制

3. 弹幕能否支持图文以及标签

4. 弹幕内移动的item出现的位置是否可随机

5. 弹幕有没有交互功能，例如点击弹幕中的头像进入用户页面

6. 弹幕过于密集集中是的展示策略(是否舍弃)

7. 小屏和全屏切换后的展示策略(展示内容的空间大小不同)

8. 性能的影响

9. 弹幕太多挡住视频中的人物画面

   B站移动端的视频能够明显的看到人物边缘的锯齿.        

   ![](https://github.com/CharonChui/Pictures/blob/master/bilibili_danmaku.jpeg?raw=true)

   对观看还是有一定的体验影响，感觉可以从上面第四条的角度去出发，对于密集的弹幕做一个展示策略，例如短时间内大量的重复或者相近的弹幕可以舍弃(只有发弹幕的本人可以看到)或者是合并。

   - 抠图:任务在弹幕渲染的层级之上渲染。
   - 视频预处理:服务端做离线的视频人脸监测，生成蒙层，，弹幕遇到人脸的位置就消失，除了人脸就显示。

### 实现思考



#### 自定义View



#### 自定义SurfaceView



#### 自定义TextureView

#### OpenGL实现








