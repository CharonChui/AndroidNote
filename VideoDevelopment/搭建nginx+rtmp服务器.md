搭建nginx+rtmp服务器
===

为了更好的进行直播功能的开发，我们需要本地搭建`nginx+rtmp`服务器，下面就是介绍如何在`Mac`上的搭建步骤:  

1. 安装`Homebrew`(有关`Homebrew`的介绍请参考[Homebrew介绍](http://www.cnblogs.com/lzrabbit/p/4032515.html)

    打开终端, 查看是否已经安装了`Homebrew`, 直接终端输入命令`man brew`
    如果`Mac`已经安装了, 会显示一些命令的帮助信息. 此时输入`Q`退出即可, 直接进入第二步. 反之, 如果没有安装,执行命令 
    `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"` 
    如果安装后, 想要卸载 
    `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"`

2. 安装`nginx`

    - 先执行命令`brew tap homebrew/nginx`来`clone nginx`项目到本地
    - 通过命令`brew install nginx-full --with-rtmp-module`执行安装

    此时, `nginx`和`rtmp`模块就安装好了,输入命令`nginx`然后在浏览器里打开`http://localhost:8080`，如果出现欢迎界面就说明大功告成了。

3. 配置`nginx`和`ramp`

    首先通过命令`brew info nginx-full`查看安装目录， 然后找到`nginx.conf`文件所在位置, 例如`/usr/local/etc/nginx/nginx.conf`。 
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/nginx_path.png?raw=true)

    打开`nginx.confg`文件后在文件的最后一行空白处添加如下代码。
    
    ```java
    rtmp {
        server {
            listen 1935;
            application rtmplive {
                live on;
                record off;
            }
        }
    }
    ```
    然后通过命令`[path] -s reload`重启`nginx`，我的是`/usr/local/opt/nginx-full/bin/nginx -s reload`。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/nginx_reload.png?raw=true)

4. 安装`ffmpeg`

    执行命令`brew install ffmpeg`安装`ffmpeg`。  

5. 安装`VLC`播放器

    [VLC](http://www.videolan.org/)下载安装，可用于播放测试`rtmp`协议的流媒体。

6. `Over`.



参考:  

- [https://github.com/arut/nginx-rtmp-module/wiki](https://github.com/arut/nginx-rtmp-module/wiki)

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
