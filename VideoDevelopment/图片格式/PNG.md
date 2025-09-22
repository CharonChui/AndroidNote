# PNG

[PNG](https://www.w3.org/TR/png/#4Concepts.Format): 便携式网络图形（Portable Network Graphics）是一种无损压缩的位图图形格式,被设计为取代古老的 GIF 格式，其核心目标是无损压缩、支持真彩色和完全透明。      


### 文件结构

PNG图像格式文件（或者称为数据流）由一个8字节的PNG文件署名（PNG file signature）域和按照特定结构组织的3个以上的数据块（chunk）组成。   



1. 文件署名域（Magic Number）​


8字节的PNG文件署名域用来识别该文件是不是PNG文件。该域的值是:     

- 十进制数137 80 78 71 13 10 26 10
- 十六进制数 89 50 4e 47 0d 0a 1a 0a



​固定8字节：89 50 4E 47 0D 0A 1A 0A:        
- 89：高位设置，防止被误判为文本文件。
- 50 4E 47：ASCII "PNG"。
- 0D 0A：DOS 风格的换行符，用于换行符转换检测。
- 1A：DOS 的 EOF（文件结束）字符。
- 0A：Unix 风格的换行符。

​任何PNG解析器首先都必须检查这8个字节。​​


PNG定义了两种类型的数据块:    

- 一种是称为关键数据块（critical chunk），这是必需的数据块
- 另一种叫做辅助数据块（ancillary chunks），这是可选的数据块。

2. 数据块（Chunk）结构​

每个数据块都有统一的格式，如下所示：

```
// 数据块结构
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Length (4 bytes)                              |  // 数据字段长度
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Chunk Type (4 bytes ASCII)                    |  // 块类型，如 IHDR
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Chunk Data (Length bytes)                     |  // 数据内容
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| CRC32 (4 bytes)                               |  // 对整个块类型+数据的校验
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```




​- Length​：Data字段的字节数（32 位无符号整数，​大端序）。
​- Chunk Type​：4 个 ASCII 字符，定义了块的用途。​大小写敏感​:      
​    - 大写首字母​：关键块（解析器必须识别）
    - 小写首字母​：辅助块（可忽略）
    例如：IHDR是关键块，tEXt是辅助块。
​- CRC​：循环冗余校验码，用于检测数据在传输过程中是否出错。


3. 关键数据块（Critical Chunks）​​

解析必须按顺序处理这些块：

| 块类型 | 必须 | 说明 | 结构（Data 字段内容） |

| ：--- | :--- | :--- | :--- |

| ​IHDR​ | 是 | 文件头，包含图像基本信息 | Width(4B), Height(4B), BitDepth(1B), ColorType(1B), Compression(1B), Filter(1B), Interlace(1B) |

| ​PLTE​ | 可选 | 调色板，定义索引颜色 | 一系列 RGB 三元组（R, G, B） |

| ​IDAT​ | 是 | 图像数据，可存在多个 | 由 DEFLATE 压缩过的、经过过滤的像素数据流 |

| ​IEND​ | 是 | 图像结束标记 | 无数据 |


- IHDR：文件头数据块IHDR(header chunk):     

它包含有PNG文件中存储的图像数据的基本信息，并要作为第一个数据块出现在PNG数据流中，而且一个PNG数据流中只能有一个文件头数据块。     
文件头数据块由13字节组成，包含图像宽度(4字节)、图像高度(4字节)、图像深度(1字节)、颜色类型(1字节)、压缩方法(1字节)、滤波器方法(1字节)、隔行扫描方法(1字节)

- PLTE: 调色板数据块PLTE(palette chunk)包含有与索引彩色图像(indexed-color image)相关的彩色变换数据，它仅与索引彩色图像有关，而且要放在图像数据块(image data chunk)之前。PLTE数据块是定义图像的调色板信息，PLTE可以包含1~256个调色板信息，每一个调色板信息由3个字节组成。    
- IDAT: 图像数据块IDAT(image data chunk)：它存储实际的数据，在数据流中可包含多个连续顺序的图像数据块。IDAT存放着图像真正的数据信息，因此，如果能够了解IDAT的结构，我们就可以很方便的生成PNG图像。
- IEND: 图像结束数据IEND(image trailer chunk)：它用来标记PNG文件或者数据流已经结束，并且必须要放在文件的尾部。如果我们仔细观察PNG文件，我们会发现，文件的结尾12个字符看起来总应该是这样的：
0000000049454E44AE426082，不难明白，由于数据块结构的定义，IEND数据块的长度总是0（00 00 00 00，除非人为加入信息），数据标识总是IEND（49 45 4E 44），因此，CRC码也总是AE 42 60 82。 

除了表示数据块开始的IHDR必须放在最前面， 表示PNG文件结束的IEND数据块放在最后面之外，其他数据块的存放顺序没有限制。


4. 辅助数据块（Ancillary Chunks）​​

常见的有:      

| 块类型 | 说明 |

| ：--- | :--- |

| ​tEXt​| 文本信息数据块(textual data)，可存储任何文本信息（关键字+值），如软件、作者、版权。​你之前操作的 aigc:{}就可以写在这里。​​ |

| ​zTXt​ | 压缩的文本数据块(comporessed textual data)。 |

| ​iTXt​ | 国际化的 UTF-8 文本，可包含语言标签。例如中文、日文要写入该区域 |

| ​tRNS​ | 透明度信息数据块(transparency)。对于索引色，它是调色板索引的 Alpha 数组；对于灰度/真彩色，它指定单一颜色为透明色。 |

| ​gAMA​ | 图像y数据块(image gamma)指定图像伽马值，用于跨平台显示校正。 |

| ​pHYs​ | 物理像素尺寸数据块(physical pixel dimensions)指定像素的物理尺寸（像素比/DPI）。 |

| ​tIME​ | 图像最后修改时间数据块tIME(image last-modification time) |


4.8.2 Chunk types
Chunk types are four-byte sequences chosen so that they correspond to readable labels when interpreted in the ISO 646.IRV:1991 [ISO646] character set. The first four are termed critical chunks, which shall be understood and correctly interpreted according to the provisions of this specification. These are:

- IHDR: image header, which is the first chunk in a PNG datastream.
- PLTE: palette table associated with indexed PNG images.
- IDAT: image data chunks.
- IEND: image trailer, which is the last chunk in a PNG datastream.

The remaining chunk types are termed ancillary chunk types, which encoders may generate and decoders may interpret.

- Transparency information: tRNS (see 11.3.1 Transparency information).
- Color space information: cHRM, gAMA, iCCP, sBIT, sRGB, cICP, mDCV (see 11.3.2 Color space information).
- Textual information: iTXt, tEXt, zTXt (see 11.3.3 Textual information).
- Miscellaneous information: bKGD, hIST, pHYs, sPLT, eXIf (see 11.3.4 Miscellaneous information).
- Time information: tIME (see 11.3.5 Time stamp information).
- Animation information: acTL, fcTL, fdAT (see 11.3.6 Animation information).

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/png_1.png?raw=true)             


