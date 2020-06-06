RTMP
===





http://billchan.me/2019/04/27/livestreamprotocol/



Real Time Messaging Protocol(实时消息传送协议):是Adobe Systems公司为`Flash`播放器和服务器之间音频、视频和数据传输开发的开放协议。协议基于`TCP`，是一个协议族(默认端口1935)，包括`RTMP`基本协议及`RTMPT/RTMPS/RTMPE`等多种变种。`RTMP` 是一种设计用来进行实时数据通信的网络协议，主要用来在`Flash/AIR`平台和支持`RTMP`协议的流媒体/交互服务器之间进行音视频和数据通信。市面上绝大部分PC秀场使用的都是它，他有低延迟(2s左右)、稳定性高、技术完善、高支持度、编码兼容性高等特点。但是RTMP协议不使用标准的HTTP接口传输数据(TCP、UDP端口)，所以在一些特殊的网络环境下可能被防火墙屏蔽掉。

RTMP的整体流程为:  

视频采集器 -> 支持RTMP的视频编码器 -> 网络传输  -> 流媒体服务器 -> 网络 -> 客户端







https://blog.csdn.net/qq_37382077/article/details/103386289



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 