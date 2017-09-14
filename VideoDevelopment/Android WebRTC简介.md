Android WebRTC简介
===


![image](https://github.com/CharonChui/Pictures/blob/master/webrtc-logo.png?raw=true)
              

WebRTC简介
---

[WebRTC](https://webrtc.org/)


`WebRTC`名称源自网页实时通信(Web Real-Time Communication`)的缩写，是一个支持网页浏览器进行实时语音对话或视频对话的技术，是谷歌2010年以6820万美元收购`Global IP Solutions`公司而获得的一项技术。`Google`于2011年6月3日开源的即时通讯项目，旨在使其成为客户端视频通话的标准。其实在`Google`将`WebRTC`开源之前，微软和苹果各自的通讯产品已占用很大市场份额（如`Skype`），`Google`也是为了快速扩大市场，所以将他给开源。在行业内得到了广泛的支持和应用，成为下一代视频通话的标准。更多介绍可以去官网上看。

`WebRTC`被誉为是`web`长期开源开发的一个新启元，是近年来`Web`开发的最重要创新。`WebRTC`允许`Web`开发者在其`web`应用中添加视频聊天或者点对点数据传输，不需要复杂的代码或者昂贵的配置。目前支持`Chrome`、`Firefox`和`Opera`，后续会支持更多的浏览器，它有能力达到数十亿的设备。

然而，`WebRTC`一直被误解为仅适合于浏览器。事实上，`WebRTC`最重要的一个特征是允许本地和`web`应用间的互操作，很少有人使用到这个特性。


本文主要以开源项目[AndroidRTC](https://github.com/pchab/AndroidRTC)为例。
下载后通过`Android Studio`打开，打开的时候可能会报错，说找不到`org.json:json:20090211`，这时只要将`AndroidRTC/webrtc-client/build.gradle`中的
```
dependencies {
    compile ('com.github.nkzawa:socket.io-client:0.4.1')
    compile 'io.pristine:libjingle:8871@aar'
}
```
修改成
```
dependencies {
    compile ('com.github.nkzawa:socket.io-client:0.4.1'){ // //webSocket相关
        exclude group: 'org.json', module: 'json' // 这句话是我新加的，不然会提示org.json:json:20090211找不到
    }

    compile 'io.pristine:libjingle:8871@aar' // //webRTC官方aar包
}
```
就可以了。 


运行一下你会发现进去会黑屏，这是因为需要依赖`node.js`服务。

`WebRTC Live Streaming`: 

- `Node.js server`
- `Desktop client`
- `Android client`

如果电脑上没有安装`nodejs`需要先安装`nodejs`，因为需要依赖`node.js`所以我们要先安装`node.js`。去[nodejs官网](https://nodejs.org/en/#download)下载安装。

然后是要本地下载[ProjectRTC](https://github.com/pchab/ProjectRTC)项目:    
- `git clone https://github.com/pchab/ProjectRTC.git`
- `cd ProjectRTC/`
- `npm install`
- `npm start`
服务默认会运行在3000端口，你可以在浏览器中打开`localhost:3000`。  

![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_web_1.png?raw=true)

如图就表示运行成功了。

接下来我们进行打开`android`客户端，你会发现，还是黑屏。
这是因为我们需要在项目中的`strings.xml`中将配置修改成本地电脑在局域网下的`ip`地址和运行`Node`服务端的端口。如下:   
```xml
<resources>
    <string name="app_name">AndroidRTC</string>
    <string name="host">172.16.55.27</string>
    <string name="port">3000</string>
    <string name="action_settings">Options</string>
</resources>
```
好了，现在再运行`android`项目，然后打开浏览器输入`localhost:3000`，在浏览器页面点击`start`，你就可以看到电脑摄像头获取的画面，如下图: 


![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_web_start.png?raw=true)


然后点击左边的`call`，就会去申请连手机端，接下来你就可以在浏览器和手机端看到画面了，如下图:

![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_web_call.png?raw=true)

手机端的画面:   

![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_android.jpg?raw=true)

既然连通了，下面就仔细分析一下代码:   
代码文件比较少，主要是三个类:  

![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_demo_file.png?raw=true)

其中在`WebRtcClient`中实现的逻辑最为核心，我们就从他入手。  

`Android`相关的`API`有`VideoCapturerAndroid`,`VideoRenderer`,`MediaStream`,`PeerConnection`,
和`PeerConnectionFactory`。下面我们将逐一讲解。

在开始之前，需要创建`PeerConnectionFactory`，这是`Android`上使用`WebRTC`最核心的`API`。

### `PeerConnectionFactory`

`Android WebRTC`最核心的类。理解这个类并了解它如何创建其他任何事情是深入了解`Android`中`WebRTC`的关键。
它和我们期望的方式还是有所不同的，所以我们开始深入挖掘它。

我们先到`WebRtcClient`文件中找到`PeerConnectionFactory`初始化的地方:   

![image](https://github.com/CharonChui/Pictures/blob/master/webrtc_factory_code.png?raw=true)

`initializeAndroidGlobals`的参数分别是:   
- `context`:应用上下文
- `initializeAudio`:是否初始化音频的布尔值。
- `initializeVideo`:是否初始化视频的布尔值。跳过这两个就允许跳过请求`API`的相关权限，例如数据通道应用。
- `videoCodecHwAcceleration`:是否允许硬件加速的布尔值。
- `renderEGLContext`:用来提供支持硬件视频解码，可以在视频解码线程中创建共享`EGL`上下文。
可以为空——在本文例子中硬件视频解码将产生`yuv420`帧而非`texture`帧。
- `initializeAndroidGlobals`也是返回布尔值，`true`表示一切OK，`false`表示有失败。


有了`peerConnectionFactory`实例，就可以从用户设备获取视频和音频，最终将其渲染到屏幕上。在`Android`中，我们需要了解`VideoCapturerAndroid`，`VideoSource`，`VideoTrack`和`VideoRenderer`。     
先从`VideoCapturerAndroid`开始:     

#### `VideoCapturerAndroid`

`VideoCapturerAndroid`其实是一系列`Camera API`的封装，为访问摄像头设备的流信息提供了方便。它允许获取多个摄像头设备信息，包括前置摄像头，或者后置摄像头。

在`WebRtcClient`类中的代码为:     
```java
private void setCamera(){
    localMS = factory.createLocalMediaStream("ARDAMS");
    if(pcParams.videoCallEnabled){
        MediaConstraints videoConstraints = new MediaConstraints();
        videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxHeight", Integer.toString(pcParams.videoHeight)));
        videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxWidth", Integer.toString(pcParams.videoWidth)));
        videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxFrameRate", Integer.toString(pcParams.videoFps)));
        videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("minFrameRate", Integer.toString(pcParams.videoFps)));

        videoSource = factory.createVideoSource(getVideoCapturer(), videoConstraints);
        localMS.addTrack(factory.createVideoTrack("ARDAMSv0", videoSource));
    }

    AudioSource audioSource = factory.createAudioSource(new MediaConstraints());
    localMS.addTrack(factory.createAudioTrack("ARDAMSa0", audioSource));

    mListener.onLocalStream(localMS);
}

private VideoCapturer getVideoCapturer() {
	// 获取前置摄像头的名称
    String frontCameraDeviceName = VideoCapturerAndroid.getNameOfFrontFacingDevice();
    // 创建前置摄像头对象
    return VideoCapturerAndroid.create(frontCameraDeviceName);
}
```

有了包含摄像流信息的`VideoCapturerAndroid`实例，就可以创建从本地设备获取到的包含视频流信息的`MediaStream`，从而发送给另一端。但做这些之前，我们首先研究下如何将自己的视频显示到应用上面。


#### `VideoSource/VideoTrack`

从`VideoCapturer`实例中获取一些有用信息，或者要达到最终目标————为连接端获取合适的媒体流，或者仅仅是将它渲染给用户，我们需要了解`VideoSource`和`VideoTrack`类。

`VideoSource`允许方法开启、停止设备捕获视频。这在为了延长电池寿命而禁止视频捕获的情况下比较有用。
`VideoTrack`是简单的添加`VideoSource`到`MediaStream`对象的一个封装。

我们通过代码看看它们是如何一起工作的。`capturer`是`VideoCapturer`的实例，`videoConstraints`是`MediaConstraints`的实例。
代码还是在上面贴出的`setCamera()`方法中:    

```java
private void setCamera(){
    localMS = factory.createLocalMediaStream("ARDAMS");
    if(pcParams.videoCallEnabled){
        MediaConstraints videoConstraints = new MediaConstraints();
        ......
        // 创建VideoSource
        videoSource = factory.createVideoSource(getVideoCapturer(), videoConstraints);
        // 创建VideoTrack
        localMS.addTrack(factory.createVideoTrack("ARDAMSv0", videoSource));
    }
    // 创建AudioSource
    AudioSource audioSource = factory.createAudioSource(new MediaConstraints());
    // 创建AudioSource
    localMS.addTrack(factory.createAudioTrack("ARDAMSa0", audioSource));

    mListener.onLocalStream(localMS);
}
```

#### `AudioSource/AudioTrack`

`AudioSource`和`AudioTrack`与`VideoSource`和`VideoTrack`相似，只是不需要`AudioCapturer`来获取麦克风，
代码同上。   

#### `VideoRenderer`

进入`VideoRenderer`，`WebRTC`库允许通过`VideoRenderer.Callbacks`实现自己的渲染。另外，
它提供了一种非常好的默认方式`VideoRendererGui`。简而言之，`VideoRendererGui`是一个`GLSurfaceView`，使用它可以绘制自己的视频流。我们通过代码看一下它是如何工作的，以及如何添加`renderer`到`VideoTrack`。
因为他是渲染的类，所以代码就不是在`WebRtcClient`里面了，在`RtcActivity`中，我们看下代码:    

```java
// Local preview screen position before call is connected.
private static final int LOCAL_X_CONNECTING = 0;
private static final int LOCAL_Y_CONNECTING = 0;
private static final int LOCAL_WIDTH_CONNECTING = 100;
private static final int LOCAL_HEIGHT_CONNECTING = 100;
// Local preview screen position after call is connected.
private static final int LOCAL_X_CONNECTED = 72;
private static final int LOCAL_Y_CONNECTED = 72;
private static final int LOCAL_WIDTH_CONNECTED = 25;
private static final int LOCAL_HEIGHT_CONNECTED = 25;
// Remote video screen position
private static final int REMOTE_X = 0;
private static final int REMOTE_Y = 0;
private static final int REMOTE_WIDTH = 100;
private static final int REMOTE_HEIGHT = 100;

private VideoRendererGui.ScalingType scalingType = VideoRendererGui.ScalingType.SCALE_ASPECT_FILL;
private GLSurfaceView vsv;
private VideoRenderer.Callbacks localRender;
private VideoRenderer.Callbacks remoteRender;
private WebRtcClient client;
private String mSocketAddress;
private String callerId;

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    getWindow().addFlags(
            LayoutParams.FLAG_FULLSCREEN
                    | LayoutParams.FLAG_KEEP_SCREEN_ON
                    | LayoutParams.FLAG_DISMISS_KEYGUARD
                    | LayoutParams.FLAG_SHOW_WHEN_LOCKED
                    | LayoutParams.FLAG_TURN_SCREEN_ON);
    setContentView(R.layout.main);
    mSocketAddress = "http://" + getResources().getString(R.string.host);
    mSocketAddress += (":" + getResources().getString(R.string.port) + "/");

    // 获取GLSurfaceView 
    vsv = (GLSurfaceView) findViewById(R.id.glview_call);
    vsv.setPreserveEGLContextOnPause(true);
    vsv.setKeepScreenOn(true);
    // 设置给VideoRendererGui
    VideoRendererGui.setView(vsv, new Runnable() {
        @Override
        public void run() {
            init();
        }
    });

    // 创建本地和远程的Render
    // local and remote render
    remoteRender = VideoRendererGui.create(
            REMOTE_X, REMOTE_Y,
            REMOTE_WIDTH, REMOTE_HEIGHT, scalingType, false);
    localRender = VideoRendererGui.create(
            LOCAL_X_CONNECTING, LOCAL_Y_CONNECTING,
            LOCAL_WIDTH_CONNECTING, LOCAL_HEIGHT_CONNECTING, scalingType, true);

    final Intent intent = getIntent();
    final String action = intent.getAction();

    if (Intent.ACTION_VIEW.equals(action)) {
        final List<String> segments = intent.getData().getPathSegments();
        callerId = segments.get(0);
    }
}

......


public void onLocalStream(MediaStream localStream) {
	// 将Render添加到VideoTrack中
    localStream.videoTracks.get(0).addRenderer(new VideoRenderer(localRender));
    VideoRendererGui.update(localRender,
            LOCAL_X_CONNECTING, LOCAL_Y_CONNECTING,
            LOCAL_WIDTH_CONNECTING, LOCAL_HEIGHT_CONNECTING,
            scalingType);
}

@Override
public void onAddRemoteStream(MediaStream remoteStream, int endPoint) {
    remoteStream.videoTracks.get(0).addRenderer(new VideoRenderer(remoteRender));
    VideoRendererGui.update(remoteRender,
            REMOTE_X, REMOTE_Y,
            REMOTE_WIDTH, REMOTE_HEIGHT, scalingType);
    VideoRendererGui.update(localRender,
            LOCAL_X_CONNECTED, LOCAL_Y_CONNECTED,
            LOCAL_WIDTH_CONNECTED, LOCAL_HEIGHT_CONNECTED,
            scalingType);
}

@Override
public void onRemoveRemoteStream(int endPoint) {
    VideoRendererGui.update(localRender,
            LOCAL_X_CONNECTING, LOCAL_Y_CONNECTING,
            LOCAL_WIDTH_CONNECTING, LOCAL_HEIGHT_CONNECTING,
            scalingType);
}

```

#### `MediaConstraints`

`MediaConstraints`是支持不同约束的`WebRTC`库方式的类，可以加载到`MediaStream`中的音频和视频轨道。对于大多数需要`MediaConstraints`的方法，一个简单的`MediaConstraints`实例就可以做到。

```java
private MediaConstraints pcConstraints = new MediaConstraints();
......
pcConstraints.mandatory.add(new MediaConstraints.KeyValuePair("OfferToReceiveAudio", "true"));
pcConstraints.mandatory.add(new MediaConstraints.KeyValuePair("OfferToReceiveVideo", "true"));
pcConstraints.optional.add(new MediaConstraints.KeyValuePair("DtlsSrtpKeyAgreement", "true"));

......
this.pc = factory.createPeerConnection(iceServers, pcConstraints, this);
```

#### `MediaStream`


本地看见自己后接下来就要想办法让对方看见自己。我们需要自己创建`MediaStream`。
接下来我们就研究如何添加本地的`VideoTrack`和`AudioTrack`来创建一个合适的`MediaStream`。
代码还是在前面分析的`setCamera()`方法中:   
```java
private MediaStream localMS;

private void setCamera(){
	// 创建MediaStream
	localMS = factory.createLocalMediaStream("ARDAMS");
	if(pcParams.videoCallEnabled){
	    MediaConstraints videoConstraints = new MediaConstraints();
	    videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxHeight", Integer.toString(pcParams.videoHeight)));
	    videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxWidth", Integer.toString(pcParams.videoWidth)));
	    videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("maxFrameRate", Integer.toString(pcParams.videoFps)));
	    videoConstraints.mandatory.add(new MediaConstraints.KeyValuePair("minFrameRate", Integer.toString(pcParams.videoFps)));

	    videoSource = factory.createVideoSource(getVideoCapturer(), videoConstraints);
	    // 添加VideoTrack
	    localMS.addTrack(factory.createVideoTrack("ARDAMSv0", videoSource));
	}

	AudioSource audioSource = factory.createAudioSource(new MediaConstraints());
	// 添加AudioTrack
	localMS.addTrack(factory.createAudioTrack("ARDAMSa0", audioSource));

	mListener.onLocalStream(localMS);
}
```

我们现在有了包含视频流和音频流的`MediaStream`实例，而且在屏幕上显示了我们的预览画面。下面要做的就该把这些信息传送给对方了。
这篇文章不会介绍如何建立自己的信号流，我们直接介绍对应的`API`方法，以及它们如何与`web`关联的。 `AppRTC`使用`autobahn`使得`WebSocket`连接到信号端。


#### `PeerConnection`

现在我们有了自己的`MediaStream`，就可以开始连接远端了。创建`PeerConnection`很简单，只需要`PeerConnectionFactory`的协助即可。    


```java
private PeerConnection pc;
private PeerConnectionFactory factory;
private LinkedList<PeerConnection.IceServer> iceServers = new LinkedList<>();


iceServers.add(new PeerConnection.IceServer("stun:23.21.150.121"));
iceServers.add(new PeerConnection.IceServer("stun:stun.l.google.com:19302"));

this.pc = factory.createPeerConnection(iceServers, pcConstraints, this);
```

参数的作用如下：

`iceServers`:连接到外部设备或者网络时需要用到这个参数。在这里添加STUN 和 TURN 服务器就允许进行连接，即使在网络条件很差的条件下。
`constraints`:前面介绍的`MediaConstraints`的实例，应该包含`offerToRecieveAudio`和`offerToRecieveVideo`。
`observer`:`PeerConnection.Observer`实例。

`PeerConnection`和`web`上的对应`API`很相似，包含了`addStream`、`addIceCandidate`、`createOffer`、`createAnswer`、`getLocalDescription`、`setRemoteDescription`和其他类似方法。为了了解如何协调所有工作在两点之间建立起通讯通道，我们先看下下面的几个类:  


#### `addStream`

这个是用来将`MediaStream`添加到`PeerConnection`中的,如同它的命名一样。如果你想要对方看到你的视频、听到你的声音，就需要用到这个方法。

#### `addIceCandidate`

一旦内部`IceFramework`发现有`candidates`允许其他方连接你时，就会创建`IceCandidates`。当通过`PeerConnectionObserver.onIceCandidate`传递数据到对方时，需要通过任何一个你选择的信号通道获取到对方的`IceCandidates`。使用`addIceCandidate`添加它们到`PeerConnection`，以便`PeerConnection`可以通过已有信息试图连接对方。

#### `createOffer/createAnswer`

这两个方法用于原始通话的建立。在`WebRTC`中，已经有了`caller`和`callee`的概念，一个是呼叫，一个是应答。`createOffer`是`caller`使用的，它需要一个`sdpObserver`，它允许获取和传输会话描述协议`Session Description Protocol (SDP)`给对方，还需要一个`MediaConstraint`。一旦对方得到了这个请求，它将创建一个应答并将其传输给`caller`。`SDP`是用来给对方描述期望格式的数据（如`video`、`formats`、`codecs`、`encryption`、`resolution`、 `size`等）。一旦`caller`收到这个应答信息，双方就相互建立的通信需求达成了一致，如视频、音频、解码器等。

#### `setLocalDescription/setRemoteDescription`

这个是用来设置`createOffer`和`createAnswer`产生的`SDP`数据的，包含从远端获取到的数据。它允许内部`PeerConnection`配置链接以便一旦开始传输音频和视频就可以开始真正工作。

#### `PeerConnection.Observer`

这个接口提供了一种监测`PeerConnection`事件的方法，例如收到`MediaStream`时，或者发现`iceCandidates`时，或者需要重新建立通讯时。这个接口必须被实现，以便你可以有效处理收到的事件，例如当对方变为可见时，向他们发送信号`iceCandidates`。


`WebRTC打开了人与人之间的通讯，对开发者免费，对终端用户免费。 它不仅仅提供了视频聊天，还有其他应用，比如健康服务、低延迟文件传输、种子下载、甚至游戏应用。
想要看到一个真正的`WebRTC`应用实例，请下载[appear.in](https://play.google.com/store/apps/details?id=appear.in.app&referrer=utm_source=tech.appear.in&utm_medium=blog&utm_campaign=android-launch-may15)。它在浏览器和本地应用间运行的相当完美，在同一个房间内最多可以8个人免费使用。不需要安装和注册。


最后总结一下: 

直播用的主流协议:  

- `RTMP`: (`Real Time Messaging Protocol`)实时消息传送协议是`Adobe Systems`公司为`Flash`播放器和服务器之间音频、视频和数据传输 开发的开放协议。目前直播的主流平台95%都是基于该协议做的。      

    - 优点：成熟，稳定，效果好等等
    - 缺点：有延迟，达不到实时互动等等

- `WebRTC`: 名称源自网页实时通信（`Web Real-Time Communication`）的缩写，是一个支持网页浏览器进行实时语音对话或视频对话的技术，是谷歌2010年以6820万美元收购`Global IP Solutions`公司而获得的一项技术。2011年5月开放了工程的源代码，在行业内得到了广泛的支持和应用，成为下一代视频通话的标准。      

    - 优点：实时互动，开发难度小等等
    - 缺点：太实时，如果网络不好，视频质量会严重下降，体验不好；考研服务器等等



那既然`WebRTC`这么好，那直接用来做直播不就好了。结果往往并没有想象中的那么完美。

`WebRTC`整体的技术并不适合做直播。`WebRTC`设计的初衷只是为了在两个浏览器/`native app`之间解决直接连接发送`media streaming/data`数据的，也就是所谓的`peer to peer`的通信，大多数的情况下不需要依赖于服务器的中转，因此一般在通信的逻辑上是一对一。而我们现在的直播服务大部分的情况下是一对多的通信，一个主播可能会有成千上万个接收端，这种方式用传统的`P2P`来实现是不可能的，所以目前直播的方案基本上都是会有直播服务器来做中央管理，主播的数据首先发送给直播服务器，直播服务器为了能够支持非常多用户的同事观看，还要通过边缘节点`CDN`的方式来做地域加速，所有的接收端都不会直接连接主播，而是从服务器上接收数据。

`WebRtc`只适合小范围（8人以内）音视频会议，不适合做直播：
1. 视频部分：vpx的编码器太弱，专利原因不能用264，做的好的都要自己改264/265代码才行。
2. 音频部分：音频只适合人声编码，对音乐和其他非人声的效果很糟糕。
3. 网络部分：对国内各种奇葩网络适应性太低，网络糟糕点或者人多点就卡。
4. 信号处理：同时用过`GIPS`和`WebRTC`进行对比，可以肯定目前开源的代码是`GIPS`阉割过的。
5. 使用规模：10人以内使用，超过10人就挂了，

现在的直播服务器请使用`nginx rtmp-module`架设，架设好了用`ffmpeg`命令行来测试播摄像头。主播客户端请使用`rtmp`进行推流给`rtmp-module`，粉丝请使用`rtmp/flv+http stream`进行观看，`PC-web`端的粉丝请使用`Flash NetStream`来观看，移动`web`端的粉丝请使用`hls/m3u8`来观看。如果你试验成功要上线了，出现压力了，那么把`nginx`分层（接入层+交换层），稍微改两行代码，如果资金不足以全国部署服务器，那么把`nginx-rtmp-module`换为`cdn`的标准直播服务，也可以直接调过`nginx`，一开始就用`cdn`的直播服务，比如网宿（斗鱼的直播服务提供商）。这是正道，别走弯路了。

那既然`WebRTC`不适合做直播，还写这么多干啥，`WebRTC`内部包含的技术模块是非常适合解决直播过程中存在的各种问题的，而且应该在大多数直播技术框架中都已经得到了部分应用，例如音视频数据的收发、音频处理回音消除降噪等。所以综上，可以使用`WebRTC`内部的技术模块来解决直播过程中存在的技术问题，但是不适合直接用`WebRTC`来实现直播的整体框架。


参考: 

- [Introduction to WebRTC on Android](https://tech.appear.in/2015/05/25/Introduction-to-WebRTC-on-Android/)   
- [Android WebRTC 编译](https://webrtc.org/native-code/android/)
- [Android WebRTC开源项目](https://github.com/pchab/AndroidRTC)
- [ProjectRTC](https://github.com/pchab/ProjectRTC)
- [Getting Started with WebRTC](https://www.html5rocks.com/en/tutorials/webrtc/basics/)
- [WebRTC Github](https://github.com/webrtc)
- [WebRTC GoogleSource](https://chromium.googlesource.com/external/webrtc)
- [Android WebRTC](https://webrtc.org/native-code/android/)
- [Samples](https://github.com/webrtc/samples/)
- [WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Streams_API)
- [WebRTC适合做直播吗？](https://www.zhihu.com/question/25497090)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 