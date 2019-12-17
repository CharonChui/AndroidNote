MP4格式详解
===

MP4是一套用于音频、视频信息的压缩编码标准，由国际标准化组织（ISO）和国际电工委员会（IEC）下属的“动态图像专家组”（Moving Picture Experts Group，即MPEG）制定。MPEG-4格式的主要用途在于网上流、光盘、语音发送（视频电话），以及电视广播，是一种常见的多媒体封装格式。


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

声明该违建为MP4文件的相关信息，必须在第一个，而且只有一个

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

该box包含于文件层，可以有多个，也可以没有。用来存储媒体数据。


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mp4_stract.jpg?raw=true)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 