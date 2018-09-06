DLNA
===

一、DLNA简介
---

**DLNA**成立于`2003年6月24日`,其前身是`DHWG（Digital Home Working Group 数字家庭工作组）`，由`Sony、Intel、Microsoft`等发起成立、旨在解决个人`PC` ，消费电器，移动设备在内的无线网络和有线网络的互联互通，使得数字媒体和内容服务的无限制的共享和增长成为可能,目前成员公司已达280多家。`DLN`全称为`DIGITAL LIVING NETWORK ALLIANCE`， 直译成中文就是“数字生活网络联盟”,其宗旨是`Enjoy  your music, photos and videos, anywhere anytime`.

更多内容请访问[DLNA官网](http://www.dlna.org)

（通俗的举个例子：我坐在马桶上点我手机里的歌曲，客厅里电脑的音箱立刻播放了我手机里的音乐，下一首快进暂停都可以手机控制，或者你拍了一段高清的视频，用手机那小小的屏幕实在是没有阖家观赏的效果，不怕，我从容的选择电脑播放，在手机里点到视频，家里23寸的显示器立刻流畅的播放刚刚拍下的热腾腾的高清视频。而且在手机也可以浏览电脑里的图片音乐和视频)。

二、DLNA成员
---

这个组织将加入者分为两个层次，最高层次为`promoter`,  其次为`contributor`。`promoter`制定标准和协议，`contributor`可以分享这个组织的资源，也可以提交标准，参与讨论。现在绝大多数的电子制造商都加入了该组织，至少是`contributor`，而且年费还很贵。成员名单可以从http://www.dlna.org/about_us/roster/中可以找到。    
`DLNA`的骨干成员包括以Intel为首的芯片制造商；以HP为首的PC制造商，以Sony，Panasonic，Sharp，Samsung，LG 为首的家电、消费电子制造商；以CISCO，HUWEI，MOTOROLA，ERICSSON为首的电信设备/移动终端/标准商；一家独大的Microsoft软件/操作系统商等等。

值得注意的有几点：  
  
1. DLNA这个东西基本Intel，Microsoft两个领域巨头在推，一个搞芯片，一个搞系统。AMD没出现在2011的promoter名单中；Google来年会不会掺一脚不好说。还有QUALCOMM也参加进来了，这几年的智能手机芯片处理器他家的也比较多，而且他家还有很多专利可以吃。

2. 2011就剩HP一个大PC商了，其他大PC商如Acer,Asus都还不是promoter，他们肯定要抢着加入的。lenovo不仅从promotor名单中消失了，自然也不会是contributor了，和AMD一样。最开始时lenovo是很积极的，在DHWG的时候也是骨干成员，回来中国搞了一个“IGRS闪联”，退出的原因不知道和这个有没有关系。IGRS在很大程度上和DLNA是比较类似的，框架协议和UPnP也是比较像的。

3. Awox和Cablelabs都是做互联多媒体设备的。Broadcom主要是做移动消费电子，有硬件solution，也有产芯片。

4. ACCESS（爱可视）是做软件的。现在软件的需求很大，给第三方提供软件solution是一块很大的蛋糕。cyberlink和arcsoft也在做这方面，已经有些成熟的软件solution了，像EMC，NeuSoft也有在做。

5. 运营商开始加入了，像at&t美国电报电话公司，at&t也挺厉害的，到处搞签约机，像是跟PSP VITA也签了。以后中国移动联通不知道会不会也跑来参加(有点难...)。

6. dts和dolby都是做音视频标准的，他们基本是跑来收钱的，你机器上到他们的专利你就得付钱，跟以后肯定其他人也会跑来收钱。


三、DLNA标准的制定
---

该组织旨在建立一个基于开放的工业标准的互操作平台，并将确立技术设计规则，供企业开发数字家庭有关的产品。其工作目标是根据开放工业标准制定媒体格式，传输和协议互操作性的指南和规范，和其他工业标准化组织进行联络，提供互操作性测试，并进行数字家庭市场计划的制定和实施。

**DLNA并不是创造技术，而是形成一种解决的方案，一种大家可以遵守的规范。**所以DLNA选择的各种技术和协议都是目前所应用很广泛的技术和协议。所以很多家都要参加，希望DLNA采纳自己的协议和标准，以后自己好办事，可以的话顺便吃点专利费。大方向上肯定打不过Intel和Microsoft的，只能跟着他们走，可以提起其他方面的协议和标准。DLNA的标准写在DLNA GUIDELINES里面，就是大家开会一起写出来的，再开会不停修改的一个standard，一个specification。参加DLNA的商家必须按这个标准走。里面内容不太清楚，我现在没有这个GUIDELINES，这个必须是DLNA会员才能拿到，加会员要10000刀。

四、DLNA设备分类
---

1. Home NetWork Device(HND)。这类设备指家庭设备，具有比较大的尺寸及较全面的功能，主要与移动设备区别开来，下属5类设备：

    - Digital Media Server(DMS)。数字媒体服务器，提供媒体获取、记录、存储和输出功能。同时，内容保护功能是对DMS的强制要求。    
    DMS总是包含DMP的功能，并且肯能包含其他智能功能，包括设备/用户服务的管理；丰富的用户界面；媒体管理/收集和分发功能。DMS的例子有PC、数字机顶盒（附带联网，存储功能）和摄像机等等。

	- DMP。数字媒体播放器。能从DMS/M-DMS上查找并获取媒体内容并播放和渲染显示。比如智能电视、家庭影院等

	- DMC。数字媒体控制器，查找DMS的内容并建立DMS与DMR之间的连接并控制媒体的播放。如遥控器。

	- DMR。数字媒体渲染设备。通过其他设备配置后，可以播放从DMS上的内容。**与DMP的区别在于DMR只有接受媒体和播放功能，而没查找有浏览媒体的功能**。比如显示器、音箱等。

	- DMPr。数字媒体打印机，提供打印服务。网络打印机，一体化打印机就属于DMPr。

