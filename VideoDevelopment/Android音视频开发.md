Android音视频开发
===






二、Android音视频的开发
·播放流程: 获取流—>解码—>播放

·录制播放路程: 录制音频视频—>剪辑—>编码—>上传服务器 别人播放.

·直播过程 : 录制音视频—>编码—>流媒体传输—>服务器—>流媒体传输到其他app—>解码—>播放 

几个重要的环节

·录制音视频 AudioRecord/MediaRecord

·视频剪辑 mp4parser 或ffmpeg

·音视频编码 aac&h264

·上传大文件 网络框架,进度监听,断点续传

·流媒体传输 流媒体传输协议rtmp rtsp hls

·音视频解码 aac&h264

·渲染播放 MediaPlayer











正如上文所说，Android本身对音视频流媒体传输协议，以及音视频编解码支持有限。所以对于直播类应用，要自己解码。

3.1 调研过程

·vitamio

·webRTC

·ffmpeg

·vlc

·ijkplayer

先说下 vitamio，这个是功能很强大，但是企业收费版的，个人用户可以玩玩。

目前WebRTC，只适合小范围（8人以内）音视频会议，不适合做直播。

接下来介绍下 ffmpeg、vlc、ijkplayer以及选择方案。

ffmpeg是一个非常强大的音视频编解码开源库，目前市场上流行的播放器，大部分都是基于此开发的，包括暴风、腾讯等等以及上面提到的vitamio、vlc、ijkplayer。

vlc支持android开发，ijkplayer也支持。通过反编译网易云音乐，以及YY等音视频app，发现网易云音乐、斗鱼用的ijkplayer，YY用的VLC。二、Android音视频的开发
·播放流程: 获取流—>解码—>播放

·录制播放路程: 录制音频视频—>剪辑—>编码—>上传服务器 别人播放.

·直播过程 : 录制音视频—>编码—>流媒体传输—>服务器—>流媒体传输到其他app—>解码—>播放 

几个重要的环节

·录制音视频 AudioRecord/MediaRecord

·视频剪辑 mp4parser 或ffmpeg

·音视频编码 aac&h264

·上传大文件 网络框架,进度监听,断点续传

·流媒体传输 流媒体传输协议rtmp rtsp hls

·音视频解码 aac&h264

·渲染播放 MediaPlayer











正如上文所说，Android本身对音视频流媒体传输协议，以及音视频编解码支持有限。所以对于直播类应用，要自己解码。

3.1 调研过程

·vitamio

·webRTC

·ffmpeg

·vlc

·ijkplayer

先说下 vitamio，这个是功能很强大，但是企业收费版的，个人用户可以玩玩。

目前WebRTC，只适合小范围（8人以内）音视频会议，不适合做直播。

接下来介绍下 ffmpeg、vlc、ijkplayer以及选择方案。

ffmpeg是一个非常强大的音视频编解码开源库，目前市场上流行的播放器，大部分都是基于此开发的，包括暴风、腾讯等等以及上面提到的vitamio、vlc、ijkplayer。

vlc支持android开发，ijkplayer也支持。通过反编译网易云音乐，以及YY等音视频app，发现网易云音乐、斗鱼用的ijkplayer，YY用的VLC。








---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 