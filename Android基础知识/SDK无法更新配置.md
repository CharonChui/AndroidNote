SDK无法更新配置
===

1. 在`SDK Manager下Tools->Options`勾选`Force https://… sources to be fetched using http://…`，强制使用`http`协议。             
2. 修改`hosts`文件。`Windows`在`C:\WINDOWS\system32\drivers\etc`目录下，`Linux`用户打开`/etc/hosts`文件。打开文件后添加以下内容。
    ```
	#Google主页 
	203.208.46.146 www.google.com 
	#这行是为了方便打开Android开发官网 现在好像不翻墙也可以打开 
	74.125.113.121 developer.android.com 
	#更新的内容从以下地址下载 
	203.208.46.146 dl.google.com 
	203.208.46.146 dl-ssl.google.com
	```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 