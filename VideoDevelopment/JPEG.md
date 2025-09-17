# JPEG

JPEG（Joint Photographic Experts Group)，全称为联合图像专家小组，是国际标准化组织（ISO）制定的静态图像压缩标准，文件扩展名为.jpg或.jpeg，适用于连续色调静止图像处理。


JPEG只描述一副图像如何转换成一组数据流，而不论这些字节存储在何种介质上。由独立JPEG组创立的另一个进阶标准，JFIF（JPEGFileInterchangeFormat，JPEG文件交换格式）则描述JPEG数据流如何生成适于电脑存储或传送的图像。在一般应用中，我们从数码相机等来源获得的“JPEG文件”，指的就是JFIF文件，有时是ExifJPEG文件。


该格式采用有损压缩算法，通过牺牲部分画质换取较小文件体积，压缩过程包含色彩空间转换（RGB转YCbCr）、离散余弦变换（DCT）、量化和熵编码等步骤。

支持标准JPEG、渐进式JPEG和JPEG2000三种格式，其中渐进式格式可实现图像由模糊到清晰的渐进加载.  

JPEG压缩技术通过分离高频与低频信息并对高频部分进行压缩，压缩比可通过量化表参数调节，典型压缩率为原始大小的10%.   


### 说明
1. jpeg是一种压缩标准，大幅度缩小数据流，jpeg以FF D8开头，FF D9结束。
2. jpeg文件中有一些形如0xFF**这样的数据，它们被称为标志(Markeer),它表示jpeg信息数据段。例如0xFFD8代表SOI(Startof image)。OxFFD9代表EOI(End of image)。
4. jpeg图像由多个maker组成，多个maker+compressed组成了jpeg。
5. jiff是一种在万维网上进行jpeg传输的格式，可以理解是对jpeg图片的封装，符合jpeg标准，jiff的maker是app0，记录了图像的基本信息，也可能有缩略图。jiff格式比较老，老式的数码相机使用此格式。
6. exif新比较新的jpeg封装格式，exif的maker是app1，记录了更多的东西，如ISP信息、GPS信息、相机信息，图像旋转等等
7. jiff和exif可以共存，很多jpeg图像都有app0的jiff段和app1的exif段



## 压缩流程

1. 原始图像
2. 8x8分块
3. DCT(Discrete Cosine Transform,离散余弦变换)变换: 把数据从时域转化到频域的数学方法，把图像数据转换到频域之后，可以从中分离出各种频率的信息。   
4. 量化
5. Z字形扫描
6. 对系数编码
7. 熵编码
8. 压缩数据


 

 
JPEG文件除了图像数据之外，还保存了与图片相关的各种信息，这些信息通过不同类型的TAG存储在文件中。 


## TAG

JPEG通过TAG编辑压缩数据之外的信息。    

所有的TAG都包含一个TAG类型，TAG类型的大小为两个字节，位于一个TAG的最前面。TAG类型的第一个字节一定为0xFF

一般情况下，是按照这个顺序排列的:     


| TAG 类型          | 数值(十六进制标识符)     | 全称                       | 其他备注                          |
|-------------------|----------|----------------------------|-----------------------------------|
| SOI               | 0xFFD8   | Start of Image             | 必带                              |
| APP0              | 0xFFE0   | application0               | 必带                              |
| APPn              | 0xFFEn   | applicationn               | 可选带(APP1一般为Exif信息)        |
| DQT               | 0xFFDB   | Define Quantization Table  | 必带                              |
| SOF               | 0xFFC0   | Start of Frame             | 必带                              |
| DHT               | 0xFFC4   | Define Huffman Table       | 必带                              |
| SOS               | 0xFFDA   | Start of Scan              | 必带                              |
| compress data     | ...      | ...                        | 必带                              |
| EOI               | 0xFFD9   | End of Image               | 必带                              |

标志OxFFE0~OxFFEF被称为"Application Marker"，它们不是解码JPEG文件必须得，可以被用来存储配置信息等。    
EXIF也是利用这个标志段来插入信息的。具体来说，是APP1(0xFFE1)Marker，所有的EXIF信息都存储在该数据段。    
​EXIF（Exchangeable Image File Format）​​ 是嵌入在 JPEG 文件中的 ​元数据标准，记录拍摄设备、参数和场景信息，相当于照片的“数字身份证”。


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/jpeg_1.png?raw=true)             


#### 典型JPEG文件结构    

[SOI] -> [APP0] -> [APP1(EXIF)] -> [DQT] -> [SOF] -> [DHT] -> [SOS] -> [压缩数据] -> [EOI]


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/jpeg_2.png?raw=true)             

##### 怎么判断是不是JPEG图片？ 

jpeg是一种压缩标准，大幅度缩小数据流，jpeg以FF D8开头，FF D9结束。


其实很简单，就是判断前面3个字节是什么，如果发现是FF D8 FF开始，那就认为它是JEPG图片。(注意android不是根据后缀名来判断是什么文件的，当然你必须是图片的后缀名文件管理器才可以打开

