2. Mobile Handheld Devices(MHD)手持设备。相比家庭设备，手持设备的功能相对简化一些，支持的媒体格式也会不同。

	- M-DMS。与DMS类似，如移动电话，随身音乐播放器等。

	- M-DMP。与DMP类似。比如智能移动电视。

	- M-DMD。移动多媒体下载设备。如随身音乐播放器，车载音乐播放器和智能电子相框等

	- M-DMU。移动多媒体下载设备。如摄像设备和手机等。

	- M-DMC。与DMC类似。P如DA，智能遥控器。 手持设备没有定义M-DMR，因为手持设备会讲究便利性，会附加查找控制功能，要不然就只是普通的移动电视或收音机了。

3. Networked Infrastructure Devices (NID) 联网支持设备。

	- Mobile Network Connectivity Function (M-NCF)。移动网络连接功能设备。提供各种设备接入移动网络的物理介质。 DLNA的希望是全部实现无线化。

	- Interoperability Unit (MIU)媒体交互设备。提供媒体格式的转换以支持各种设备需要。

设备示例：

1. 你下了班回到家，掏出手机拨到家庭模式，然后就在手机上遥控打开了等离子电视和PC，然后把订阅的新闻通过PC下载完成后打到等离子电视上播放。这时手机就是一个DMC/M-DMC，等离子电视是一个DMR，PC就是DMS。然后你手机上收到一张朋友从巴西传来的照片，你看完之后把它同步到PC上存储起来，这样手机现在的身份是M-DMU，然后你把这张图片放到电子相框里面。这个电子相框就是一个M-DMD，相框也有play的能力，所以他又是一个M-DMP。所以说这些设备的功能角色都是不定的，界限也不是那么严格。在DLNA Guidelines v1.0的时候还没有智能手机，后来在v1.5加入了。这个设备分类只是定义了功能，而且功能也会变的。以后还会出其它新设备，像pad,tab,touch各种各样，到时候标准也会变的。

2. 现在目前的一些电视盒子大多都是`DMS`,像腾讯视频Android客户端有DLNA的功能，我们通过该客户端可以投放视频到盒子上面进行播放，对于此时手机就是一个DMC的功能。    
	又或者盒子通过`DLNA`功能共享出了一些视频、图片等，我们开发一个手机客户端，通过该客户端去获取盒子共享的内容，然后进行播放，这时候手机就相当于一个`DMP`的功能，稍后我们会分别实现`DMC`以及`DMP`的功能。
	
五、DLNA架构
---

`DLNA`架构是个互联系统，因此在逻辑上它也类似`OSI（Open System Interconnection，开放系统互连)`七层网络模型。

DLNA架构分为如下图7个层次：

1. NetWorking Connectivity 网络互联方式:包括物理连接的标准，有有线的，比如符合IEEE802.3标准的Ethernet，；有无线的 ，比如符合IEEE802.11a/g标准的WiFi，能做到54Mbps，蓝牙(802.15)等,技术都很成熟。现在OFDM和MIMO(802.11n)已经能做到300Mbps了，早就超过比较普及的100Mbps的Ethernet了，只不过产品还没有普及，以后肯定会用到。

2. NetWorking Stack 网络协议栈：DLNA的互联传输基本上是在IPV4协议簇的基础上的。用TCP或者UDP来传都可以。这一层相当于OSI网络层。

