## AVI

音频视频交错(Audio Video Interleaved，AVI)格式，是一门成熟的老技术，尽管国际学术界公认AVI已经属于被淘汰的技术，但是简单易懂的开发API，还在被广泛使用。AVI符合RIFF(ResourceInterchange File Format)文件规范，使用四字符码(Four-Character Code，FOURCC)表征数据类型。AVI的文件结构分为头部、主体和索引3部分。主体中图像数据和声音数据是交互存放的，从尾部的索引可以索引到想放的位置。AVI本身只提供了这么一个框架，内部的图像数据和声音数据格式可以是任意的编码形式。因为索引放在了文件尾部，所以在播放网络流媒体时已力不从心。例如从网络上下载AVI文件，如果没有下载完成，则很难正常播放出来。


AVI中有两种最基本的数据单元，一个是Chunks，另一个是Lists，结构体如下：

```c++
// Chunks
typedef struct {
    DWORD dwFourcc;
    DWORD dwSize;          // data
    BYTE data[dwSize];     // contains headers or audio/video data
} CHUNK;

// Lists
typedef struct {
    DWORD dwList;
    DWORD dwSize;
    DWORD dwFourCC;
    BYTE data[dwSize - 4];
} LIST;
```

由如上代码可知，Chunks数据块由一个四字符码、4B的data size（指下面的数据大小）及数据组成。Lists由4部分组成，包括4B的四字符码、4B的数据大小（指后面列的两部分数据大小）、4B的list类型及数据组成。与Chunks数据块不同的是，Lists数据内容可以包含字块（Chunks或Lists）。






