Github个人主页绑定域名
===

`Github`虽然很好，可毕竟是免费的，还是有不少限制的。写到这里，特意去看了下`Github`对免费用户究竟有什么限制。发现除了300M的空间限制（还是所谓软限制），没有其他限制。所以用它来作为博客平台，真是再理想不过了。

创建步骤
---

1. 建立一个博客repository
    建立一个命名为username.github.io的repository, username就是你在Github上的用户名或机构名

2. 增加主页
    clone该repository到本地，增加index.html

3. 提交
    commit并且push该次修改。
	
4. OK
    打开浏览器输入 http://username.github.io 即可。注意提交之后可能需要一小段时间的延迟。


绑定域名
---

1. 在repository根目录新建CNAME文件, 内容为xxx.com(要绑定的域名),然后commit，push
2. 在自己的域名管理页面中,进入域名解析.
![image](https://github.com/CharonChui/AndroidNote/blob/master/Pic/bindhost.jpg?raw=true)    

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 