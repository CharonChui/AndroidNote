# HLS协议

[HLS(HTTP Live Streaming)协议](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/Introduction/Introduction.html)：2009年由苹果公司实现的基于HTTP的流媒体传输协议，可实现流媒体的直播和点播。它的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。

HLS协议规定:  

- 视频的封装格式是TS
- 视频的编码格式为H264，音频编码格式为MP3、AAC或者AC-3
- 除了TS视频文件本身，还定义了用来控制播放的m3u8文件(文本文件)

HLS协议由三部分组成:  

- HTTP:传输协议
- m3u8:索引文件
- TS:音视频媒体信息

HLS点播，基本上就是常见的分段HTTP点播，不同在于，它的分段非常小。相对于常见的流媒体直播协议，例如RTMP协议、RTSP协议、MMS协议等，HLS直播最大的不同在于，直播客户端获取到的，并不是一个完整的数据流。HLS协议在服务器端将直播数据流存储为连续的、很短时长的媒体文件（MPEG-TS格式），而客户端则不断的下载并播放这些小文件。因为服务器端总是会将最新的直播数据生成新的小文件，这样客户端只要不停的按顺序播放从服务器获取到的文件，就实现了直播。由此可见，基本上可以认为，HLS是以点播的技术方式来实现直播。由于数据通过HTTP协议传输，所以完全不用考虑防火墙或者代理的问题，而且分段文件的时长很短，客户端可以很快的选择和切换码率，以适应不同带宽条件下的播放。不过HLS的这种技术特点，决定了它的延迟一般总是会高于普通的流媒体直播协议。它也解决了RTMP协议存在的一些问题，例如RTMP协议不使用标准的HTTP接口传输数据(TCP、UDP端口)，所以在一些特殊的网络环境下可能被防火墙屏蔽掉，而HLS使用的是HTTP协议传输数据(80端口)，不会遇到被防火墙屏蔽的情况。

## TS

