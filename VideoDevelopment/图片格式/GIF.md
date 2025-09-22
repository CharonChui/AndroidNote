# GIF

GIF(Graphics Interchange Format，图像文件存储格式)是CompuServe公司开发的。     

它的核心特点是支持动画和透明度，但牺牲了色彩深度。

1987年开发的GIF文件格式版本号是GIF87a，1989年进行了扩充，扩充后的版本号定义为GIF89a。    

 
GIF图像文件以数据块(block)为单位来存储图像的相关信息。      


一个GIF文件由表示图形/图像的数据块、数据子块以及显示图形/图像的控制信息块组成，称为GIF数据流(Data Stream)。      


数据流中的所有控制信息块和数据块都必须在文件头(Header)和文件结束块(Trailer)之间 。
 
GIF文件格式采用了LZW(Lempel-Ziv Walch)压缩算法来存储图像数据，定义了允许用户为图像设置背景的透明(transparency)属性。     

此外，GIF文件格式可在一个文件中存放多幅彩色图形/图像。如果在GIF文件中存放有多幅图，它们可以像演幻灯片那样显示或者像动画那样演示。     


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/gif_1.png?raw=true)             
 

## GIF文件结构

GIF格式的文件结构整体上分为三部分：文件头、GIF数据流、文件结尾。其中，GIF数据流分为全局配置和图像块。


GIF 文件由一系列数据块（Blocks）和扩展块（Extension Blocks）顺序组成。理解这个结构是解析元数据的关键。
​- 文件头 (Header)​​：GIF87a或 GIF89a，标识文件版本： GIF署名（Signature）和版本号（Version）:GIF的前6个字节内容是GIF的署名和版本号。我们可以通过前3个字节判断文件是否为GIF格式，后3个字节判断GIF格式的版本。
​- 逻辑屏幕描述符 (Logical Screen Descriptor)​​：定义画布大小和全局调色板信息。
​- 全局调色板 (Global Color Table)​​：可选，定义所有帧共用的颜色表。
​- 数据块​:      
​    - 图像描述符 (Image Descriptor)​​：定义帧的图像区域和局部调色板信息。
​    - 基于调色板的图像数据​：使用 LZW 压缩后的像素数据。
​- 扩展块 (Extension Blocks)​​:       
​    - 图形控制扩展 (Graphic Control Extension)​​：​最重要的扩展，包含帧的延迟时间（动画速度）、透明色索引和处置方法。
​    - 注释扩展 (Comment Extension)​​：​这是存储纯文本元数据的主要位置！​​
​    - 应用扩展 (Application Extension)​​：用于存储应用程序的特定信息（如 Netscape 的循环次数）。
    - 明文扩展、等。
​- 文件终结器 (Trailer)​​：一个分号 ;，标识文件结束。



![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/gif_2.png?raw=true)             



数据块可分成3类：控制块(Control Block)，图形描绘块(Graphic-RenderingBlock)和专用块(Special Purpose Block)。


- 控制块: 控制块包含有用来控制数据流(Data Stream)或者设置硬件参数的信息，其成员包括:     
    - GIF文件头(Header)
    - 逻辑屏幕描述块(LogicalScreen Descriptor)
    - 图形控制扩展块(GraphicControl Extension)
    - 文件结束块(Trailer)

- 图形描绘块：包含有用来描绘在显示设备上显示图形的信息和数据，其成员包括:      
    - 图像描述块(ImageDescriptor)
    - 无格式文本扩展块(PlainText Extension)
    - 全局调色板、局部调色板、图像压缩数据和图像说明扩充块。

 - 特殊用途数据块；包含有与图像处理无关的信息，其成员包括:    
    - 注释扩展块(CommentExtension)
    - 应用扩展块(ApplicationExtension)
    - 除了在控制块中的逻辑屏幕描述块(Logical Screen Descriptor)和全局彩色表(GlobalColor Table)的作用范围是整个数据流(Data Stream)之外, 所有其他控制块仅跟在它们后面的图形描绘块。



第二部分：元数据与 XMP 在 GIF 中的存储位置​
这是一个非常关键的点，也是 GIF 与 JPEG、PNG 等格式最大的不同。
​1. 元数据存储在哪里？​​
GIF 格式没有为 EXIF、IPTC 或 XMP 这类现代、结构化的元数据设计标准的存储位置。它的元数据能力非常有限。
​主要位置：注释扩展块 (Comment Extension)​​
​块标识符​：0x21 0xFE
​内容​：可以包含任意纯文本信息。这是存储版权信息、作者、描述等简单元数据的传统位置。
​限制​：​只能是纯文本，没有键值对或结构化的标准。不同软件写入的格式完全不同。
​次要位置：应用扩展块 (Application Extension)​​
​块标识符​：0x21 0xFF
​内容​：通常被特定应用程序用于存储私有数据。例如，NETSCAPE2.0应用扩展用于指定动画循环次数。
​理论上，某个软件可以自定义一个应用扩展来存储 XMP 数据，但这绝非标准，其他软件很可能无法识别。


​2. XMP 存储在哪里？​​
​标准的、符合 Adobe XMP 规范的元数据通常不存在于 GIF 文件中。​​
​原因​：GIF 格式诞生时，XMP 规范还不存在。GIF 的结构没有为容纳一大段 XML 数据而设计。
​变通方案​：极少数专业软件（如 Adobe 的部分产品）​可能会将 XMP 数据作为一串很长的 XML 文本字符串写入到注释扩展 (Comment Extension)​​ 中。
​现实情况​：​99.9% 的 GIF 文件不包含 XMP 数据。​​ 如果您的目标是读取 XMP，遇到 GIF 格式的概率极低。















