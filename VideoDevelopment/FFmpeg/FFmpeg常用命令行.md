FFmpeg常用命令行
===
### ffmpeg

1.分离视频音频流

ffmpeg -i input_file -vcodec copy -an output_file_video　　//分离视频流
ffmpeg -i input_file -acodec copy -vn output_file_audio　　//分离音频流

2.视频解复用

ffmpeg –i test.mp4 –vcodec copy –an –f m4v test.264
ffmpeg –i test.avi –vcodec copy –an –f m4v test.264

3.视频转码

ffmpeg –i test.mp4 –vcodec h264 –s 352*278 –an –f m4v test.264              //转码为码流原始文件
ffmpeg –i test.mp4 –vcodec h264 –bf 0 –g 25 –s 352*278 –an –f m4v test.264  //转码为码流原始文件
ffmpeg –i test.avi -vcodec mpeg4 –vtag xvid –qsame test_xvid.avi            //转码为封装文件
//-bf B帧数目控制，-g 关键帧间隔控制，-s 分辨率控制

4.视频封装

ffmpeg –i video_file –i audio_file –vcodec copy –acodec copy output_file

5.视频剪切

ffmpeg –i test.avi –r 1 –f image2 image-%3d.jpeg        //提取图片
ffmpeg -ss 0:1:30 -t 0:0:20 -i input.avi -vcodec copy -acodec copy output.avi    //剪切视频
//-r 提取图像的频率，-ss 开始时间，-t 持续时间

6.视频录制

ffmpeg –i rtsp://192.168.3.205:5555/test –vcodec copy out.avi

7.YUV序列播放

ffplay -f rawvideo -video_size 1920x1080 input.yuv

8.YUV序列转AVI

ffmpeg –s w*h –pix_fmt yuv420p –i input.yuv –vcodec mpeg4 output.avi

常用参数说明：

主要参数： -i 设定输入流 -f 设定输出格式 -ss 开始时间 视频参数： -b 设定视频流量，默认为200Kbit/s -r 设定帧速率，默认为25 -s 设定画面的宽与高 -aspect 设定画面的比例 -vn 不处理视频 -vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器 音频参数： -ar 设定采样率 -ac 设定声音的Channel数 -acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器 -an 不处理音频

------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

0.压缩转码mp4文件

ffmpeg -i input.avi -s 640x480 output.avi

ffmpeg -i input.avi -s vga output.avi

 

1、将文件当做直播送至live

ffmpeg -re -i localFile.mp4 -c copy -f flv rtmp://server/live/streamName

2、将直播媒体保存至本地文件

 

ffmpeg -i rtmp://server/live/streamName -c copy dump.flv

3、将其中一个直播流，视频改用h264压缩，音频不变，送至另外一个直播服务流

 

ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv rtmp://server/live/h264Stream

 

4、将其中一个直播流，视频改用h264压缩，音频改用faac压缩，送至另外一个直播服务流

ffmpeg -i rtmp://server/live/originalStream -c:a libfaac -ar 44100 -ab 48k -c:v libx264 -vpre slow -vpre baseline -f flv rtmp://server/live/h264Stream

5、将其中一个直播流，视频不变，音频改用faac压缩，送至另外一个直播服务流

ffmpeg -i rtmp://server/live/originalStream -acodec libfaac -ar 44100 -ab 48k -vcodec copy -f flv rtmp://server/live/h264_AAC_Stream

6、将一个高清流，复制为几个不同视频清晰度的流重新发布，其中音频不变

ffmpeg -re -i rtmp://server/live/high_FMLE_stream -acodec copy -vcodec x264lib -s 640×360 -b 500k -vpre medium -vpre baseline rtmp://server/live/baseline_500k -acodec copy -vcodec x264lib -s 480×272 -b 300k -vpre medium -vpre baseline rtmp://server/live/baseline_300k -acodec copy -vcodec x264lib -s 320×200 -b 150k -vpre medium -vpre baseline rtmp://server/live/baseline_150k -acodec libfaac -vn -ab 48k rtmp://server/live/audio_only_AAC_48k

7、功能一样，只是采用-x264opts选项

ffmpeg -re -i rtmp://server/live/high_FMLE_stream -c:a copy -c:v x264lib -s 640×360 -x264opts bitrate=500:profile=baseline:preset=slow rtmp://server/live/baseline_500k -c:a copy -c:v x264lib -s 480×272 -x264opts bitrate=300:profile=baseline:preset=slow rtmp://server/live/baseline_300k -c:a copy -c:v x264lib -s 320×200 -x264opts bitrate=150:profile=baseline:preset=slow rtmp://server/live/baseline_150k -c:a libfaac -vn -b:a 48k rtmp://server/live/audio_only_AAC_48k