3. Device Discovery&Control 设备发现和控制。 
	这一层是DLNA的基础协议框架。**DLNA用UPnP协议来实现设备的发现和控制**。下面重点看一下UPnP。    
	`UPnP`，英文是`Universal Plug and play`，翻译过来就是通用即插即用。UPnP最开始Apple和Microsoft在搞，后来Apple不做了(这里多一嘴，为什么Apple不做了，因为Apple现在出了个)，Microsoft还在继续做，Intel也加进来做，Sony，Moto等等也有加入。UPnP有个网站[http://www.upnp.org/](http://www.upnp.org/)，我们发现DLNA的网页和UPnP的网页很像，颜色也差不多，就可以知道他们关系很好了。DNLA主要是在推UPnP。  

	微软官方网站对UPnP的解释：通用即插即用 (UPnP) 是一种用于 PC 机和智能设备（或仪器）的常见对等网络连接的体系结构，尤其是在家庭中。UPnP 以 Internet 标准和技术（例如 TCP/IP、HTTP 和 XML）为基础，使这样的设备彼此可自动连接和协同工作，从而使网络（尤其是家庭网络）对更多的人成为可能。
    举个例子。我们在自己的PC（win7）里面打开网络服务的UPnP选项，然后再家庭网络中共享一个装着视频的文件夹，然后买一台SmartTV回来打开就可以找到这台PC的共享文件夹，然后就直接在电视上选文件播放了。
    UPnP的另外一个作用是给家庭网内的devices做自动的网络地址转换NAT(NAT,Network Address Translation)和端口映射(Port Mapping)，因为家庭网络里面没有那么多IP，所有的devices可能都要通过同一个ip出去。转换映射之后，家庭网络内外的devices就可以通过internet自由地相互连接，而不受内网地址不可访问的阻碍。    
    UPnP Device Architecture 1.0中会说明设备是怎样通过UPnP来相互发现和控制，以及传递消息的。

4. Media Management媒体管理。   
	媒体管理包括媒体的识别、管理、分发和记录（保存），UPnP AV Architecture:1 and UPnP Printer Architecture:1这两个属于UPnP的文档会说明如何进行媒体管理。
	UPnP AV Architecture 定义了UPnP AV设备间媒体传送以及和CP间的交互。UPnP AV也定义了两种UPnP AV设备：UPnP AV MediaServer（MS）和UPnP AV MediaRender（MR），以及他们具有的4种服务：

	- Content Directory Service(CDS)：能将可访问的媒体内容列出。

	- Connection Manager Service(CMS)：决定媒体内容可以通过何种方式由UPnP AV Media Server传送至UPnP AV MediaRender。

	- AVTransport Service：控制媒体内容，比如播放、停止、暂停、查找等。

	- Rendering Control Service：控制以何种方式播放内容，比如音量、静音、亮度等。

5. Media Transport 媒体传输：
	这一层用HTTP(HyperText Transfer Protocol)超文本传输协议。就是平时我们上网用的媒体传输协议。HTTP用TCP可靠传输，也有混合UDP方式的HTTP。现在HTTP的最新版本是HTTP1.1。可选协议是RTP。
    举例:我们输入一个网址，回车，给server发一个request，用TCP我们就可以等server给我们消息，说明server收到我们的消息了，否则我们就重发；接着server给我们TCP包，我们收一个就给server回信说我们收到了，要是server收不到回信，他就认为包丢掉了，会再传一个同样的包过来。不停地回信就是会比较慢。

   那如果我们用UDP会怎样？就是说我们不给server回信说我们收到编号是x的包了，server也就不给我们重发丢掉的包了，这样我们就丢包了。

   但是我们传stream的时候，比如视频流，不用存，看完就完了，这种时候就可以用UDP来传。加上局域网里面QoS本来就很高，丢包都是不太可能的。所以UDP肯定会用。局域网多播的时候也用UDP，这个在后面讲。
   媒体的传输方案如下：
   - 从DMS/M-DMS至DMP/M-DMP，即使不立即播放。
   - 从一个DMS到另一个DMS，这时接收方DMS播放接收媒体内容，表现为一个DMP；也可以不立即播放，可能只是存储或者处理。    
          
   传输 模式有三种：
   - 流传输。当DMR/DMP需要实时渲染接收媒体，媒体具时序性。
   - 交互传输。不包含时序的媒体，如图片传输。
   - 后台传输。非实时的媒体传输，比如上传下载等。

6. Media Formats媒体格式。格式Formats在这里等同于编码格式Codec，平时我们说的编码格式比如Mpeg-2，AVC，x264就是视频编码格式；PCM，mp3(MPEG-2 Layer 3)，aac，flac就是音频编码格式。而avi，rmvb，mkv这些是媒体封装格式，包含视频音频可能还有字幕流。比如一个常见的后缀为mkv的文件，它的视频Codec是x264，音频是aac，它的视音频编码属于Mpeg-4 Codec Family。

7. Remote UI 远程用户接口。
	说白了就是遥控器。比如说有个TV，我们说不管是用遥控器还是直接在TV上按按钮，效果是一样的。不过两者按钮的排列布局是不一样的。好了，现在到DLNA了，我想用手机当遥控器可不可以？当然可以，只要获得TV上按钮的功能，传到手机上来，模拟一个遥控器就好了。DLNA现在想用浏览器的方式，TV给你一个XML，手机上就出现遥控器界面了，有点像webQQ，webOS那种，这样在手机上就不需要客户端了，TV功能更新了，手机直接跟TV要新的XML，很方便。	


六、DLNA可以支持的格式
---

Image：JPEG PNG, GIF, TIFF    
Audio：LPCM AAC, AC-3, ATRAC 3plus, MP3, WMA9    
AV： MPEG2 MPEG-1, MPEG-4, AVC, WMV9     

UPNP协议简介
===

**DLNA通讯采用UPNP协议来进行设备的发现和控制**，UPnP英文名称：Universal Plug and Play中文译名：通用即插即用协议。
我们在做DLNA开发的时候都是用现有的upnp开源框架，upnp官网地址是：
http://upnp.org/
相关SDK地址为http://upnp.org/sdcps-and-certification/resources/sdks/

一、UPnP的工作过程
---

1. 寻址(Addressing)。

    地址是整个UPnP系统工作的基础条件，每个设备都应当是DHCP（Dynamic Host Configuration Protocol 动态主机配置协议）的客户。当设备首次与网络建立连接后，利用DHCP服务，使设备得到一个IP地址。这个IP地址可以是DHCP系统指定的，也可以是由设备选择的。当局域网内没有提供DHCP服务时，UPnP设备将按照Auto-IP的协议，从169.254/169.16地址范围获取一个局域网内唯一的IP地址。设备还可以使用friendly name，这就需要域名解析服务（DNS）来转换name和IP。这个过程用到的东西都是现存的，而且是很普及的，市面上买的路由器都会有。

2. 发现(Discovery)。

	发现是 UPnP工作第一步。 当一个 设备被添加到网络后，UPnP的发现协议允许该设备向网络上的Control Points(CPs)通知(advise)自己拥有的服务。同样，当一个CP被添加到网络后， UPnP发现协议允许该CP 搜索网络上可用的设备 。 这两种情况下的组播消息一般是设备和服务的基本信息，如它的类型， 唯一标识符，当前状态参数等等。要注意设备信息和服务信息都是要组播出去的。发现的过程可以用下面Figure 1-1来描述。

	下面详细叙述UPnP发现设备用到的协议：SSDP(Simple Service Discovery Protocol，简单服务发现协议)，说明设备是怎样向网络通知或者撤销自己可以提供的服务；CP是如何搜索设备以及设备是如何回应搜索的。

	SSDP格式套用HTTP1.1的部分消息头字段，但是和HTTP不同，SSDP是采用UDP传输的，而且SSDP没有Message Body，就是说SSDP只有信头而没有信件内容的。

	SSDP第一个要填充的字段是star - line，说明这是个什么类型的消息。

	比如填"NOTIFY * HTTP/1.1/r/n"，就说明这个SSDP消息是个通知消息，一般设备加入网络或者离开网络都要NOTIFY，更新自己的服务后也要NOTIFY一下。别的设备看见这个消息的star - line就知道有设备状态变了，自己就打开这个消息看一下有没有需要更新的。如果填"NOTIFY * HTTP/1.1/r/n"，就要填LOCATION字段，填一个description URL，CP可以通过这个地址来取得设备的详细信息。

	填"M-SEARCH * HTTP/1.1/r/n"就是要搜索了；respone别人的搜索就填"HTTP/1.1 200 OK/r/n"。

    SSDP第二个要填充的字段是目的地址HOST。比如填上"HOST: 239.255.255.250:1900"，就是组播(multicast)搜索，这里239.255.255.250是组播地址，就是说这条消息会给网络里面该组地址的设备发，1900是SSDP协议的端口号。如果HOST地址是特定地址，那这就是单播(unicast)。Respone不填这个字段，他会在ST字段里面填respone address，就是发来搜索信息的设备的地址，Respone消息的话还会发送一个包含自己地址URL的字段，Respone的意思就是跟Searcher说：我好像是你要找的人，我的电话是XXX，详细情况请CALL我。Respone也是UDP单播。
 
3. 描述(Description)

	前面我们说了CP想要一个device更详细的信息，就打给它的URL跟它要。返回来的东西一般是个XML（Extensible Markup Language，是种结构化的数据。和HTML比较像，有tag和data，具体不说了自己去查），描述分为两部分：一个是device description，是device的物理描述，就是说这个device是什么；还有一个是service descriptions，就是device的服务描述了，就是device能干些什么。这些device和device service的描述的格式也是有要求的，开发商也可以自定义，只要符合UPnP Forum的规范。

	这里稍微解释一下设备描述和服务描述。
	首先说设备，比如一个家庭影院，有显示屏，有功放音响，还有蓝光机。那么这个家庭影院home threatre，就是一个根设备(root device)，它下属有Screen，Amplifier,BDplayer这些从设备。home threatre的描述XML中会有一个device list,列出Screen，Amplifier,BDplayer这些设备的基本信息及这些设备描述的URL，以及设备的presentationURL（这类似于web服务器，通过访问presentationURL，本地会加载一个网页，在这个网页上可以操作设备及其它拥有的服务）；还会有一个sevice list,里面列出home threatre可调用的服务基本信息及服务描述URL。

	再来是服务，通过访问服务描述URL，可以取得服务描述XML，里面会详细介绍服务的信息，包括干什么用的，属于哪个设备，有哪些action，需要哪些参数，怎么调用等等。

4. 控制(Control)

   拿到device description和service descriptions以后，那我们怎么去遥控这些设备呢？    
   在设备描述部分，device description还有关于如何控制device的描述，会给出一个Control URL，CP可以向这个URL发送不同的控制信息就可以控制device了，然后device也可以返回一个信息反馈。    
	这种CP和device之间沟通信息按照Simple Object Access Protocol (SOAP)的格式来写。SOAP通过HTTP来传，现在的版本是1.1，叫做SOAP 1.1 UPnP Profile。这个Profile把控制/反馈信息分成三种：UPnP Control Request，UPnP Control Response和UPnP Control Error Response，都比较好理解。SOAP协议是有信内容Body的，和SSDP不一样。消息Body里面就可以写想调用的动作了，叫做Action invocation，可能还要传参数，比如想播放一个视频，要把视频的URL传过去；device收到后要respone，表示能不能执行调用，出错的话会返回一个错误代码。

5. 事件(Eventing)

	在服务进行的整个时间内，只要变量值发生了变化或者模式的状态发生了改变，就产生了一个事件，该事件服务提供者(某设备的某个服务)会把该事件向整个网络进行多播（multicast）。而且，CP也可以事先向事件服务器订阅事件信息，就像RSS订阅一样，保证将该CP感兴趣的事件及时准确地单播传送过来（unicast）。

	下面是一个Unicast eventing 的architecture图，CP是subscriber，服务器是publisher。

	subscriber(通常是个CP)向publisher(通常是个service)发送订阅消息(subscribe)，更新订阅消息(renewal)，退订消息(cancel)。publisher向subscriber推送订阅(event:SIDX)。
	事件的订阅和推送这块用的通信协议是GENA(General Event Notification Architecture) ，通过HTTP/TCP/IP传送。GENA的格式就不细说了，详细请参阅UPnP-arch-DeviceArchitecture-v1.1。下面列出订阅过程供参考：

	- 订阅。subscriber发送订阅消息主要包含事件URL(evenURL)，服务ID号(service identifier),这两个可以在设备服务描述信息中找到，以及寄送地址(delivery URL)。还会包含一个订阅期限(duration)。

	- 成功订阅。publisher收到订阅信息，如果同意订阅的话就会为每个新subscriber 生成一个唯一的subscriber identifier并记录subscriber 的duration和delivery URL。还会记录一个顺序增长event key用来保证事件确实推送到subscriber那里。比如说有个新事件，key是6，然后把这个事件推送给某个subscriber那里，subscriber那里记录的event key是4，现在收到的事件key是6，他就知道他没收到key为5的事件，这样他就向publisher索要漏收的事件，从而保证双方变量值或状态的一致。

	- 首次推送。订阅同意订阅之后还会向subscriber发送一组初始变量或状态值，进行首次同步。

	- 续订。subscriber必须在订阅到期前发送renewal续订。

	- 订阅到期。订阅到期后publisher会把subscriber的信息删除，subscriber又回到订阅前的状态。

	- 退订。subscriber发送cancel信息将会取消订阅。subscriber因非正常退出网络的话，则不会退订直到订阅到期。

	- 订阅操作失败信息。当订阅、续订和退订不能被publisher接收或者出现错误时，publisher会发送一个错误代码。

	再简单说下多播（multicast，或者叫组播，本文中两者等同）和单播。even的组播采用UDP/IP，和SSDP一样，就是端口号变成了7900。下图是几个协议的所处层的位置，可以清楚地看到它们之间的差别。首先关于IP多播，要知道只存在UDP多播，没有TCP多播这回事。为什么呢？多播的重点是提高网络效率，将同一数据包发送给尽可能多的可能未知的计算机。像这种对网内所有设备的频繁消息通知采用多播是为了减小网络负担，SSDP也是一样。

	但是SSDP和multicast这种采用UDP方式的协议存在一个问题，就是可靠性不够。解决的办法就是多次通知，但是一般不会超过三次以免增加网络负担，这样就得不偿失了。像SSDP的话会采用定期广播advertice的方式，使各种各样原因而没收到advertice的CP重新获得advertice，又解决了UDP丢包的问题。

	前面在寻址的时候用到的DHCP用的是UDP广播(broadcast)。当一个新的设备加入网络时，他想要分个IP，但又不知道DHCP服务器的IP地址，所以他就在网内广播，用255.255.255.255地址来通知所有计算机。DHCP服务器收到请求后会为他申请并返回一个IP地址。

6. 表达(Presentation)

	只要得到了设备的URL，就可以取得该设备表达的URL，取得该设备表达的HTML，然后可以将此HTML纳入CP的本地浏览器上。这部分还包括与用户对话的界面，以及与用户进行会话的处理。因此设备表达可以理解成“遥控器”。这部分定义描述界面，规范界面以及传输界面内容。远程界面是供CP用户使用的，CP用户通过远程界面完成设备描述的获取，控制设备，订阅收取设备事件等等。

到此UPnP的工作过程的讲解就结束了。总结一下：

UPnP分为6个步骤：

先是Addressing，设备加入网络，通过DHCP或者Auto-IP获得IP；这部分在闪联IGRS中是没有定义的。

然后是Discovery，采用SSDP协议(UDP)，用multicast/unicast可以完成设备的上线和离线通知和组播搜索设备，设备用unicast(单播,UDP)响应CP的搜索。

往下是Description，通过HTTP协议(TCP)取回来是一个XML文档，包含物理描述和服务描述；

再来是Control，采用SOAP协议(HTTP/TCP)，完成CP和devices之间的交互；

Eventing，采用GENA协议(HTTP/TCP)，完成设备事件消息的订阅和推送，为保证可靠性，故是TCP传输；事件的推送还有multicast (UDP)。

最后是Presentation。UPnP并没有定义Presentation应该有哪些东西。一个HTML嘛，哪样写得好哪样来！

二、UPNP协议常用类
---

下面是讲解UPnP AV的会用到的一些对象术语。
AVTransport   AVT

ConnectionManager  CM

ContentDirectory  CD

MediaRenderer  MR

MediaServer  MS

RenderingControl  RCS

ScheduledRecording  SRS


在UPnP AV Architecture:1 (Document Version: 1.1) 文档最开始的是这样介绍的UPnP AV的：

本文档描述了整体的UPnP AV 架构 。该架构是 UPnP AV 设备和服务范例的基础架构。

该架构定义了 UPnP 控制端与 UPnP AV设备基本交互，并且与特定设备类型，媒体内容格式与传输协议无关。它支持如电视机，录像机和 CD / DVD 播放机 / 自动点唱机，机顶盒，音响系统， MP3 播放器，静止图像照相机，摄像机，电子相框，以及 PC 等各种设备，。该 AV 架构允许设备支持不同格式的多媒体格式（如 MPEG2， MPEG4 和 JPEG 格式， MP3 ， Windows 媒体架构（ WMA ），位图（ BMP ）， NTSC 制式， PAL 制式，ATSC 标准等）和多种类型的传输协议，如 IEC-61883/IEEE-1394 ， HTTP GET ， RTP 协议， HTTP 的 PUT/邮政， TCP / IP 协议等）。以下各节描述了 AV 架构，以及如何各种 UPnP AV 设备和服务协同工作，使各种最终用户的情况。

“与特定设备类型，媒体内容格式与传输协议无关”的内在含意是 UPnP AV Architecture只是提供了某种机制、模型，并没有规定采用何种技术来实现。技术的实现部分在  UPnP Device Architecture中有说明。

UPnP AV Architecture 定义了 UPnP AV 设备间媒体传送以及和 CP 间的交互。 UPnP AV 也定义了两种 UPnP AV 设备： UPnP AV MediaServer （ MS ）和 UPnP AV MediaRender （ MR ），以及他们具有的 4 种服务：

- Content Directory Service(CDS) ：能将可访问的媒体内容列出。

- Connection Manager Service(CMS) ：决定媒体内容可以通过何种方式由 UPnP AV Media Server 传送至 UPnP AV MediaRender 。

- AVTransport Service ：控制媒体内容，比如播放、停止、暂停、查找等。

- Rendering Control Service ：控制以何种方式播放内容，比如音量、静音、亮度等。

UPnP协议概述
---

随着越来越多的设备联入网络，对于共享设备以及共享设备提供的资源和服务的需求也越来越强烈，透明的访问各种联入网络的资源也成为了一种非常复杂的任务。因此，在1999年，Microsoft公司开始大张旗鼓地宣传下一代即插即用技术--通用即插即用（ Universal Plug and Play，简称UPnP）。UPnP实际上是扩展了传统单机的设备和计算机系统的概念，在"零配置"的前提下提供了连网设备之间的发现、接口声明和其他信息的交换等互动操作功能。Microsoft公司称"UPnP将延伸到家庭中的每一个设备，它会成为个人电脑、应用程序、智能设备集成工作所必需的框架、协议和接口标准"。

UPnP是实现智能设备端到端网络连接的结构。它也是一种架构在TCP/IP和HTTP技术之上的，分布式、开放的网络结构，以使得在联网的设备间传递控制和数据。UPnP 技术实现了 控制点、 设备和 服务之间通讯的支持，并且设备和相关服务的也使用XML定义并且公布出来。使用UPnP，设备可以动态加入网络，自动获得一个IP地址，向其他设备公布它的能力或者获知其他设备的存在和服务，所有这些过程都是自动完成的，此后设备能够彼此直接通讯。

UPnP不需要设备驱动程序，因此使用UPnP建立的网络是介质无关的。同时UPnP使用标准的TCP/IP和网络协议，使它能够无缝的融入现有网络。构造UPnP应用程序时可以使用任何语言，并在任何操作系统平台上编译运行。对于设备的描述，使用HTML表单表述设备控制界面。它既允许设备供应商提供基于浏览器的用户界面和编程控制接口，也允许开发人员定制自己的设备界面。

典型应用场景

随着PC成为网络的中心并提供日益丰富的介质和连接服务，在设备与PC相连之后，越来越多的应用将被开发出来。下面的例子只是其中很小的一部分：

智能家庭网络 
许多智能家居环境使用了现存的家庭控制网络，例如X10网络来控制和监控整个家居环境，比如灯光，安防和其他家庭设备。这些网络可以连接PC上，但是除了PC之外，不能被其他的设备存取。使用UPnP设备可以桥接这些网络成为一个网络，并提供用户更多设备存取家庭网络中的设备。在实现时也无须对X10网络中的现有布线和设备进行昂贵的升级，只需要将设备变成UPnP设备并能够与控制点通讯并接受控制点的控制命令。
数字音频文件管理 
可以在PC和其他设备上播放的数字化音频文件在近几年正在成指数级的增长。一个家庭中，可能有几台计算机或者其他设备用于保存这些音频文件。使用UPnP可以使这些分布在不同PC上的音乐库被统一的管理。这些设备能被发现然后被其他控制点（比如个人电脑、UPnP接收器）控制。同时这些控制点也可以控制家庭中的任何一个扬声器。
数字图片库 
许多家庭使用数字相机拍照，或者将已有照片扫描保存，然后将这些照片上载到他们的计算机中保存。在计算机中对其进行分类，或者以幻灯片的形式进行显示。随着照片的增加，照片可能保存在多种设备或者多种介质上，比如光盘、硬盘、Flash卡。使用UPnP技术，图片库可以自己作为一个设备存在，并自动在网络上声明。这使得一个照片库可能临时为多个应用程序使用，例如可以进行幻灯片显示的同时，在电子像框、机顶盒和电视上进行显示。

UPnP的关键术语

1. Auto-IP 
在Ipv4网络中自动选择一个IP地址。你可以访问IETF文档， http://search.ietf.org/internet-drafts/draft-ietf-dhc-ipv4-autoconfig-05.txt。     

2. DHCP 
动态主机控制协议，可以访问 RFC 2131获得更详细的信息。    

3. HTTPMU 
在UDP上实现HTTP协议的多址传送。    

4. HTTPU 
在UDP上实现普通的HTTP传送协议。    

5. SOAP 
简单对象存取协议（Simple Object Access Protocol ），它是一种应用程序之间进行数据通讯的机制。它是一种在HTTP上使用XML发送命令并接收值的远程过程调用。    

6. UPC 
通用产品编码的缩写（Universal Product Code），它由12个数字构成，由统一编码委员会（Uniform Code Council）管理。这个值可由UPnP制造商指定。     

7. 单一设备名（UDN） 
单一设备名（Unique Device Name）基于UUID，每个表示一个设备。在不同的时间，对于同一个设备此值应该是唯一的。    

8. 设备  
设备是指其他服务或者是设备的容器。一个设备可以包含其他的逻辑设备。

9. 设备描述     
设备描述包含一个物理设备上所有设备一系列通用属性，它包括服务，设备结构和设备属性。

10. 设备类型 
设备类型的一般格式为 urn:schemas-upnp-org:device:uuid-device，uuid-device 为UPnP工作委员会定义的标准设备类型。在UPnP设备模版和设备类型之间是一一对应的，设备制造商也可以指定其他的名字，一般格式为 urn:domain-name:device:uuid-device， uuid-device为制造商定义的标准设备类型，domain-name字段为设备制造商注册的域名。    

11. 根设备 
根设备是指处于设备树最顶层的设备。    

12. 控制点 
控制点是一个控制器，它可以检索设备和服务描述，发送动作到服务，查询服务的状态变量和从服务接收事件。允许用户使用或运行一个设备，例如CD播放机，的程序可以认为是控制点。    

13. 动作 
表示客户端发出的完成特定功能的命令。

14. 事件 
事件是指服务的状态变量的一个或多个改变的通知。

15. 事件变量 
事件变量是指在改变一个服务的状态变量时触发事件的变量。任何订阅此变量的事件源的控制点将接收到改变通知。非事件变量与事件通知没有关系。

16. 服务 
服务是一个逻辑功能单位，服务代表动作和使用状态变量的物理设备的部分或所有状态。

17. 服务描述 
服务描述是指设备提供的一系列动作以及和动作相关的状态变量。

18. 服务类型 
服务类型是表示服务的统一资源名。服务类型和UPnP服务模版之间是一一对应的。UPnP任务组定义了几种标准的服务类型。服务类型的一般格式为： urn:schemas-upnp-org:service:serviceType:version。例如，扫描仪的服务类型应该为 urn:schemas-upnp-org:service:scanner:1。 UPnP设备制造商可以指定附加服务，这样的服务一般格式为：urn:domain-name:service:serviceType:version ， domain-name字段为设备制造商注册的域名。

19. 状态变量 
状态变量是用于描述服务状态的数据片断。

UPnP设备工作过程
---

UPnP定义了设备之间、设备和控制点、控制点之间通讯的协议。完整的UPnP由设备寻址、设备发现、设备描述、设备控制、事件通知和基于Html的描述界面几部分构成。
在最高层中仅包含UPnP制造商定义的特定设备信息，紧接着UPnP工作组定义的内容补充制造商信息。从这层往下，定义的消息为UPnP特定的消息。也就是说，这些消息定义为以下几个协议：简单设备发现协议（Simple Service Discovery Protocol ），通用事件通知结构（General Event Notification Architecture）和 简单对象存取协议（Simple Object Access Protocol）。这些消息使用 HTTPU或者 HTTPMU发送。

1. 设备寻址

    UPnP网络的基础就是TCP/IP协议族，UPnP设备能在TCP/IP协议下工作的关键就是正确的设备寻址。一个UPnP设备寻址的一般过程是：首先向 DHCP服务器发送DHCPDISCOVER消息，如果在指定的时间内，设备没有收到DHCPOFFERS回应消息，设备必须使用 Auto-IP完成IP地址的设置。使用Auto-IP时，设备在地址范围169.254/169.16范围中查找空闲的地址。在选中一个地址之后，设备测试此地址是否在使用。如果此地址被占用，则重复查找过程直到找到一个未被占用的地址，此过程的执行需要底层操作系统的支持，地址的选择过程应该是随机的以避免多个设备选择地址时发生多次冲突。为了测试选择的地址是否未被占用，设备必须使用地址分辨协议（ARP）。一个ARP查询请求设置发送者的硬件地址为设备的硬件地址，发送者的IP地址为全0。设备应该侦听ARP查询响应，或者是否存在具有相同IP地址的ARP查询请求。如果发现，设备必须尝试新的地址。
    使用Auto IP的设备必须定时检测DHCP服务器是否存在，这可以通过定时发送DHCPDISCOVER消息实现，如果接收到DHCPOFFERS回应消息，设备必须释放Auto IP分配的地址，此时设备必须取消所有的广告消息并重新发出新的。
    一个设备可以使用UPnP之外的更高层的协议，这些协议将为设备使用友好的名称。在这种情况下，将这些友好的主机名解析为IP地址就很必要了，DNS通常是用来实现此功能的。使用此功能的设备可能要包含一个DNS客户端，而且支持动态的DNS注册，通过注册将它自己的名字加入到地址分布图中。

2. 设备发现

    一旦设备连接到网上并且分配了地址，就要进行发现的操作了。设备发现是UPnP网络实现的第一步。设备发现是由简单发现协议SSDP（Simple Service Discovery Protocol）来定义的。在设备发现操作之后，控制点可以发现感兴趣的设备，并使得控制点获得设备能力的描述，同时控制点也可以向设备发送命令，侦听设备状态的改变，并将设备展示给用户。
    当一个设备加入到网络中，设备发现过程允许设备向网络上的控制点告知它提供的服务。当一个控制点加入到网络中时，设备发现过程允许控制点寻找网络上感兴趣的设备。在这两种情况下，基本的交换信息就是发现消息。发现消息包括设备的一些特定信息或者某项服务的信息，例如它的类型、标识符、和指向XML设备描述文档的指针。
    在一个新设备加入网络时，如果它存在多个嵌入设备，那么它将多目传送一系列发现消息公开它的设备和服务。任何感兴趣的控制点可以在此标准的多目地址上监听新服务可用通知消息。同样，在一个控制点加入网络时，它多目传送发现消息寻找相关设备或服务。所有的设备必须在标准多目传送地址上监听这些消息并且存在匹配的设备或服务时自动响应发现消息。在设备从网络中除去时，它也应该发出一系列声明，表示此设备包含的设备和服务已经失效。下图表示设备和控制点交互的一般过程：
    简单发现协议（SSDP）定义了在网络中发现网络服务，控制点定位网络上相关资源和设备在网络上声明其可用性的方法。它是建立在 HTTPU和 HTTPMU的基础上的，用于控制设备发送声明和离开消息，以及控制点发送的查询消息实现设备发现操作的。简单发现协议使用租用模型减少了本来是必需的系统开销，网络上的每一个控制点拥有网络状态的全部信息并保持着网络较低的通讯量。为了增加租用模型的健壮性，简单发现协议通过定期发送声明消息的办法保证在设备超时决定设备是否可以使用。缺省的声明消息租用时间是30分钟。

3. 设备描述

    UPnP网络结构的第二步是设备描述。在控制点发现了一个设备之后，控制点仍然对设备知之甚少，控制点可能仅仅知道设备或服务的UPnP类型，设备的UUID和设备描述的URL地址。为了让控制点更多的了解设备和它的功能或者与设备交互，控制点必须从发现消息中得到设备描述的URL，通过URL取回设备描述。设备描述的一般过程：

    对于一个设备的UPnP描述一般分成两个部分：描述设备和描述设备提供的服务。UPnP对某一设备的描述以XML形式表示出来，设备描述包括制造商信息，包括模块名称和编号，序列号，制造商名称，制造商网站的URL等等。设备描述也包括所有嵌入设备描述和URL地址集。对于一个物理设备可以包含多个逻辑设备，多个逻辑设备既可以是一个根设备其中嵌入多个设备，也可以是多个根设备的方式实现。设备描述是由设备制造商提供的，采用XML表述，并且遵循UPnP设备模版。此模版是由UPnP工作委员会生成的。
    UPnP服务描述包括一系列命令或者动作，服务响应，动作的参数。服务的描述也包含一系列变量，这些变量描述了服务运行时刻的状态，这包括数据类型、取值范围和事件特性的描述。服务描述也是由设备制造商提供的，采用XML方式表述，遵循UPnP服务模版。

4. 设备控制

    设备控制是UPnP网络的第三步。在接收设备和服务描述之后，控制点可以向这些服务发出动作，同时控制点也可以轮询服务的状态变量值。发出动作实质上是一种远程过程调用，控制点将动作送到设备服务，在动作完成之后，服务返回相应的结果。设备控制的一般过程如下图：
    为了控制一个设备，控制点向设备服务发出一个动作。这一般通过向服务的控制URL地址发送一个适当的控制消息，而服务则做出相应的响应。动作的效果可以通过改变一个描述服务运行状态的变量。在这些状态变量改变时，时间将发送到所有相关的控制点。控制点也会轮询服务的状态变量值以获得状态变量的当前值，与发出一个动作的过程相似，控制点向服务的控制URL发送一个适当的查询消息，而服务则返回相应的变量值。每个服务必须保持状态变量的一致性，以便控制点能够轮询并接收到有意义的值。

5. 设备事件

    设备事件是UPnP网络的第四步。一个服务的UPnP描述包括服务响应的动作列表和运行时模拟服务状态的变量列表。当这些变量改变时，服务就会发布更新，则控制点就会收到设备事件。设备事件发送的一般过程如下图：
    为了订阅事件，订阅者发送一个订阅消息。如果出版者收到此消息，它将以这个订阅的持续时间作为响应。为了保持订阅，订阅者必须在订阅到期之前进行续订。在订阅者不需要出版者发送的事件时，订阅者必须取消这个订阅。出版者通过发送事件消息提醒订阅者状态变量改变。事件消息包含多个状态变量的名字和这些变量的当前值。在订阅者第一次订阅时，需要发送初始化事件消息，这个事件包含所有事件变量的名和值并且允许订阅者出示化服务变量值。为了支持多个控制点，在动作生效之后所有订阅者都将接到通知。事件消息使用HTTP协议传送，事件详细定义在通用事件通知结构（General Event Notification Architecture）协议中。

6. 设备表征    

    设备表征是UPnP设备的最后一步。如果设备有表征的URL，那么控制点就能通过URL得到页面，在浏览器中装载页面，并使得用户根据页面提供的功能控制设备或者浏览设备状态。它具体能完成到什么与设备和表征页面的功能有关。
    设备表征包含在设备描述的presentationURL字段。设备表征可以完全由设备制造商提供，它采用HTML页的形式，使用HTTP进行发布。

设备发现过程简介
---

UPnP协议的设备发现过程使用简单服务发现协议（Simple Service Discovery Protocol），此协议为网络客户提供一种无需任何配置、管理和维护网络上设备服务的机制。此协议采用基于通知和发现路由的多播发现方式实现。协议客户端在保留的多播地址239.255.255.250发现服务，同时每个设备服务也在此地址上监听服务发现请求。如果服务监听到的发现请求与此服务相匹配，此服务会使用单播方式响应。每个服务也可以向多播端口发送通知声明服务存在。
常见的协议请求消息有两种类型，第一种是服务通知，设备和服务使用此类通知消息声明自己存在；第二种是查询请求，协议客户端用此请求查询某种类型的设备和服务。请求消息中包含设备的特定信息或者某项服务的信息，例如设备类型、标识符和指向设备描述文档的URL地址。下图显示这两类通知消息和HTTP协议的关系：

设备发现过程允许控制点使用一个设备类型或标识，或者是服务类型进行查询。这要求标准设备或服务类型，或者设备特定实例的发现和广告消息基于一个独一无二的标识，UPnP设备和服务类型的定义是UPnP论坛工作委员会的责任。从设备获得响应的内容基本上与多址传送的设备广播相同，只是采用单址传送方式。

DLNA开发
---

目前来说在android中用到的UPNP框架基本为cyberlink框架和cling框架。开心视频和快手看片用的是基于cling框架的dlna开发，而腾讯视频和搜狐视频用的就是基于cyberlink的dlna开发。所以我们也采用了cyberlink这个框架。cyberlink框架效率稍微低而且有几个致命的Bug，但是比较稳定

UPNP协议的几个重要服务:     
`AVTransport`：传输服务，提供媒体文件传输，播放控制等功能。

`ContentDirectory`：内容目录，用于提供媒体文件浏览，检索，获取媒体文件信息等功能。

`ConnectionManager`：连接管理，用于提供连接方面的管理，例如获取源/目的双方支持的MIME格式信息。

`RendringControl`：渲染控制，用于播放时的一些渲染控制，如调节音量，调节亮度等。厂商也可自定义服务

cyberlink框架的构建,http://www.cybergarage.org/
但是官网提供的Android开发框架非常不完善，只能实现基本的DMP功能，对于完整框架的使用请使用[CyberLink4Android](https://github.com/CharonChui/CyberLink4Android),该框架针对CyberLink4Java与Android部分进行了整合。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 




