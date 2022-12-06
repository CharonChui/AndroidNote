FLV
===


FLV封装格式是由一个文件头（FLV header）和很多tag组成的（FLV body）二进制文件。
Tag中包含了音频数据以及视频数据，每个Tag又有一个preTagSize字段，标记着前面一个Tag的大小，FLV的结构如下图所示。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_tag.jpg?raw=true)


tag又可以分成三类:
- Audio Tag：音频流
- Video Tag：视频流
- Script Tag：脚本流，又称Metadata Tag

每个tag又由tag header和tag data组成。一般一个flv文件由一个头部信息，一个script Tag，以及若干个video Tag和audio Tag组成。


#### FLV整体结构图：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_tag.jpg?raw=true)


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_header_tag.png?raw=true)


#### FLV文件头结构图

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_tag_2.png?raw=true)   

FLV文件头由9bytes组成，前3个bytes是文件类型，总是“FLV”，也就是（0x46 0x4C 0x56）。
第4btye是版本号，目前一般是0x01。第5byte是流的信息，倒数第一bit是1表示有视频（0x01），倒数第三bit是1表示有音频（0x4），有视频又有音频就是0x01 | 0x04（0x05），其他都应该是0。最后4bytes表示FLV 头的长度，3+1+1+4 = 9。

２、 FLV body结构分析

FLV body由若干个tag 组成。每一个tag第一部分是tag header，tag header长度为11bytes，但是每个tag header前面有4bytes记录着上一个tag的长度。

tag结构图：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_body_tag.png?raw=true)   

tag header：

１）第1个byte为记录着tag的类型，音频（0x8），视频（0x9），脚本（0x12）；

２）第2到4bytes是数据区的长度，也就是tag data的长度；

３）再后面3个bytes是时间戳，单位是毫秒，类型为0x12则时间戳为0，时间戳控制着文件播放的速度，可以根据音视频的帧率类设置；

４）时间戳后面一个byte是扩展时间戳，时间戳不够长的时候用；

５）最后3bytes是streamID，但是总为0，再后面就是数据区了（tag data），也即是h264的裸流；

６）tag header 长度为1+3+3+1+3=11。

音频TagData结构分析：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_audio_tag.png?raw=true)   

音频参数中各字段的值及其意义如下表所示：


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_audio_tag2.png?raw=true)   

音频参数对照表

视频TagData结构：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_video_tag.png?raw=true)   

Script TagData结构

Script Tag通常被称为Metadata Tag，会放一些关于FLV视频和音频的元数据信息如：duration、width、height等。通常此类型Tag会跟在File Header后面作为第一个Tag出现，而且只有一个。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_script_tag.png?raw=true)   



第一个AMF包：

第1个字节表示AMF包类型，一般总是0x02，表示字符串。第2-3个字节为UI16类型值，标识字符串的长度，一般总是 0x000A（“onMetaData”长度）。后面字节为具体的字符串，一般  为“onMetaData”（6F,6E,4D,65,74,61,44,61,74,61）。

所以第一个AMF包总共占13字节。

第二个AMF包结构图：

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_awf_tag.png?raw=true)   

 第二个AMF包结构图

第1个字节表示AMF包类型，一般总是0x08，表示数组。第2-5个字节为UI32类型值，表示数组元素的个数，后面即为各数组元素的封装。数组元素为元素名称和值组成的对。“数组元素结构”部分是推测，已经确认适用于duration、width、height等常见元素，但并不确认适用于所有元素。常见的数组元素如下表所示。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/flv_amf_tag.jpg?raw=true)   





参考: 

- [flv/video_file_format_spec_v10](https://www.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10.pdf)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 