# 简介

除了前面讲的Apple的HLS，还有Adobe HTTP Dynamic Streaming (HDS)、Microsoft Smooth Streaming (MSS)。他们各家的协议原理大致相同，但是格式又不一样，也无法兼容，所以Moving Picture Expert Group (MPEG) 就把大家叫到了一起，呼吁大家一起来制定一个标准的，然后就有了[MPEG-DASH](https://www.encoding.com/mpeg-dash/),它的主要目标是形成IP网络承载单一格式的流媒体并提供高效与高质量服务的统一方案，解决多制式传输方案(HTTP Live  Streaming, Microsoft Smooth Streaming, HTTP Dynamic  Streaming)并存格局下的存储与服务能力浪费、运营高成本与复杂度、系统间互操作弱等问题。

[DASH(MPEG-DASH)](https://mpeg.chiariglione.org/standards/mpeg-dash/)全称为Dynamic Adaptive Streaming over HTTP.是由MPEG和ISO批准的独立于供应商的国际标准，它是一种基于HTTP的使用TCP传输协议的流媒体传输技术。它诞生的目的是为了统一标准，因此是兼容SmoothStreaming和HLS的.同时支持TS profile和 ISO  profile，支持节目观看等级控制，支持父母锁. mpeg  dash支持的DRM类型包括PlayReady和Marlin，而HLS支持的是AES128（密钥长度为128位的高级加密标准Advanced  Encryption Standard）加密类型。

MPEG-DASH是一种自适应比特率流技术，可根据实时网络状况实现动态自适应下载。和HLS, HDS技术类似， 都是把视频分割成一小段一小段，  通过HTTP协议进行传输，客户端得到之后进行播放；不同的是MPEG-DASH支持MPEG-2 TS、MP4(最新的HLS也支持了MP4)等多种格式,  可以将视频按照多种编码切割, 下载下来的媒体格式既可以是ts文件也可以是mp4文件，MPEG—DASH技术与编解码器无关，可使用H.265，H.264，VP9等任何编解码器进行编码。

安卓平台上的ExoPlayer支持MPEG-DASH。另外，三星、索尼、飞利浦、松下的一些较新型号的智能电视支持MPEG—DASH。Google的Chromecast、YouTube 和Netflix 也已支持MPEG-DASH。

![dash_compare](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_compare.png)

HLS在16年支持了fmp4，在17年支持了4K。



## 思想

![dash_main_idea](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_main_idea.png)

DASH的核心思想是，在服务端，视频被提前编好多种码率，并且被切成固定长度的视频片段，存放到HTTP服务器中，当客户端播放时，通过HTTP请求向服务器请求视频切片，并根据网络状况的变化，请求相应质量的视频切片，从而达到对网络带宽的最大利用，并且保证播放流畅。可以实现不同画质内容无缝切换。所以在 YouTube 切换画质时完全不会黑屏，更不会影响观看。

![image-20200406163508642](https://raw.githubusercontent.com/CharonChui/Pictures/master/a_dash_scenario.png)



DASH的整个流程:

内容生成服务器(编码模块、封装模块) -> 流媒体服务器(MPD、媒体文件、HTTP服务器) <-> DASH客户端(控制引擎、媒体引擎、HTTP接入容器)

![image-20200406164238038](https://raw.githubusercontent.com/CharonChui/Pictures/master/mpd_hierarchical_data.png)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MPD id="0564e940-122b-42bb-9d56-98f3def67247" profiles="urn:mpeg:dash:profile:isoff-main:2011" type="static" availabilityStartTime="2016-01-14T09:30:35.000Z" publishTime="2016-01-14T09:31:33.000Z" mediaPresentationDuration="P0Y0M0DT0H2M17.000S" minBufferTime="P0Y0M0DT0H0M1.000S" bitmovin:version="1.6.0" xmlns:ns2="http://www.w3.org/1999/xlink" xmlns="urn:mpeg:dash:schema:mpd:2011" xmlns:bitmovin="http://www.bitmovin.net/mpd/2015">
    <Period>
        <AdaptationSet mimeType="video/mp4" codecs="avc1.42c00d">
            <SegmentTemplate media="../video/$RepresentationID$/dash/segment_$Number$.m4s" initialization="../video/$RepresentationID$/dash/init.mp4" duration="120119" startNumber="0" timescale="30000"/>
            <Representation id="1920_9000000" bandwidth="9000000" width="3840" height="1920" frameRate="30"/>
            <Representation id="1080_5000000" bandwidth="5000000" width="2160" height="1080" frameRate="30"/>
            <Representation id="720_3000000" bandwidth="3000000" width="1440" height="720" frameRate="30"/>
            <Representation id="540_1500000" bandwidth="1500000" width="1080" height="540" frameRate="30"/>
            <Representation id="360_1000000" bandwidth="1000000" width="720" height="360" frameRate="30"/>
        </AdaptationSet>
        <AdaptationSet lang="en" mimeType="audio/mp4" codecs="mp4a.40.2" bitmovin:label="english stereo">
            <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
            <SegmentTemplate media="../audio/$RepresentationID$/dash/segment_$Number$.m4s" initialization="../audio/$RepresentationID$/dash/init.mp4" duration="191472" startNumber="0" timescale="48000"/>
            <Representation id="1_stereo_128000" bandwidth="128000" audioSamplingRate="48000"/>
        </AdaptationSet>
    </Period>
</MPD>

```



### MPD

DASH采用3GPP AHS中定义的MPD(Media Presentation Description)作为媒体文件的描述文件（manifest），作用类似HLS的m3u8文件。MPD文件以XML格式组织，用于描述segment的信息，比如时间、url、视频分辨率、码率等。   
DASH会通过media presentation description (MPD)将视频内容切片成一个很短的文件片段，每个切片都有多个不同的码率，DASH Client可以根据网络的情况选择一个码率进行播放，支持在不同码率之间无缝切换。
DASH中的重要概念

### Period

标注了视频的时长信息，也可以看做是更新mpd文件的最长时长，MPD文件包含一个或者多个片段(Period)，它表示时间轴上的一段时间，每个片段都有一个起始时间和结束时间，并且包含了一个或者多个适配集合(Adaptation Set)。每个适配集合提供了一个或者多个媒体组件的信息，并包含了多种不同的码率。每个适配集合又是由多个呈现(Representation)组成，每个呈现就是同一个视频的不同特征的版本，如码率、分辨率等特征。由于每个的视频都要被切成固定长度的切片，因此每个呈现包括多个视频切片(Segment)，每个视频切片都有一个URL地址，这样客户端就可以通过这个地址向服务器发送HTTP GET请求获取该片段。同一个Period内，意味着可用的媒体内容及其各个可用码率（Representation）不会发生变更。直播情况下，“可能”需要周期地去服务器更新MPD文件，服务器可能会移除旧的已经过时的Period,或是添加新的Period。新的Period中可能会添加新的可用码率或去掉上一个Period中存在的某些码率(Representation)。

### Adaptation Set

包含了媒体呈现的形式，（视频/音频/字幕）。 一个Period由一个或者多个Adaptationset组成。Adaptationset由一组可供切换的不同码率的码流（Representation)组成，这些码流中可能包含一个（ISO profile)或者多个(TS profile)media content components，因为ISO profile的mp4或者fmp4 segment中通常只含有一个视频或者音频内容，而TS profile中的TS segment同时含有视频和音频内容，当同时含有多个media component content时，每个被复用的media content component将被单独描述

### Representation

包含不同的码率、编码方式、帧率信息等。每个Adaptationset包含了一个或者多个Representations,一个Representation包含一个或者多个media streams，每个media stream对应一个media content component。为了适应不同的网络带宽，实际播放的时候，视频会在一个AdaptationSet中的不同Representaiton 之间切换码率，可能会从一个Representation切换到另外一个Representation，会依次请求该Representaiton下不同Segment序列。

### Segments

每一个具体的片段。（1,2,4,6,10s …） Segments可以包含任何媒体数据，关于容器，官方提供了两种建议: ISO base media file format(比如MP4文件格式)和MPEG-2 Transport Stream。

每个Representation会划分为多个Segment。Segment分为4类，其中，最重要的是：Initialization Segment（每个Representation都包含1个Init Seg），Media Segment（每个Representation的媒体内容包含若干Media Seg）

- Initialization Segment：

  Representation的Segments一般都采用1个Init Segment+多个普通Segment的方式，还有一种形式就是Self Initialize Segment，这种形式没有单独的Init Segment，初始化信息包括在了各个Segment中。Init Segment中包含了解封装需要的全部信息，比如Representation中有哪些音视频流，各自的编码格式及参数。对于 ISO profile来说(容器为MP4)，包含了moov box,H264的sps/pps数据等关键信息存放于此（avCc box）。另外，同一个Adaptation set的多个Representation还可能共享同一个Init Segment，该种情况下，对于ISO profile来说，诸如stsd box，avCc box等重要的box会含有多个entry，每个entry对应一个Representation，第一个entry对应第一个Representation，第二个entry对应第二个Representation，以此类推。

- Subsegment

   Segment可能进一步划分为subsegment，每个subsegment由数个Acess Unit组成，Segment index提供了subsegment相对于Segment的字节范围和presentation time range 。客户端可以先下载Segment index。
   
   
   
   `*_init.mp4`: 初始的mp4文件，相当于视频头，在这个头文件中包含了完整的视频元信息(moov)，具体的可以使用 `MP4Box  -info` 查看。
   
   `*.m4s`: 即上面提到的Segments文件，每个m4s仅包含媒体信息 (moof + mdat)，而播放器是不能直接播放这个文件的，需要用支持DASH的播放器从init文件开始播放。



确切的说，当Adaptation set的属性@segmentAlignment为真（true）时，同一个Adaptation  set中的多个Representation之中的媒体段是对齐的，因此，当从一个Representation  A切换到另一个Representation B时，若Representation  A的第N个媒体段已经下载完成，切换时可直接下载Representation B的第N+1个媒体段。

DASH对媒体段定义了三种方式：

BaseURL：单段表示

SegmentList：段列表

SegmentTemplate：段模板

单段表示是最简单的：每个Representation只有一个媒体段。用BaseURL表示。举例如下：



### SAP和无缝切换以及SEEK

  SAP：Stream Acess Point，可以简单理解为I帧，每个Segment的第一个帧都是SAP，因此Seek时可直接Seek到某一个Segment的起始位置，利用Init Segment+Seek到的某个Segment的数据，在解封装后可实现完美解码。一般来说，同一个Adaptation set中的多个Representation是Segment Align的（当Adaptation set的属性@segmentAlignment不为false时），因此，当从Representation A切换到Representation B时，如果当前Representation A的第N个Segment已经下载完成，切换时直接下载Representation B的第N+1个Segment即可。

码率自适应切换算法

- 基于带宽的码率自适应切换算法
- 基于缓存的码率自适应切换算法

![image-20200406165443536](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_change_compare.png)

此基于时间缓存的码率自适应算法对网络带宽变化反应敏感，能够有效的提高平均码率，但同时码率切换次数过大，尤其是在网络状况波动很大的情况下，这势必会造成用户体验的下降。

- MSS拥有最高的平均码率和较少的切换次数

- HLS的切换次数最少，但是以最低的平均码率作为代价

- HDS不能保证流程的播放

- DASH有足够的竞争力，也具有巨大的提升空间。



## 为什么使用DASH

- DASH支持多种编码，支持H.265、H.264、VP9等。

- DASH支持MultiDRM，支持PlayReady、Widewine，采用通用加密技术，支持终端自带DRM，可以大幅度降低DRM投资成本。

- DASH支持多种文件封装，支持MPEG-4、MPEG-2 TS。

- DASH支持多种CDN对接，采用相同的封装描述对接多厂家CDN。

- DASH支持直播、点播、录制等丰富的视频特性。

- DASH支持动态码率适配Adaptive Bitrate (ABR) ，支持多码率平滑切换。

- DASH支持缩略型描述以支持快速启动。

    





### HLS vs DASH



- 在标准HTTP服务器上的用法： HLS和DASH均可在常规HTTP服务器（例如Nginx，Apache等）上使用。
- 多个音频通道: 特别是对于多语言内容，重要的是能够在各个语言的不同音频通道之间进行切换。 DASH和HLS都可以做到这一点。
- 字幕和标题: 为了给视频添加字幕，通常创建一个单独的文件，例如，文件可以具有WebVTT格式。然后从清单（即.m3u8或.mpd文件）中引用该文件。
- 插入广告: 通常，可以在HLS和DASH的实时流中插入广告。为此，只需交换单个视频块。 DASH为此提供了一种有效的方法：标准化的界面允许有效地插入广告。
- 快速频道切换: 您可以在各个通道之间切换的速度取决于最大的子段（块）。块越小，通道更改速度越快。正如引言中已经提到的，HLS块通常长约10秒，而DASH块通常长2至4秒。因此，DASH在这方面领先一步。小块还具有降低代码效率的缺点。具有较小块的播放列表必须比具有较大块的播放列表更频繁地更新。这意味着包含较短视频片段的播放列表必须通过HTTP更频繁地更新。

# **结构与编码**

MPEG-DASH支持TS和MP4 / ISO BMFF媒体段。HLS只支持MPEG-2 TS。DASH媒体段通常比HLS短，2至4秒比较常见。DASH不需要特定的编解码器。视频可以使用H264编码，也可以用其他编码，VP9和H265也是比较受欢迎的编码。

一般而言，与HLS相比，DASH可以提供实质上更低的端对端延迟。这对于现场直播的工作流程很重要。此外， MPEG-DASH的基于模板的MPD不需要更新，可以在网络边缘服务器进行缓存，HLS则需要周期性地更新传播多次。

DASH支持索引和基于时间的模版，播放器能够基于公开的时钟，如NTPS，进行同步。这对于多相机的情况下，多个播放器之间同步会比较容易。

# **DRM**

DASH和HLS之间的另一个关键区别是它支持DRM。可是，在DASH中不存在一个单一通用的DRM解决方案。例如，Google的Chrome支持Widevine，而Microsoft的Internet  Explorer支持PlayReady。然而，通过使用MPEG-CENC（MPEG通用加密）结合加密媒体扩展（EME），视频流内容可以仅被加密一次。HLS支持AES-128加密，以及苹果自己的DRM，Fairplay。



### 测试流

- HEVC HLS with fMP4: [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_fmp4.m3u8](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_fmp4.m3u8)

- HEVC HLS with TS (not supported by Apple): [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_ts.m3u8](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream_ts.m3u8)

    [https://bitmovin-a.akamaihd.net/content/playhouse-vr/m3u8s/105560.m3u8](https://bitmovin-a.akamaihd.net/content/playhouse-vr/m3u8s/105560.m3u8)

- HEVC MPEG-DASH: [http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream.mpd](https://bitmovin-a.akamaihd.net/content/dataset/multi-codec/hevc/stream.mpd)

- Multi-Codec MPEG-DASH (AVC/H.264, HEVC/H.265, VP9): http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/stream.mpd

    https://bitmovin-a.akamaihd.net/content/playhouse-vr/mpds/105560.mpd

- VP9 MPEG-DASH: http://bitmovin-a.akamaihd.net/content/dataset/multi-codec/stream_vp9.mpd



参考:  

- [HLS，MPEG-DASH - What is ABR?](http://telestreamblog.telestream.net/2017/05/what-is-abr/)
- [B站我们为什么使用DASH](https://www.bilibili.com/read/cv855111)
- [Adaptive HTTP Streaming Technologies: HLS vs. DASH](https://strivecast.io/hls-vs-mpeg-dash/)
- [自适应流媒体传输](https://blog.csdn.net/nonmarking/article/details/86351147)





在线测试播放器: 

http://demo.theoplayer.com/test-your-stream-with-statistics





