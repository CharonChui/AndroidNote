# 简介

除了前面将的Apple的HLS，还有Adobe HTTP Dynamic Streaming (HDS)、Microsoft Smooth Streaming (MSS)。他们各家的协议原理大致相同，但是格式又不一样，也无法兼容，所以Moving Picture Expert Group (MPEG) 就把大家叫到了一起，呼吁大家一起来制定一个标准的，然后就有了[MPEG-DASH](https://www.encoding.com/mpeg-dash/),它的主要目标是形成IP网络承载单一格式的流媒体并提供高效与高质量服务的统一方案，解决多制式传输方案(HTTP Live  Streaming, Microsoft Smooth Streaming, HTTP Dynamic  Streaming)并存格局下的存储与服务能力浪费、运营高成本与复杂度、系统间互操作弱等问题。

[DASH(MPEG-DASH)](https://mpeg.chiariglione.org/standards/mpeg-dash/)全称为Dynamic Adaptive Streaming over HTTP.是由MPEG和ISO批准的独立于供应商的国际标准，它是一种基于HTTP的使用TCP传输协议的流媒体传输技术。MPEG-DASH是一种自适应比特率流技术，可根据实时网络状况实现动态自适应下载。和HLS, HDS技术类似， 都是把视频分割成一小段一小段，  通过HTTP协议进行传输，客户端得到之后进行播放；不同的是MPEG-DASH支持MPEG-2 TS、MP4(最新的HLS也支持了MP4)等多种格式,  可以将视频按照多种编码切割, 下载下来的媒体格式既可以是ts文件也可以是mp4文件，MPEG—DASH技术与编解码器无关，可使用H.265，H.264，VP9等任何编解码器进行编码。

安卓平台上的ExoPlayer支持MPEG-DASH。另外，三星、索尼、飞利浦、松下的一些较新型号的智能电视支持MPEG—DASH。Google的Chromecast、YouTube 和Netflix 也已支持MPEG-DASH。

![dash_compare](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_compare.png)

## 思想

![dash_main_idea](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_main_idea.png)

DASH的核心思想是，在服务端，视频被提前编好多种码率，并且被切成固定长度的视频片段，存放到HTTP服务器中，当客户端播放时，通过HTTP请求向服务器请求视频切片，并根据网络状况的变化，请求相应质量的视频切片，从而达到对网络带宽的最大利用，并且保证播放流畅。可以实现不同画质内容无缝切换。所以在 YouTube 切换画质时完全不会黑屏，更不会影响观看。

![image-20200406163508642](https://raw.githubusercontent.com/CharonChui/Pictures/master/a_dash_scenario.png)



DASH的整个流程:

内容生成服务器(编码模块、封装模块) -> 流媒体服务器(MPD、媒体文件、HTTP服务器) <-> DASH客户端(控制引擎、媒体引擎、HTTP接入容器)

![image-20200406164238038](https://raw.githubusercontent.com/CharonChui/Pictures/master/mpd_hierarchical_data.png)

### MPD

DASH采用3GPP AHS中定义的MPD(Media Presentation Description)作为媒体文件的描述文件（manifest），作用类似HLS的m3u8文件。MPD文件以XML格式组织，用于描述segment的信息，比如时间、url、视频分辨率、码率等。   
DASH会通过media presentation description (MPD)将视频内容切片成一个很短的文件片段，每个切片都有多个不同的码率，DASH Client可以根据网络的情况选择一个码率进行播放，支持在不同码率之间无缝切换。
DASH中的重要概念

### Period

MPD文件包含一个或者多个片段(Period)，它表示时间轴上的一段时间，每个片段都有一个起始时间和结束时间，并且包含了一个或者多个适配集合(Adaptation Set)。每个适配集合提供了一个或者多个媒体组件的信息，并包含了多种不同的码率。每个适配集合又是由多个呈现(Representation)组成，每个呈现就是同一个视频的不同特征的版本，如码率、分辨率等特征。由于每个的视频都要被切成固定长度的切片，因此每个呈现包括多个视频切片(Segment)，每个视频切片都有一个URL地址，这样客户端就可以通过这个地址向服务器发送HTTP GET请求获取该片段。同一个Period内，意味着可用的媒体内容及其各个可用码率（Representation）不会发生变更。直播情况下，“可能”需要周期地去服务器更新MPD文件，服务器可能会移除旧的已经过时的Period,或是添加新的Period。新的Period中可能会添加新的可用码率或去掉上一个Period中存在的某些码率(Representation)。

### Adaptation Set

 一个Period由一个或者多个Adaptationset组成。Adaptationset由一组可供切换的不同码率的码流（Representation)组成，这些码流中可能包含一个（ISO profile)或者多个(TS profile)media content components，因为ISO profile的mp4或者fmp4 segment中通常只含有一个视频或者音频内容，而TS profile中的TS segment同时含有视频和音频内容，当同时含有多个media component content时，每个被复用的media content component将被单独描述

### Representation

每个Adaptationset包含了一个或者多个Representations,一个Representation包含一个或者多个media streams，每个media stream对应一个media content component。为了适应不同的网络带宽，dash客户端可能会从一个Representation切换到另外一个Representation，如果不支持某个Representation的编码格式，在切换时可以忽略之。

### Segments

Segments可以包含任何媒体数据，关于容器，官方提供了两种建议: ISO base media file format(比如MP4文件格式)和MPEG-2 Transport Stream。

每个Representation会划分为多个Segment。Segment分为4类，其中，最重要的是：Initialization Segment（每个Representation都包含1个Init Seg），Media Segment（每个Representation的媒体内容包含若干Media Seg）

- Initialization Segment：

  Representation的Segments一般都采用1个Init Segment+多个普通Segment的方式，还有一种形式就是Self Initialize Segment，这种形式没有单独的Init Segment，初始化信息包括在了各个Segment中。Init Segment中包含了解封装需要的全部信息，比如Representation中有哪些音视频流，各自的编码格式及参数。对于 ISO profile来说(容器为MP4)，包含了moov box,H264的sps/pps数据等关键信息存放于此（avCc box）。另外，同一个Adaptation set的多个Representation还可能共享同一个Init Segment，该种情况下，对于ISO profile来说，诸如stsd box，avCc box等重要的box会含有多个entry，每个entry对应一个Representation，第一个entry对应第一个Representation，第二个entry对应第二个Representation，以此类推。

- Subsegment

   Segment可能进一步划分为subsegment，每个subsegment由数个Acess Unit组成，Segment index提供了subsegment相对于Segment的字节范围和presentation time range 。客户端可以先下载Segment index。

### SAP和无缝切换以及SEEK

  SAP：Stream Acess Point，可以简单理解为I帧，每个Segment的第一个帧都是SAP，因此Seek时可直接Seek到某一个Segment的起始位置，利用Init Segment+Seek到的某个Segment的数据，在解封装后可实现完美解码。一般来说，同一个Adaptation set中的多个Representation是Segment Align的（当Adaptation set的属性@segmentAlignment不为false时），因此，当从Representation A切换到Representation B时，如果当前Representation A的第N个Segment已经下载完成，切换时直接下载Representation B的第N+1个Segment即可。


    
    

![dash_compare2.png](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_compare2.png)

码率自适应切换算法

- 基于带宽的码率自适应切换算法
- 基于缓存的码率自适应切换算法

![image-20200406165443536](https://raw.githubusercontent.com/CharonChui/Pictures/master/dash_change_compare.png)

此基于时间缓存的码率自适应算法对网络带宽变化反应敏感，能够有效的提高平均码率，但同时码率切换次数过大，尤其是在网络状况波动很大的情况下，这势必会造成用户体验的下降。

- MSS拥有最高的平均码率和较少的切换次数

- HLS的切换次数最少，但是以最低的平均码率作为代价

- HDS不能保证流程的播放

- DASH有足够的竞争力，也具有巨大的提升空间。



## DASH地址

http://ftp.itec.aau.at/datasets/mmsys12/BigBuckBunny/MPDs/BigBuckBunnyNonSeg_2s_isoffmain_DIS_23009_1_v_2_1c2_2011_08_30.mpd

http://www-itec.uni-klu.ac.at/ftp/datasets/mmsys12/BigBuckBunny/bunny_2s/bunny_2s_50kbit/bunny_50kbit_dashNonSeg.mp4



## fMP4

fMP4（fragmented MP4），可以简单理解为分片化的MP4，是DASH采用的媒体文件格式，文件扩展名通常为（.m4s或直接用.mp4）。     

![fMP4](https://img-blog.csdn.net/20171107114807709?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveXVlX2h1YW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



### fMP4与ts的区别

### 媒体数据与元数据的分离

在mp4格式中，元数据可以和媒体数据很好地分开存储，后者都在mdat box中，而在ts中，诸多es流和header/metadata信息是复用在一起的。
 元数据的分离允许我们在streaming中先读取各个流的元数据，知道他们的媒体内容的属性（比如不同的视频质量、不同的语言等），从而可以更好地在不同的media data之间做自适应切换。
 当然，在更实际的应用场景中，比如在dash协议中会直接把这些元数据信息写在mpd中，player可以只读一个mpd就知道各个媒体数据的属性

### 各个Track独立存储

在fmp4中，不仅媒体数据和metadata相互独立的存储，音视频track的数据也可以分开存储，这里的“分开”已经不仅仅局限于box层面的分开，而是真的可以分开存储于不同的目录。在这种情况下，player只需要读一个记录了它们各自存储位置的manifest，即可去对应的位置download它们的分片，只要做好音视频分片之间的同步工作，就可以正常的播放。
 举个例子，下面是一个dash中常见的mpd：

在这个mpd中我们看到视频的分片都是存储在video/1/这个目录下，而音频分片都存储在audio/und/mp4a/1这个目录下，而player还是可以将它们拼接到一起完成播放。
 相对地，在streaming TS流时，音视频往往是复用在一起的，以HLS这个应用场景为例的话，server端一定还需要提前将TS切片做好，这样就会带来几个问题：

1. 媒体文件存储成本和媒资管理成本增加
    假设server端将video track编码为三个质量级别V1, V2, V3，audio track也被编码为三个质量级别A1, A2,  A3，那么如果利用fmp4格式的特性，我们只需要存储这6份媒体文件，player在播放时再自主组合不同质量级别的音视频track即可。而对于TS，则不得不提前将3x3=9种不同的音视频复用情况对应的媒体文件都存储到server端，平白无故多出三份文件的存储成本。实际中，因为要考虑到大屏端、小屏端、移动端、桌面端、不同语言、不同字幕等各种情况，使用TS而造成的冗余成本将更加可观。同时，存储文件的增加也意味着媒资管理成本的增加。这也是包括Netflix在内的一些公司选择使用fmp4做streaming格式的原因。
2. manifest文件更加复杂
    fmp4格式的特性可以确保每一个单独的媒体分片都是可解密可解码的（当然player需要先从moov  box中读到它们的编解码等信息），这意味着server端甚至根本不需要真的存储一大堆分片，player可以直接利用byte range  request技术从一个大文件中准确地读出一个分片对应的media data，这也使得对应manifest(mpd)文件可以更加简洁，如下：

针对不同语言的音频，都只需要存一个大文件就够了。
相对地，在streaming TS流时，不得不在manifest（m3u8）文件中把成百上千个ts分片文件全都老老实实地记录下来。

所以Dash同样可以支持下面的操作: 

- 音频视频分离，在后台播放时可以只拉取音频
- 支持多音轨，多视频轨，多字幕任意切换

### 服务器的cache效率会降低
 实际的streaming应用场景中，往往需要cdn的支持，经常会被client请求的媒体分片就会存在距离client最近的edge  server上。对于fmp4 streaming的情况，因为需要的文件更少，cache命中率也就更高，举个例子：可能某一个audio  track会和其他各种video track组合，那么就可以将这个audio track放在edge server上，而不用每次都跟origin server去请求。
 相对地，在streaming TS流时，因为每一个音视频组合的都需要以复用文件的形式存储，组合数又非常多，相当于分母大了，edge server就会有很大的几率没有缓存需要的组合而要去向orgin server请求。

### 对Trick-play的支持

所谓Trick-play，就是快进、快退、直接跳到章节起点、慢动作播放这些“花式”播放功能。支持这些功能往往意味着要快速找到播放流中的关键帧，以快进播放为例，如果利用fmp4格式的特点，可以通过只读取每个媒体分片的moof加上mdat的起始（包含了关键帧图像）部分即可，说白了就是通过只显示关键帧的方法达到“快进”的视觉效果。因为fmp4格式中可以保证每一个分片一定是以IDR帧开始的，这就使得上述的方案实现起来非常方便。
 相对地，在streaming TS流时，没有办法保证关键帧一定在什么位置，所以你可能需要解析一大堆TS packets才能找到关键帧的位置。

### 无缝码流切换

无缝码流切换实现的关键在于：当第一个码流播放结束时，也就是发生切换的时间，第二个码流一定要以关键帧开始播放。在streaming  TS流时，因为不能保证每一个TS  chunk一定以关键帧开始，做码流切换时就意味着要同时download两个码流的相应分片，同时解析两个码流，然后找到关键帧对应的位置，才能切换。同时下载、解析两个码流的媒体内容对网络带宽以及设备性能都形成了挑战。而且有意思的是，如果当前网络环境不佳，player想要切换到低码率码流，结果还要在本来就不好的网络环境下同时进行两个码流的下载，可谓是雪上加霜。
 而在fmp4中，除了保证各个分片一定以IDR帧开始外，还能保证不同码流的分片之间在时间线上是对齐的。而且streaming  fmp4流时因为不要求音视频复用存储，也就意味着视频和音频的同步点可以不一样，视频可以全都以GOP边界作为同步点，音频可以都以sync  frame作为同步点，这都使得无缝码流切换更简单。

### 与DRM的集成

所谓DRM即数字版权管理，说白了就是对流进行加密，这东西在国内用的不多，但是在国外可是每一个内容提供商必须要有的东西。和编码标准一样，业界也存在很多DRM方案，为了避免每采用一个新的加密方案就要重新编一个码流，MPEG推出了通用加密（CENC）标准（23001-7 - Common Encryption）。使用这一标准的码流，就可以将一个码流应用于各种不同的DRM方案。在DASH  spec中，也定义了Content Protection字段来对应这种加密方案。
 CENC使用的就是fMP4格式，这是利用了fMP4中音视频可以不复用同时还能提供独立于media data存储的metadata的特点。TS流就享受不了这样的好处了。

DASH和HLS之间的另一个关键区别是它支持DRM。可是，在DASH中不存在一个单一通用的DRM解决方案。例如，Google的Chrome支持Widevine，而Microsoft的Internet  Explorer支持PlayReady。然而，通过使用MPEG-CENC（MPEG通用加密）结合加密媒体扩展（EME），视频流内容可以仅被加密一次。HLS支持AES-128加密，以及苹果自己的DRM，Fairplay。