8、将当前摄像头及音频通过DSSHOW采集，视频h264、音频faac压缩后发布

ffmpeg -r 25 -f dshow -s 640×480 -i video=”video source name”:audio=”audio source name” -vcodec libx264 -b 600k -vpre slow -acodec libfaac -ab 128k -f flv rtmp://server/application/stream_name

9、将一个JPG图片经过h264压缩循环输出为mp4视频

ffmpeg.exe -i INPUT.jpg -an -vcodec libx264 -coder 1 -flags +loop -cmp +chroma -subq 10 -qcomp 0.6 -qmin 10 -qmax 51 -qdiff 4 -flags2 +dct8x8 -trellis 2 -partitions +parti8x8+parti4x4 -crf 24 -threads 0 -r 25 -g 25 -y OUTPUT.mp4

10、将普通流视频改用h264压缩，音频不变，送至高清流服务(新版本FMS live=1)

ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv “rtmp://server/live/h264Stream live=1〃


------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

 

1.采集usb摄像头视频命令：

ffmpeg -t 20 -f vfwcap -i 0 -r 8 -f mp4 cap1111.mp4

 

./ffmpeg -t 10 -f vfwcap -i 0 -r 8 -f mp4 cap.mp4

具体说明如下：我们采集10秒，采集设备为vfwcap类型设备，第0个vfwcap采集设备（如果系统有多个vfw的视频采集设备，可以通过-i num来选择），每秒8帧，输出方式为文件，格式为mp4。

 

2.最简单的抓屏：

ffmpeg -f gdigrab -i desktop out.mpg 

 

3.从屏幕的（10,20）点处开始，抓取640x480的屏幕，设定帧率为5 ：

ffmpeg -f gdigrab -framerate 5 -offset_x 10 -offset_y 20 -video_size 640x480 -i desktop out.mpg 

 

4.ffmpeg从视频中生成gif图片：

ffmpeg -i capx.mp4 -t 10 -s 320x240 -pix_fmt rgb24 jidu1.gif


### ffplay
ffplay是以FFmpeg框架未基础，外加渲染音视频的库libSDL构建的媒体文件播放器。

ffplay [选项] [输入文件]

```
ffplay /Users/xxx/Desktop/111.mp4
ffplay http://219.151.31.38/liveplay-kk.rtxapp.com/live/program/live/hnwshd/4000000/mnf.m3u8
```
如果按s键就可以进入frame-step模式，即按s键一次就会播放下一帧图像。

##### 主要选项

- '-x width'        强制以 "width" 宽度显示
- '-y height'       强制以 "height" 高度显示
- '-an'             禁止音频
- '-vn'             禁止视频
- '-ss pos'         跳转到指定的位置(秒)
- '-t duration'     播放 "duration" 秒音/视频
- '-bytes'          按字节跳转
- '-nodisp'         禁止图像显示(只输出音频)
- '-f fmt'          强制使用 "fmt" 格式
- '-window_title title'  设置窗口标题(默认为输入文件名)
- '-loop number'    循环播放 "number" 次(0将一直循环)
- '-showmode mode'  设置显示模式
	- 可选的 mode ：
	- '0, video'    显示视频
	- '1, waves'    显示音频波形
	- '2, rdft'     显示音频频带
	- 默认值为 'video'，你可以在播放进行时，按 "w" 键在这几种模式间切换
- '-i input_file'   指定输入文件
- '-sync type'          设置主时钟为音频、视频、或者外部。默认为音频。主时钟用来进行音视频同步
- '-threads count'      设置线程个数
- '-autoexit'           播放完成后自动退出
- '-exitonkeydown'      任意键按下时退出
- '-exitonmousedown'    任意鼠标按键按下时退出
- '-acodec codec_name'  强制指定音频解码器为 "codec_name"
- '-vcodec codec_name'  强制指定视频解码器为 "codec_name"
- '-scodec codec_name'  强制指定字幕解码器为 "codec_name"

如果希望从视频的第30秒开始播放，播放10秒钟的文件，则可以使用如下命令 
`ffplay -ss 30 -t 10 /Users/xxx/Desktop/111.mp4`

##### 音画同步

ffplay也是一个视频播放器，所以不得不提出来的一个问题是：音画同步。
ffplay的音画同步的实现方式其实有三种，分别是：
- 以音频为主时间轴作为同步源
- 以视频为主时间轴作为同步源
- 以外部时钟为主时间轴作为同步源。

下面就以音频为主时间轴来作为同步源来作为案例进行讲解，而且ffplay默认也是以音频为基准进行对齐的，那么以音频作为对齐基准是如何实现的呢？

