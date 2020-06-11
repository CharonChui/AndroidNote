RTMP
===



Real Time Messaging Protocol(实时消息传送协议):是Adobe Systems公司为Flash播放器和服务器之间音频、视频和数据传输开发的开放协议。协议基于`TCP`，是一个协议族(默认端口1935)，包括RTMP基本协议及RTMPT/RTMPS/RTMPE等多种变种。RTMP 是一种设计用来进行实时数据通信的网络协议，主要用来在Flash/AIR平台和支持RTMP协议的流媒体/交互服务器之间进行音视频和数据通信。市面上绝大部分PC秀场使用的都是它，他有低延迟(2s左右)、稳定性高、技术完善、高支持度、编码兼容性高等特点。但是RTMP协议不使用标准的HTTP接口传输数据(TCP、UDP端口)，所以在一些特殊的网络环境下可能被防火墙屏蔽掉。默认端口号：1935

RTMP的整体流程为:  

视频采集器 -> 支持RTMP的视频编码器 -> 网络传输  -> 流媒体服务器 -> 网络 -> 客户端



#### 2 握手流程：

以下为简单握手的流程

[![Arb6TU.jpg](https://s2.ax1x.com/2019/03/31/Arb6TU.jpg)](https://imgchr.com/i/Arb6TU)

#### 2.1 包格式：

![img](https://upload-images.jianshu.io/upload_images/1720840-23c7fd9d1b1f0fd3.png?imageMogr2/auto-orient/)

#### 2.2 流程说明：

##### Step 1：C0 + C1

`C0 + C1` 一起发送，其中 C0 为 `1` 个字节，固定为 `0x03`，C1 为 `1536` 个字节，所以包总长度为：`1 + 1535 = 1537`。

C0 标记着客户端 RTMP 的版本号，目前用到 RTMP 为第三版，所以为 03。

> In C0, this field identifies the RTMP version requested by the client. In S0, this field identifies the RTMP version selected by the server. **The version defined by this specification is 3**. 0-2 are deprecated values used by earlier proprietary products; 4-31 are reserved for future implementations; 32-255 are not allowed (to allow distinguishing RTMP from text-based protocols, which always start with a printable character).

##### Step 2：S0 + S1 + S2

`S0 + S1 + S2` 一起发送，其中 S0 为 `1` 个字节，固定为 `0x03`，S1 和 S2 都为 `1536` 个字节，所以包总长度为：`1 + 1536 + 1536 = 3073`。

S0 标记着服务端 RTMP 的版本号，目前用到 RTMP 为第三版，所以为 03。

##### Step 3：C2

C2 为 `1536` 个字节，`RTMP Server` 接收到 `C2` 意味着**握手成功结束**。

#### 3 消息格式（Message）

握手成功后当然就是进行消息通讯，在 RTMP 中的消息都是切分块（Chunk）来发消息的，而 Chunk 发送时必须遵循在一个 Chunk 发送完成之后才能开始发送下一个 Chunk，每个 Chunk 中带有 MessageId 来代表属于哪个 Message，接受端也会按照这个  id 来将 Chunk 组装成 Message。

> 为什么 RTMP要将 Message 拆分成不同的 Chunk 呢？
>
> 通过拆分数据量较大的 Message 可以被拆分成较小的  Message，这样就可以避免优先级低的消息持续发送阻塞优先级高的数据，比如在视频的传输过程中，会包括视频帧，音频帧和 RTMP  控制信息，如果持续发送音频数据或者控制数据的话可能就会造成视频帧的阻塞，然后就会造成看视频时最烦人的卡顿现象。同时对于数据量较小的  Message，可以通过对 Chunk Header 的字段来压缩信息，从而减少信息的传输量。

Chunk 的默认大小是 128 字节，在传输过程中，通过一个叫做 Set Chunk Size 的控制信息可以设置 Chunk  数据量的最大值，在发送端和接受端会各自维护一个Chunk Size，可以分别设置这个值来改变自己这一方发送的 Chunk 的最大大小。

大一点的 Chunk 减少了计算每个 Chunk 的时间从而减少了 CPU  的占用率，但是它会占用更多的时间在发送上，尤其是在低带宽的网络情况下，很可能会阻塞后面更重要信息的传输。小一点的 Chunk  可以减少这种阻塞问题，但小的 Chunk 会引入过多额外的信息（Chunk 中的  Header），少量多次的传输也可能会造成发送的间断导致不能充分利用高带宽的优势，因此并不适合在高比特率的流中传输。

在实际发送时应对要发送的数据用不同的 Chunk Size 去尝试，通过抓包分析等手段得出合适的 Chunk 大小，并且在传输过程中可以根据当前的带宽信息和实际信息的大小动态调整 Chunk 的大小，从而尽量提高 CPU 的利用率并减少信息的阻塞机率。

#### 3.1 块格式：

![img](https://s2.ax1x.com/2019/04/27/EKy8Fx.png)

#### 3.1.1 Basic Header：

Basic Header 包含两个字段：

- **chunk stream id**（流通道id）：占用字节不固定，一共有 3 种情况( `3 字节 - type`、`2 字节 - type` 或 `1 字节 - type`）。支持用户自定义 `［3，65599］` 之间的 id，其中0，1，2 由协议保留表示特殊信息。
- **chunk type**（类型）：占用 2 位，固定长度。

Basic Header 的长度可能是 1，2，或 3 个字节，其中 type 的长度是固定的（占 2 位），Basic Header  的长度取决于 id 的大小，在足够存储这两个字段的前提下最好用尽量少的字节从而减少由于引入 Header 增加的数据量，还有 type  决定了后面 Message Header 的格式。

#### 3.1.2 Message Header：

Message Header 有可能包含以下四个字段：

- timestamp（时间戳）：占用 3 个字节，因此它最多能表示到 `16777215=0xFFFFFF=224-1`，当它的值超过这个最大值时，这三个字节都置为1，这样实际的 timestamp 会转存到 Extended Timestamp 字段中，接受端在判断 timestamp 字段 24 个位都为 1  时就会去 Extended timestamp中 解析实际的时间戳。
- message  length（消息数据的长度）：占用3个字节，表示实际发送的消息的数据如音频帧、视频帧等数据的长度，单位是字节。注意这里是 Message  的长度，也就是 Chunk 属于的 Message 的总数据长度，而不是 Chunk 本身 Data 的数据的长度。
- message type id（消息的类型id）：占用 1 个字节，表示实际发送的数据的类型，如 8 代表音频数据、9 代表视频数据。
- message stream id（消息的流id）：占用 4 个字节，表示该 Chunk 所在的流的id

Message Header 的格式和长度取决于 Basic Header 的 type，共有 4 种不同的格式：

##### 当 chunk type = 0 时 ：

Message Header 占用 11 个字节，分别有 timestamp、message length、message type id 和 message stream id。

##### 当 chunk type = 1 时 ：

Message Header 占用 7 个字节，分别有 timestamp、message length 和 message type id。

- 去掉 msg stream id 的 4个字节，表示此 Chunk 和上一次发的 Chunk 所在的流相同，如果在发送端只和对端有一个流链接的时候可以尽量去采取这种格式。
- timestamp 和 type＝0 时不同，存储的是和上一个 Chunk 的时间差，当它的值超过 3  个字节所能表示的最大值时，3 个字节都置为 1，实际的时间戳差值就会转存到 Extended Timestamp  字段中，接受端在判断timestamp 字段 24 个位都为 1 时就会去 Extended timestamp  中解析时机的与上次时间戳的差值。

##### 当 chunk type = 2 时 ：

Message Header 占用 3 个字节，只有 timestamp，和上一个 Chunk 的时间差。

相对于type＝1 格式又去掉了表示消息长度的 3 个字节和表示消息类型 1 个字节，表示此 Chunk 和上一次发送的 Chunk 所在的流、消息的长度和消息的类型都相同。

##### 当 chunk type = 3 时 ：

Message Header 占用 0 个字节

它表示这个 Chunk 的 Message Header 和上一个是完全相同的。

#### 3.1.3 Extended Timestamp（扩展时间戳）：

上面我们提到在 Chunk 中会有时间戳 timestamp 和时间戳差 timestamp delta，并且它们不会同时存在，只有这两者之一大于 3 个字节能表示的最大数值 `0xFFFFFF＝16777215` 时，才会用这个字段来表示真正的时间戳，否则这个字段为 0。扩展时间戳占 4 个字节，能表示的最大数值就是 `0xFFFFFFFF＝4294967295`。

当扩展时间戳启用时，timestamp 字段或者 timestamp delta  要全置为1，表示应该去扩展时间戳字段来提取真正的时间戳或者时间戳差。注意扩展时间戳存储的是完整值，而不是减去时间戳或者时间戳差的值。(注：这里大概是在讲 extended timestamp 存放的扩展时间戳和扩展时间戳差的区别吧)

#### 3.1.4 Chunk Data：

用户层面上真正想要发送的与协议无关的数据，长度在 (0, chunkSize] 之间。

#### 4 优点：

- 延迟低：1. 从采集推流端到流媒体服务器再到播放端是一条数据流，因此在服务器不会有落地文件。2. 基于 TCP 长连接，不需要多次建连

#### 5 缺点：

- 服务器兼容性差：使用非常规 80 端口，需要额外支持。
- 播放端兼容性不好：需要 Flash 支持。



https://blog.csdn.net/qq_37382077/article/details/103386289



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 