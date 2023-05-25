WebRTC
===

WebRTC，名称源自网页实时通信（Web Real-Time Communication）的缩写，是一个支
持网页浏览器进行实时语音通话或视频聊天的技术，是谷歌2010 年以6820 万美元收购
Global IP Solutions 公司而获得的一项技术。
WebRTC 提供了实时音视频的核心技术，包括音视频的采集、编解码、网络传输、显示
等功能，并且还支持跨平台：windows，linux，mac，android。
虽然WebRTC 的目标是实现跨平台的Web 端实时音视频通讯，但因为核心层代码的
Native、高品质和内聚性，开发者很容易进行除Web 平台外的移殖和应用。很长一段时间内
WebRTC 是业界能免费得到的唯一高品质实时音视频通讯技术。

WebRTC 是一项在浏览器内部进行实时视频和音频通信的技术，是谷歌于2010 年以
wwwwww..hheelllloottoonnggttoonngg..ccoom ©®
6820 万美元收购VoIP 软件开发商Global IT Solutions 公司而获得一项技术，谷歌于2011 年
6 月3 日开源该项目。
谷歌在官方博客中称：“我们希望让浏览器成为实时通信的创新地所在，到目前为止，
实时通信需要使用受版权保护的信号处理技术，并通过插件或下载客户端才能实现，而
WebRTC 则允许开发人员使用HTML 和JavaScript API 来创建实时应用。”


1.webrtc 是什么
浏览器为音视频获取传输提供的接口
2.webrtc 可以做什么
浏览器端到端的进行音视频聊天、直播、内容传输
3.数据传输需要些什么
IP、端口、协议
客户端、服务端


![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/webrtc_artic_1.png?raw=true)
1.绿色部分是WebRTC 核心部分（核心库）
2.紫色部分是JS 提供的API（应用层）
整体是应用层调用核心层。
核心层，第一层C++ API
提供给外面的接口。最主要的是(PeerConnedtion 对等连接）。
核心层，第二层Session

上下文管理层（音视频）。
核心层，第三层[最重要的部分]
音视频引擎：编解码;音频缓冲BUFFER 防止音频网络抖动NetEQ;回音消除;降噪；静音检
测；
视频引擎：编解码；jitter buffer 防止视频网络抖动；图像处理增强；
传输：SRTP 加密后的RTP；多路复用；P2P（STUN+TURN+ICE)
核心层，第四层,硬件相关层
音视频采集; 网络IO


WebRTC 实现了基于网页的视频会议，标准是WHATWG 协议，目的是通过浏览器提供
简单的javascript 就可以达到实时通讯（Real-Time Communications (RTC)）能力。
WebRTC（Web Real-Time Communication）项目的最终目的主要是让Web 开发者能够基
于浏览器（Chrome/FireFox/...）轻易快捷开发出丰富的实时多媒体应用，而无需下载安装任
何插件，Web 开发者也无需关注多媒体的数字信号处理过程，只需编写简单的Javascript 程
序即可实现，W3C 等组织正在制定Javascript 标准API，目前是WebRTC 1.0 版本，Draft 状
态；另外WebRTC 还希望能够建立一个多互联网浏览器间健壮的实时通信的平台，形成开发
者与浏览器厂商良好的生态环境。同时，Google 也希望和致力于让WebRTC 的技术成为
HTML5 标准之一，可见Google 布局之深远。
WebRTC 提供了视频会议的核心技术，包括音视频的采集、编解码、网络传输、显示等
功能，并且还支持跨平台：windows，linux，mac，android。


- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