#### Textual information

PNG provides the tEXt, iTXt, and zTXt chunks for storing text strings associated with the image, such as an image description or copyright notice. Keywords are used to indicate what each text string represents. Any number of such text chunks may appear, and more than one with the same keyword is permitted.

11.3.3.1 Keywords and text strings
The following keywords are predefined and should be used where appropriate.

Table 21 Predefined keywords
Keyword value	Description
Title	Short (one line) title or caption for image
Author	Name of image's creator
Description	Description of image (possibly long)
Copyright	Copyright notice
Creation Time	Time of original image creation
Software	Software used to create the image
Disclaimer	Legal disclaimer
Warning	Warning of nature of content
Source	Device used to create the image
Comment	Miscellaneous comment
XML:com.adobe.xmp	Extensible Metadata Platform (XMP) information, formatted as required by the XMP specification [XMP]. The use of iTXt, with Compression Flag set to 0, and both Language Tag and Translated Keyword set to the null string, are recommended for XMP compliance.
Collection	Name of a collection to which the image belongs. An image may belong to one or more collections, each named by a separate text chunk.
Other keywords MAY be defined by any application for private or general interest.

Keywords SHOULD be .

- reasonably self-explanatory, since the aim is to let other human users understand what the chunk contains; and
- chosen to minimize the chance that the same keyword is used for incompatible purposes by different applications.

## 所有块

The constraints on the positioning of the individual chunks are listed in Table 7 and illustrated diagrammatically for static images in Figure 11 and Figure 12, for animated images where the static image forms the first frame in Figure 13 and Figure 14, and for animated images where the static image is not part of the animation in Figure 15 and Figure 16. These lattice diagrams represent the constraints on positioning imposed by this specification. The lines in the diagrams define partial ordering relationships. Chunks higher up shall appear before chunks lower down. Chunks which are horizontally aligned and appear between two other chunk types (higher and lower than the horizontally aligned chunks) may appear in any order between the two higher and lower chunk types to which they are connected. The superscript associated with the chunk type is defined in Table 8. It indicates whether the chunk is mandatory, optional, or may appear more than once. A vertical bar between two chunk types indicates alternatives.

