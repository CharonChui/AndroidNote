# HEIC


HEIC（High Efficiency Image Container​ 高效图像容器）是苹果公司基于HEIF（高效图像文件格式）和HEVC（高效视频编码）技术开发的一种图像存储格式。自2017年iOS 11系统更新起，该格式被设定为iPhone 7及后续机型的默认照片存储格式，在相同画质下可将文件体积较JPEG减少约50%。


High Efficiency Image File Format (HEIF, 发音为：heef)，由 Moving Picture Experts Group ( MPEG，即动态图像专家组) 于2013年开发，它基于ISOBMFF标准。HEIF是一个容器的图片格式，它可以包含图片和图片序列（一个文件可以包含不止一个图片）。当前的编码格式有：HEVC 和H.264/MPEG-4 AVC 两种，并且未来可能有新的编码格式加入。



### HEIF和HEIC的关系

- HEIF是规则、HEIC是实例。 
- HEIF定义了如何存储图像、图像序列(动画、连拍)、音频和元数据。  
- 而HEIC文件是遵循HEIF规则，并使用HEVC(H.265)编码来压缩图像数据的具体文件。   
- .HEIC只是一个HEIF文件格式的一种扩展名，言外之意是：HEIF不仅有.HEIC这种扩展名，还有其它的，比如说: .HEIF和.avci，它们都是属于HEIF文件格式。当然，常见的只有.heif和.heic这两种，而.avci 很少见。



HEIF 的强大源于其现代的设计理念，它本质上是一个基于 ISO 基础媒体文件格式（ISOBMFF）的容器，该格式也是 MP4 视频的基础。


​1. 核心结构：'box' (盒子) 体系​

HEIF/HEIC 文件由一系列嵌套的 ​​'box'​​（或称 'atom'） 组成。每个 'box' 都有一个头部（声明类型和大小）和负载（数据）。
​- ftyp​：文件类型盒，标识这是一个 HEIC 文件。
​- meta​：元数据盒，是最重要的盒子，包含了描述图像数据的各种信息。
​- mdat​：媒体数据盒，存储实际的、经过压缩的图像比特流。














