SDK Manager无法更新的问题
===

由于伟大的防火墙，大陆访问`Google`服务会无法连接。不过作为程序猿，一般都会科学上网，所以这都不是事。今天这里说明一下普通情况下`SDK Manager`无法更新的问题.


- 在更新的时候使用`Http`协议而不是`Https`协议，因为`Https`进行了加密处理，大陆无法审查，所以禁止了，而`Http`协议在过滤的时候发现不是一些禁止内容就没问题了。
    在`SDK Manager`下`Tools->Options`,选中`Force https://… sources to be fetched using http://…`，强制使用`Http`协议.
- 修改`hosts`
   `Windows`在`C:\WINDOWS\system32\drivers\etc`目录下，`Linux`用户打开`/etc/hosts`文件打开后添加以下内容：
   ```
	203.208.46.146    www.google.com 
	74.125.113.121    developer.android.com 
	203.208.46.146    dl.google.com 
	203.208.46.146    dl-ssl.google.com
   ```
   


----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
 