TS(Transport Stream)，全程为MPEG2-TS。它的特点就是要求从视频流的任一片段开始都是可以独立解码的。DVD节目中的MPEG2格式，确切的说是MPEG2-PS，全程是Program Stream，而TS的全称是Transport Stream。MPEG2-PS主要应用于存储的具有固定市场的节目，如DVD电影，而MPEG2-TS则主要应用于实时传送的节目，比如实时广播的电视节目。这两种格式的主要区别是什么呢？简单地打个比喻说，你将DVD上的[VOB](https://baike.baidu.com/item/VOB)文件的前面一截cut掉（或者干脆就是数据损坏），那么就会导致整个文件无法解码了，而电视节目是你任何时候打开电视机都能解码（收看）的，所以，MPEG2-TS格式的特点就是要求从[视频流](https://baike.baidu.com/item/视频流)的任一片段开始都是可以独立解码的。



在开始一个流媒体会话时，客户端会下载一个包含元数据的extended M3U(m3u8)  playlist文件，用于寻找可用的媒体流(将视频分为一个个视频小分片，然后用m3u8索引表进行管理，由于客户端下载到的视频都是5-10秒的完整数据，所以视频的流畅性很好，但是同样也引入了很大的延迟(一般延迟在10-30s左右))。

- 是一个索引地址/播放列表，通过FFmpeg将本地的xxx.mp4进行切片处理，生成m3u8播放列表（索引文件）和N多个  .ts文件，并将其（m3u8、N个ts）放置在本地搭建好的webServer服务器的指定目录下，我就可以得到一个可以实时播放的网址，我们把这个m3u8地址复制到 VLC 上就可以实时观看！ 在 HLS 流下，本地视频被分割成一个一个的小切片，一般10秒一个，这些个小切片被 m3u8管理，并且随着终端的FFmpeg  向本地拉流的命令而实时更新，影片进度随着拉流的进度而更新，播放过的片段不在本地保存，自动删除，直到该文件播放完毕或停止，ts  切片会相应的被删除，流停止，影片不会立即停止，影片播放会滞后于拉流一段时间


<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/transport_stream_2x.png" style="zoom:50%;" />

- Media encoder(媒体编码)

媒体编码器获取到音视频设备的实时信号，将其编码后压缩用于传输

- Stream segmenter(流切片器)

  通常是一个软件，将流进行切片，切片的同时会创建一个索引文件(index file)，索引文件会包含这些切片的引用。

从左到右讲，左下方的inputs的视频源是什么格式都无所谓，他与server之间的通信协议也可以任意（比如RTMP），总之只要把视频数据传输到服务器上即可。这个视频在server服务器上被转换成HLS格式的视频（既TS和m3u8文件）文件。细拆分来看server里面的Media encoder的是一个转码模块负责将视频源中的视频数据转码到目标编码格式（H264）的视频数据，视频源的编码格式可以是任何的视频编码格式。转码成H264视频数据之后用硬件打包到MPEG-2(MPEG-2 Transport Stream)的传输流中，传输流再经过stream segmenter模块，它的工作是把MPEG-2传输流分散为小片段然后保存为一个或多个系列的.ts格式的媒体文件，结果就是index file（m3u8）和ts文件了。图中的Distribution其实只是一个普通的HTTP文件服务器，然后客户端只需要访问一级index文件的路径就会自动播放HLS视频流了。

### 多码率适配流(Master Playlist)

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/indexing_2x.png" style="zoom:70%;" />

客户端播放HLS视频流的逻辑其实非常简单，先下载一级Index file，它里面记录了二级索引文件（Alternate-A、Alternate-B、Alternate-C）的地址，然后客户端再去下载二级索引文件，二级索引文件中又记录了TS文件的下载地址，这样客户端就可以按顺序下载TS视频文件并连续播放。

#### 一级index文件

```bash
#EXTM3U    // m3u8文件头
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1064000
1000kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=564000
500kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=282000
250kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2128000
2000kbps.m3u8
```

bandwidth指定视频流的比特率，PROGRAM-ID无用无需关注，每一个#EXT-X-STREAM-INF的下一行是二级index文件的路径，可以用相对路径也可以用绝对路径。例子中用的是相对路径。这个文件中记录了不同比特率视频流的二级index文件路径，客户端可以自己判断自己的现行网络带宽，来决定播放哪一个视频流。也可以在网络带宽变化的时候平滑切换到和带宽匹配的视频流。客户端会默认选择码率最高的情况，如果发现码率达不到，会请求降低码率的流。

```css
#EXTM3U
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-TARGETDURATION:10  // 指定当前视频流中的单个切片(ts)文件的最大时长(秒)
#EXTINF:10,     // 指定每个媒体段ts的持续时间
2000kbps-00001.ts
#EXTINF:10,
2000kbps-00002.ts
#EXTINF:10,
2000kbps-00003.ts
#EXTINF:10,
2000kbps-00004.ts
#EXTINF:10,

... ...

#EXTINF:10,
2000kbps-00096.ts
#EXTINF:10,
2000kbps-00097.ts
#EXTINF:10,
2000kbps-00098.ts
#EXTINF:10,
2000kbps-00099.ts
#EXTINF:10,
2000kbps-00100.ts
#ZEN-TOTAL-DURATION:999.66667
#ZEN-AVERAGE-BANDWIDTH:2190954
#ZEN-MAXIMUM-BANDWIDTH:3536205
#EXT-X-ENDLIST
```

二级文件实际负责给出ts文件的下载地址，这里同样使用了相对路径。#EXTINF表示每个ts切片视频文件的时长。#EXT-X-TARGETDURATION指定当前视频流中的切片文件的最大时长，也就是说这些ts切片的时长不能大于#EXT-X-TARGETDURATION的值。#EXT-X-PLAYLIST-TYPE:VOD的意思是当前的视频流并不是一个直播流，而是点播流，换句话说就是该视频的全部的ts文件已经被生成好了，#EXT-X-ENDLIST这个表示视频结束，有这个标志同时也说明当前的流是一个非直播流。

#### 播放模式

- **点播VOD**的特点就是当前时间点可以获取到所有index文件和ts文件，二级index文件中记录了所有ts文件的地址。这种模式允许客户端访问全部内容。上面的例子中就是一个点播模式下的m3u8的结构。
- **Live** 模式就是实时生成M3u8和ts文件。它的索引文件一直处于动态变化的，播放的时候需要不断下载二级index文件，以获得最新生成的ts文件播放视频。如果一个二级index文件的末尾没有#EXT-X-ENDLIST标志，说明它是一个Live视频流。

客户端在播放VOD模式的视频时其实只需要下载一次一级index文件和二级index文件就可以得到所有ts文件的下载地址，除非客户端进行比特率切换，否则无需再下载任何index文件，只需顺序下载ts文件并播放就可以了。但是Live模式下略有不同，因为播放的同时，新ts文件也在被生成中，所以客户端实际上是下载一次二级index文件，然后下载ts文件，再下载二级index文件（这个时候这个二级index文件已经被重写，记录了新生成的ts文件的下载地址）,再下载新ts文件，如此反复进行播放。

### 单码率适配流(Media Playlist)

[http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8](https://link.jianshu.com?t=http%3A%2F%2Fwww.modrails.com%2Fvideos%2Fpassenger_nginx.mov)

```
#EXTM3U                           // m3u8文件头
#EXT-X-VERSION:3                  // 协议版本，不写默认是1
#EXT-X-MEDIA-SEQUENCE:304240      // 第一个ts文件的序列号
#EXT-X-TARGETDURATION:10          // 每个ts文件的最大时长
#EXTINF:10.000,                   // 每个ts文件的持续时间
cctv1hd-1585920024000.ts          // ts文件的url
#EXTINF:10.000,
cctv1hd-1585920034000.ts
#EXTINF:10.000,
cctv1hd-1585920044000.ts
#EXTINF:10.000,
cctv1hd-1585920054000.ts
#EXTINF:10.000,
cctv1hd-1585920064000.ts
#EXTINF:10.000,
cctv1hd-1585920074000.ts
```





## HLS 的优势

- 客户端支持简单, 只需要支持 HTTP 请求即可, HTTP 协议无状态, 只需要按顺序下载媒体片段即可.
- 使用 HTTP 协议网络兼容性好, HTTP 数据包也可以方便地通过防火墙或者代理服务器, CDN 支持良好.
- Apple 的全系列产品支持, 由于 HLS 是苹果提出的, 所以在 Apple 的全系列产品包括 iphone, ipad, safari 都不需要安装任何插件就可以原生支持播放 HLS, 现在, Android 也加入了对 HLS 的支持.
- 自带多码率自适应, Apple 在提出 HLS 时, 就已经考虑了码流自适应的问题.

## HLS 的劣势

- 相比 RTMP 这类长连接协议, 延时较高, 难以用到互动直播场景.HLS 理论延时 = 1 个切片的时长 + 0-1个 td (td 是 EXT-X-TARGETDURATION, 可简单理解为播放器取片的间隔时间) + 0-n 个启动切片(苹果官方建议是请求到 3 个片之后才开始播放) + 播放器最开始请求的片的网络延时(网络连接耗时)。为了追求低延时效果, 可以将切片切的更小, 取片间隔做的更小, 播放器未取到 3 个片就启动播放. 但是, 这些优化方式都会增加 HLS 不稳定和出现错误的风险.
- 对于点播服务来说, 由于 TS 切片通常较小, 海量碎片在文件分发, 一致性缓存, 存储等方面都有较大挑战.























