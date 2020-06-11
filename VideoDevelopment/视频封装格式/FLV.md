FLV
===



FLV封装格式是由一个文件头（FLV header）和很多tag组成（FLV body）组成的二进制文件。Tag中包含了音频数据以及视频数据。FLV的结构如下图所示。

![img](https://img-blog.csdn.net/20160118103525777)

tag又可以分成三类:audio,video,script，分别代表音频流，视频流，脚本流，而每个tag又由tag header和tag data组成。



#### FLV整体结构图：

![img](https:////upload-images.jianshu.io/upload_images/9078032-4d1e3f09df181782.png?imageMogr2/auto-orient/strip|imageView2/2/w/843)



#### FLV文件头结构图

   

![img](https:////upload-images.jianshu.io/upload_images/9078032-b0bab07d69f55262.png?imageMogr2/auto-orient/strip|imageView2/2/w/624)



​    FLV文件头由9bytes组成，前3个bytes是文件类型，总是“FLV”，也就是（0x46 0x4C 0x56）。第4btye是版本号，目前一般是0x01。第5byte是流的信息，倒数第一bit是1表示有视频（0x01），倒数第三bit是1表示有音频（0x4），有视频又有音频就是0x01 | 0x04（0x05），其他都应该是0。最后4bytes表示FLV 头的长度，3+1+1+4 = 9。

２、 FLV body结构分析

​    FLV body由若干个tag 组成。每一个tag第一部分是tag header，tag header长度为11bytes，但是每个tag header前面有4bytes记录着上一个tag的长度。

​    tag结构图：

![img](https:////upload-images.jianshu.io/upload_images/9078032-24c834de3b517f60.png?imageMogr2/auto-orient/strip|imageView2/2/w/853)

​    tag header：

​    １）第1个byte为记录着tag的类型，音频（0x8），视频（0x9），脚本（0x12）；

​    ２）第2到4bytes是数据区的长度，也就是tag data的长度；

​    ３）再后面3个bytes是时间戳，单位是毫秒，类型为0x12则时间戳为0，时间戳控制着文件播放的速度，可以根据音视频的帧率类设置；

​    ４）时间戳后面一个byte是扩展时间戳，时间戳不够长的时候用；

​    ５）最后3bytes是streamID，但是总为0，再后面就是数据区了（tag data），也即是h264的裸流；

​    ６）tag header 长度为1+3+3+1+3=11。

​    音频TagData结构分析：

![img](https:////upload-images.jianshu.io/upload_images/9078032-2339809cce2f8ab0.png?imageMogr2/auto-orient/strip|imageView2/2/w/852)

​    音频参数中各字段的值及其意义如下表所示：

![img](https:////upload-images.jianshu.io/upload_images/9078032-7265d2aa76864647.png?imageMogr2/auto-orient/strip|imageView2/2/w/654)

 音频参数对照表

​    视频TagData结构：

![img](https:////upload-images.jianshu.io/upload_images/9078032-78db278c8115b2a8.png?imageMogr2/auto-orient/strip|imageView2/2/w/851)



​    Script TagData结构

​    Script Tag通常被称为Metadata Tag，会放一些关于FLV视频和音频的元数据信息如：duration、width、height等。通常此类型Tag会跟在File Header后面作为第一个Tag出现，而且只有一个。

![img](https:////upload-images.jianshu.io/upload_images/9078032-52b10dcecd85efe9.png?imageMogr2/auto-orient/strip|imageView2/2/w/843)



​    第一个AMF包：

​    第1个字节表示AMF包类型，一般总是0x02，表示字符串。第2-3个字节为UI16类型值，标识字符串的长度，一般总是 0x000A（“onMetaData”长度）。后面字节为具体的字符串，一般  为“onMetaData”（6F,6E,4D,65,74,61,44,61,74,61）。

所以第一个AMF包总共占13字节。

​    第二个AMF包结构图：

![img](https:////upload-images.jianshu.io/upload_images/9078032-023c79ab3f1c9f83.png?imageMogr2/auto-orient/strip|imageView2/2/w/842)

 第二个AMF包结构图

​    第1个字节表示AMF包类型，一般总是0x08，表示数组。第2-5个字节为UI32类型值，表示数组元素的个数，后面即为各数组元素的封装。数组元素为元素名称和值组成的对。“数组元素结构”部分是推测，已经确认适用于duration、width、height等常见元素，但并不确认适用于所有元素。常见的数组元素如下表所示。

![img](https:////upload-images.jianshu.io/upload_images/9078032-ca0f9296f78d19e6?imageMogr2/auto-orient/strip|imageView2/2/w/407)



参考: 

- [flv/video_file_format_spec_v10](https://www.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10.pdf)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 