Table 7 Chunk ordering rules
Critical chunks
(shall appear in this order, except PLTE is optional)
Chunk name	Multiple allowed	Ordering constraints
IHDR	No	Shall be first
PLTE	No	Before first IDAT
IDAT	Yes	Multiple IDAT chunks shall be consecutive
IEND	No	Shall be last
Ancillary chunks
(need not appear in this order)
Chunk name	Multiple allowed	Ordering constraints
acTL	No	Before IDAT
cHRM	No	Before PLTE and IDAT
cICP	No	Before PLTE and IDAT
gAMA	No	Before PLTE and IDAT
iCCP	No	Before PLTE and IDAT. If the iCCP chunk is present, the sRGB chunk should not be present.
mDCV	No	Before PLTE and IDAT.
cLLI	No	Before PLTE and IDAT.
sBIT	No	Before PLTE and IDAT
sRGB	No	Before PLTE and IDAT. If the sRGB chunk is present, the iCCP chunk should not be present.
bKGD	No	After PLTE; before IDAT
hIST	No	After PLTE; before IDAT
tRNS	No	After PLTE; before IDAT
eXIf	No	Before IDAT
fcTL	Yes	One may occur before IDAT; all others shall be after IDAT
pHYs	No	Before IDAT
sPLT	Yes	Before IDAT
fdAT	Yes	After IDAT
tIME	No	None
iTXt	Yes	None
tEXt	Yes	None
zTXt	Yes	None


我们发现上面也有eXif块:   
#### eXif(Exchangeable Image File)  

The data segment of the eXIf chunk contains an Exif profile in the format specified in "4.7.2 Interoperability Structure of APP1 in Compressed Data" of [CIPA-DC-008] except that the JPEG APP1 marker, length, and the "Exif ID code" described in 4.7.2(C), i.e., "Exif", NULL, and padding byte, are not included.

The eXIf chunk size is constrained only by the maximum of 231-1 bytes imposed by the PNG specification. Only one eXIf chunk is allowed in a PNG datastream.

The eXIf chunk contains metadata concerning the original image data. If the image has been edited subsequent to creation of the Exif profile, this data might no longer apply to the PNG image data. It is recommended that unless a decoder has independent knowledge of the validity of the Exif data, the data should be considered to be of historical value only. It is beyond the scope of this specification to resolve potential conflicts between data in the eXIf chunk and in other PNG chunks.



PNG格式中的eXIf块​和​zTXt块​虽然都是用于存储元数据，但其设计目的、内部结构和适用场景有本质区别。


PNG 格式中的eXIf块​和zTXt块​ 虽然都是用于存储元数据，但其设计目的、内部结构和适用场景有本质区别。

为了更直观地理解它们的区别，我们可以通过以下流程图来快速判断哪种块更适合你的需求：
​eXIf 块 (Chunk Type: 65 58 49 66 -> ASCII "eXIf")​​

​1. 作用​
唯一目的​：用于在 PNG 图像中嵌入 ​Exif（Exchangeable Image File Format）数据。
​内容​：Exif 数据是一套高度标准化的元数据规范，主要用于记录数码照片的拍摄参数和设备信息。例如:     
​- 相机参数​：光圈、快门速度、ISO、焦距、镜头型号
​- 设备信息​：相机制造商、型号
​- 时间信息​：照片原始拍摄时间（DateTimeOriginal）
​- 位置信息​：GPS 坐标（经度、纬度、高度）
​- 版权信息​：作者、版权说明
​- 缩略图​：嵌入式的小尺寸预览图



### 应该选择哪一个？​​

​要保存相机的拍摄信息（光圈、快门、GPS等）？​​
-> 使用 ​eXIf 块。（例如：用 exiftool -tagsFromFile source.jpg -all:all拷贝）

​要写入自己定义的文字、配置或标识（如 aigc:{"model":"GPT-4"}）？​​
-> 使用 ​zTXt 块。（例如：用 exiftool -zTXt:YourKey="Your Value"写入）

​简单总结：eXIf 用于“相机说什么”，zTXt 用于“你想说什么”。​
