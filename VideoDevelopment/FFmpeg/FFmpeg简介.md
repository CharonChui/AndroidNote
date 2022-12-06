FFmpeg简介
===

FFmpeg是一个开源免费跨平台的视频和音频流方案，属于自由软件，采用LGPL或GPL许可证（依据你选择的组件）。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多codec都是从头开发的。

主要的开发库：

- libavutil：包含一些公共的工具函数，包括随机数生成、数据结构、核心多媒体工具等；
- libavcodec：用于各种类型声音/图像encode/decode编解码库；
- libavformat：用于各种音视频封装格式（mp4/AVI/Flv等）的生成和解析muxer/demuxer，包括获取解码所需信息以生成解码上下文结构和读取音视频帧等功能；
- libavdevice:读取电脑（或者其他设备上）的多媒体设备的数据 或者输出数据到指定的多媒体设备上；
- libswresample: 用于音频采样采样数据（PCM）转换的库；
- libswscale：用于视频场景比例缩放、色彩映射转换的库；
- libavfilter: 包含媒体滤波器的库

主要的工具集：

- ffmpeg：一个命令行工具，可用于格式转换、解码、编码等；
- ffsever：一个 HTTP 、RTSP的实时广播流媒体服务器；
- ffplay：是一个简单的播放器，使用ffmpeg 库解析和解码，通过SDL显示；
- ffprobe : 一个多媒体流分析工具。 它从多媒体流中收集信息 并且以人类和机器可读的形式打印出来。


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 