fMP4格式详解
===

MP4文件的基本单元是“box”，这些box既可以包括data，也可以包括metadata。MP4文件标准允许多种方式来组织data box和metadata box。将metadata放在data之前，客户端应用程序可以在播放video/audio之前获得更多的关于video/audio的信息，因此这种方式在大多数的多媒体应用场景都是比较有用的。但是，在流媒体应用场景，不可能预先保存关于整个流数据的metadata信息，因为不可能提前完全知道。而且，预先保存的metadata越少就意味着越少的开销，因此也可以缩短启动时间。

MP4 ISO Base Media文件格式标准允许以fragmented方式组织box，这也就意味着MP4文件可以组织成这样的结构，由一系列的短的metadata/data box对组成，而不是一个长的metadata/data对。Fragmented MP4文件结构如图1所示，图中只给出了两个fragments。


fmp4 是基于 MPEG-4 Part 12 的**流媒体格式**。与普通MP4相比：

- fmp4不需要一个 moov Box 来进行 initialization
- fmp4 的 moov Box 只包含了一些 track 信息
- fmp4 的 视频/音频 metadata 信息与数据都存在一个个 moof、mdat 中，它是一个流式的封装格式





![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mp4_container_format.webp?raw=true)

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/fmp4_format.png?raw=true)



Fragmented MP4 以 Fragment 的方式存储视频信息。每个 Fragment 由一个 moof box 和一个 mdat box 组成: 

- ‘mdat’（media data box）

    和普通MP4文件的‘mdat’一样，用于存放媒体数据，不同的是普通MP4文件只有一个‘mdat’box，而Fragmented MP4文件中，每个fragment都会有一个‘mdat’类型的box。

- ‘moof’（movie fragment box）

    该类型的box存放的是fragment-level的metadata信息，用于描述所在的fragment。该类型的box在普通的MP4文件中是不存在的，而在Fragmented MP4文件中，每个fragment都会有一个‘moof’类型的box。moof和moov非常像，它包含了当前片段中mp4的相关元信息。

一个‘moof’和一个‘mdat’组成Fragmented MP4文件的一个fragment，这个fragment包含一个video track或audio track，并且包含足够的metadata以保证这部分数据可以单独解码。Fragmented MP4 中的 moov box 只存储文件级别的媒体信息，因此 moov box 的体积比传统的 MP4 中的 moov box 体积要小很多。



![fmp4_parser](https://raw.githubusercontent.com/CharonChui/Pictures/master/fmp4_parser.png?raw=true)



### FMP4与普通MP4 BOX的区别

#### Movie Extends Box (mvex)(fMP4专有)

**mvex 是 fMP4 的标准盒子。它的作用是告诉解码器这是一个fMP4的文件，具体的 samples 信息内容不再放到 trak 里面，而是在每一个 moof 中**。基本格式为：



### moof

moof 主要是用来存放 FMP4 的相关内容。它本身没啥太多的内容。



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 