首先需要说明的是，播放器接收到的视频帧或者音频帧，内部都是会有时间戳（PTS时钟）来标识它实际应该在什么时刻展示，实际的对齐策略如下：
    比较视频当前的播放时间和音频当前的播放时间，如果视频播放过快，则通过加大延迟或者重复播放来降低视频播放速度，如果视频播放慢了，则通过减小延迟或者丢帧来追赶音频播放的时间点。
    关键就在于音视频时间的比较和延迟的计算，当前在比较的过程中会设置一个阈值，如果超过预设的阈值就应该作出调整（丢帧或者重复渲染），这就是整个对齐策略。

在使用ffplay的时候，我们可以明确的指定使用那种对齐方式，比如： 
`ffplay test.mp4 -sync audio`
上面这个命令显式的指定了使用以音频为基准进行音视频同步的方式播放视频文件，当然这也是ffplay的默认播放设置。
`ffplay test.mp4 -sync video`
上面这个命令显式的指定了使用以视频为基准进行音视频同步的方式播放视频文件。
`ffplay test.mp4 -sync ext`
上面这个命令显式的指定了使用外部时钟为基准进行音视频同步的方式播放视频文件。


### ffprobe
ffprobe是ffmpeg命令行中用来查看媒体文件格式的工具。 

命令格式: 
`ffprobe [文件名]`

```
ffprobe /Users/xxx/Desktop/111.mp4

ffprobe version 4.4 Copyright (c) 2007-2021 the FFmpeg developers
  built with Apple clang version 12.0.5 (clang-1205.0.22.9)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.4_2 --enable-shared --enable-pthreads --enable-version3 --cc=clang --host-cflags= --host-ldflags= --enable-ffplay --enable-gnutls --enable-gpl --enable-libaom --enable-libbluray --enable-libdav1d --enable-libmp3lame --enable-libopus --enable-librav1e --enable-librubberband --enable-libsnappy --enable-libsrt --enable-libtesseract --enable-libtheora --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libxvid --enable-lzma --enable-libfontconfig --enable-libfreetype --enable-frei0r --enable-libass --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libspeex --enable-libsoxr --enable-libzmq --enable-libzimg --disable-libjack --disable-indev=jack --enable-avresample --enable-videotoolbox
  libavutil      56. 70.100 / 56. 70.100
  libavcodec     58.134.100 / 58.134.100
  libavformat    58. 76.100 / 58. 76.100
  libavdevice    58. 13.100 / 58. 13.100
  libavfilter     7.110.100 /  7.110.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  9.100 /  5.  9.100
  libswresample   3.  9.100 /  3.  9.100
  libpostproc    55.  9.100 / 55.  9.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '/Users/xuchuanren/Desktop/111.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Multimedia Cloud Transcode (cloud.baidu.com)
    comment         : Content Adaptive Encoding 3.0
  Duration: 00:00:41.52, start: 0.000000, bitrate: 163 kb/s  // 视频的时长是41秒52毫秒，开始播放时间是0，，整个文件的比特率是163Kbit/s
  Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 1280x720 [SAR 1:1 DAR 16:9], 91 kb/s, 25 fps, 25 tbr, 12800 tbn, 50 tbc (default)
  // 上面这一行的信息是，第一个流是视频流，编码格式是h264格式(封装格式是AVC1)，每一帧的数据表示为yuv420p，分辨率为1280*720.这路流的比特率为91Kbit/s，帧率为每秒钟25帧
    Metadata:
      handler_name    : VideoHandler
      vendor_id       : [0][0][0][0]
  Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 65 kb/s (default)
  // 上面这一行的信息是，第二个流为音频流，编码方式为ACC(封装格式为MP4A),并且采用的Profile是LC规格，采样率是44.1KHz，声道是立体声，这路流的比特率是65Kbit/s
    Metadata:
      handler_name    : SoundHandler
      vendor_id       : [0][0][0][0]
```

查看流的格式信息
ffprobe -show_format -i /Users/xxx/Desktop/111.mp4 

```
[FORMAT]
filename=/Users/xxx/Desktop/111.mp4
nb_streams=2
nb_programs=0
format_name=mov,mp4,m4a,3gp,3g2,mj2
format_long_name=QuickTime / MOV
start_time=0.000000
duration=41.518000
size=848895
bit_rate=163571
probe_score=100
TAG:major_brand=isom
TAG:minor_version=512
TAG:compatible_brands=isomiso2avc1mp41
TAG:encoder=Multimedia Cloud Transcode (cloud.baidu.com)
TAG:comment=Content Adaptive Encoding 3.0
[/FORMAT]
```
查看每一帧信息
ffprobe -show_frames -i /Users/xxx/Desktop/111.mp4 

查看包信息
ffprobe -show_packets -i /Users/xxx/Desktop/111.mp4 



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 