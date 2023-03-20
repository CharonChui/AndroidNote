MP4格式详解
===

MP4是一套用于音频、视频信息的压缩编码标准，由国际标准化组织（ISO）和国际电工委员会（IEC）下属的“动态图像专家组”（Moving Picture Experts Group，即MPEG）制定。MP4 实际代表的含义是MPEG-4 Part 14。它只是MPEG标准中的14部分。它主要参考ISO/IEC标准来制定的。MP4主要作用是可以实现快进快放，边下载边播放的效果。MPEG-4格式的主要用途在于网上流、光盘、语音发送（视频电话），以及电视广播，是一种常见的多媒体封装格式。

MP4的格式稍微比FLV复杂一些，它是通过嵌的方式来实现整个数据的携带。换句话说，它的每一段内容，都可以变成一个对象，如果需要播放的话，只要得到相应的对象即可。


MP4视频文件封装格式是基于QuickTime容器格式定义的，因此参考QuickTime的格式定义对理解MP4文件格式很有帮助。MP4文件格式是一个十分开放的容器，几乎可以用来描述所有的媒体结构，MP4文件中的媒体描述与媒体数据是分开的，并且媒体数据的组织也很自由，不一定要按照时间顺序排列，甚至媒体数据可以直接引用其他文件。同时，MP4也支持流媒体。MP4目前被广泛用于封装h.264视频和AAC音频，是高清视频的代表。

**1、概述**

MP4文件中的所有数据都装在box（QuickTime中为atom）中，也就是说MP4文件由若干个box组成，每个box有类型和长度，可以将box理解为一个数据对象块。box中可以包含另一个box，这种box称为container  box。一个MP4文件首先会有且只有一个“ftyp”类型的box，作为MP4格式的标志并包含关于文件的一些信息；之后会有且只有一个“moov”类型的box（Movie Box），它是一种container  box，子box包含了媒体的metadata信息；MP4文件的媒体数据包含在“mdat”类型的box（Midia Data  Box）中，该类型的box也是container  box，可以有多个，也可以没有（当媒体数据全部引用其他文件时），媒体数据的结构由metadata进行描述。




MP4文件由若干称为Atom（或称为box）的数据对象组成，每个Atom的起首为四个字节的数据长度（Big Endian）和四个字节的类型标识，数据长度和类型标志都可以扩展，Atom的基本结构是:  
```
[4bytes atom size] [4bytes atom type] [8bytes largesize, if size ==1] [contents of the atom, if any]
```

这四部分分别是:   
- 4bytes atom size:表示整个Atom所占用的大小，包含Header部分，如果Atom很大，超过了unit32的最大值(例如存放具体视频数据的mdata)，size就会被设置为1，并用接下来的LARGESIZE来存放数据大小
- 4bytes atom type:表示这个Atom的类型，主要有ftyp、moov、mdat等
- 8bytes largesize, if size ==1: 使用8bytes unit64来存储该Atom的大小
- contents of the atom, if any: 实际的数据

### MP4文件的Atom结构
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mp4_info.png?raw=true)

这里在mac上使用[MediaParser](https://github.com/ksvc/MediaParser)来查看一个mp4文件的结构(Windows上可以使用Mp4Info): 

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mediaparser_mp4.png?raw=true)


Mp4文件需要有ftyp、moov、mdat，它们都是顶级的Atom文件，不能被其他Atom包含。
- ftyp：指示该MP4文件应用的相关信息，必须在第一个，而且只有一个，并且只能被包含在文件层，而不能被其他box包含。
- moov：该box包含了文件媒体的metadata信息，“moov”是一个container box，具体内容信息由子box诠释。
- mdat：实际媒体数据，我们最终解码播放的音视频数据都在这里面。
moov和mdat这两个Atom顺序不固定，一般来说都是moov在前。


### MP4重要box

#### ftyp(File Type Box)

声明该文件为MP4文件的相关信息，必须在第一个，而且只有一个

#### moov(movie box)

moov在mp4文件中也是有且只有一个，moov中会包含1个mvhd和若干个trak。其中mvhd为header box，一般作为moov的第一个子box出现。

##### mvhd(movie header box)

mvhd定义了整个movie的特性，通常包含媒体无关的信息，例如播放时长、创建时间等，具体的类型有:   

- box size
- box type
- version:box版本，0或1，一般为0
- flags
- creation time
- modification time
- time scale
- duration
- rate:推荐播放速率
- volume
- reserved
- matrix
- pre-defined
- next track id

##### trak(Track Box)


- track表示一些sample的集合，对于媒体数据来说，track表示一个视频或音频序列。
- hint track:一类特殊的track，并不包含媒体数据，而是包含了一些将其他数据track打包成流媒体的指示信息。
- sample:对于非hint track来说，video sample即为一帧视频或者一组连续视频帧，audio sample即为一段连续的压缩音频，他们统称为sample。对于hint track来说，sample定义一个或多个流媒体包的格式。
- chunk:一个track的几个sample组成的单元。


一个mp4文件可以包含一个或多个tracks，它们之间相互独立，各自有各自的时间和空间信息。每个track box都有与之关联的media box。trak box 要求必须有一个trak header box(tkhd)和一个media box(mdia)。

- tkhd(track header box)这里面有一个字段，track_ID，用于唯一的表示当前track。另一个字段，duration，用于记录当前track的播放长度。
- mdia(track media box)包含用于声明当前track信息的所有对象，它定义了track媒体类型以及sample数据，来描述sample信息，一般mdia包含一个mdhd(media header box),一个hdlr(handler reference box)和一个minf(media information box)。


#### mdat(media data box)

该box包含于文件层，可以有多个，也可以没有。用来存储媒体数据。MP4文件的媒体数据存放在这里。mdat中的数据帧依次存放，每个帧的位置、时间、长度都由moov中的信息指定。 mdat Box 基本上占据了视频大小的 95% 以上，得益于 mp4 边下边播的效果，浏览器获取到了部分 mdat box，就可以进行播放。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mp4_stract.jpg?raw=true)



**普通MP4文件播放时，ftyp与moov box需同时加载完成后，并下载部分mdat box的帧数据后，才能开始播放**。 那对于一些长视频，确实存在文件头过大，从而影响第一帧的加载速度问题。 另外，对于不是很规范的文件，例 `mp4视频文件举例`中moov box基本在文件最后的的MP4文件，还有可能存在视频文件基本下载完成后才能播放的问题。





---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
