HTTP FLV
===

在说HTTP-FLV之前，我们有必要对FLV adobe 官方标准有个认识，因为HTTP-FLV协议中封装格式使用的是FLV。FLV文件格式标准是写F4V/FLV fileformat spec v10.1的附录E里面的FLVFile Format。



FLV（Flash  Video）是Adobe公司设计开发的一种流行的流媒体格式，其格式相对简单轻量，不需要很大的媒体头部信息。整个FLV由Header和Body以及其他Tag组成。因此加载速度极快。它是基于HTTP/80传输，可以避免被防火墙拦截的问题，除此之外，它可以通过 HTTP 302 跳转灵活调度/负载均衡，支持使用 HTTPS 加密传输，也能够兼容支持 Android，iOS 的移动端。但是由于它的传输特性，会让流媒体资源缓存在本地客户端，在保密性方面不够好，因为网络流量较大，它也不适合做拉流协议。此外，FLV可以使用Flash Player进行播放，而Flash  Player插件已经安装在绝大部分浏览器上，这使得通过网页播放FLV视频十分容易。FLV封装格式的文件后缀通常为“.flv”。



先看看HTTP-FLV长成什么样子：http://ip:port/live/livestream.flv，协议头是http,另外”.flv”这个尾巴是它最明显的特征。

HttpFlv 就是 http+flv ，将音视频数据封装成FLV格式，然后通过 HTTP 协议传输给客户端。



下的直播平台中大部分的主线路使用的都是HTTP-FLV协议，备线路多为RTMP。小编随便在Safari中打开几个直播平台房间，一抓包就不难发现使用HTTP-FLV协议的身影：熊猫、斗鱼、虎牙、B站。



HTTP-FLV

我们这里说的HTTP-FLV，主要是说的是HTTP-FLV流，而不是基于HTTP的FLV视频文件点播，也不是能够随意SEEK的HTTP FLV伪流。

FLV渐进式下载：通过HTTP协议将FLV下载到播放器中播放，无法直接拉到中间去播放。

HTTP FLV伪流：支持SEEK，可从未下载的部分开始播放。

HTTP-FLV流：拥有和流式协议RTMP一样的特征，长连接，流式数据。

#### 4 优点：

- 服务器兼容性好：基于 HTTP 协议。
- 低延迟：直接传输 FLV 流，而且基于 HTTP 长链接。

#### 5 缺点：

- 播放端兼容性不好：需要 Flash 支持，不支持多音视频流，不便于 Seek。

▣  HTTP-FLV技术实现

HTTP协议中有个content-length字段的约定，即http的body部分的长度。服务器回复http请求时如果有这个字段，客户端就接收这个长度的数据然后认为数据传输完成了，开始播放。

如果服务器回复http请求中没有这个字段，客户端就一直保持长连接接收数据，直到服务器跟客户端的socket断开。

HTTP-FLV流就利用了上述的第二个原理，服务器回复客户端请求的时候不加content-length字段，在回复了http内容之后，进行持续的数据发送，客户端就一直接收数据，以此实现了HTTP-FLV流直播。

数据传输依然是之前讲过的内容，每一个音视频数据都被封装成包含时间戳信息头的数据包，封装格式采用FLV，传输协议采用http。





❸  RTMP和HTTP-FLV的比较：

RTMP和HTTP-FLV延迟上保持一致，在这二者的区别如下：

▲  穿墙：很多防火墙会墙掉RTMP，但是不会墙HTTP，因此HTTPFLV更不容易出问题。

▲  调度：虽然RTMP也能支持302，单实现起来较麻烦。HTTP FLV本身就支持302，更方便CDN进行重定向以便更精准的调度。

▲  容错： HTTP-FLV回源时也可以回多个源，能做到和RTMP一样，支持多级热备。

▲  简单：FLV是最简单的流媒体封装，HTTP是最广泛的协议，这两个组合在一起维护性更高，比RTMP简单。

▲  友好：HTTP-FLV代码量更小，集成SDK也更轻便。使用起来也更简单。

综上，HTTP-FLV协议具有RTMP的延迟优势，又继承了HTTP所有优势，是流媒体直播首选的分发协议。









https://blog.csdn.net/luzubodfgs/article/details/78155117



https://blog.csdn.net/qq_37382077/article/details/103386289



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 