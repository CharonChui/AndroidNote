Window
===

所有浏览器都支持`window`对象。它表示浏览器窗口。
所有`JavaScript`全局对象、函数以及变量均自动成为`window`对象的成员。
全局变量是`window`对象的属性。
全局函数是`window`对象的方法。
甚至`HTML DOM`的`document也是`window`对象的属性之一：
如
```
window.document.getElementById("header");
```
```
window.innerHeight - 浏览器窗口的内部高度 
window.innerWidth - 浏览器窗口的内部宽度 
window.open() - 打开新窗口 
window.close() - 关闭当前窗口 
window.moveTo() - 移动当前窗口 
window.resizeTo() - 调整当前窗口的尺寸 
```

- Window Screen

`window.screen`对象在编写时可以不使用`window`这个前缀。
一些属性：
```
screen.availWidth - 可用的屏幕宽度 
screen.availHeight - 可用的屏幕高度 
```

- Window Location
`window.location`对象在编写时可不使用`window`这个前缀。
```
location.hostname 返回 web 主机的域名 
location.pathname 返回当前页面的路径和文件名 
location.port 返回 web 主机的端口 （80 或 443） 
location.protocol 返回所使用的 web 协议（http:// 或 https://） 
location.href 属性返回当前页面的 URL。
location.pathname 属性返回 URL 的路径名。
location.assign() 方法加载新的文档。
```

-Window History
`window.history`对象在编写时可不使用`window`这个前缀。
```
history.back() - 与在浏览器点击后退按钮相同 
history.forward() - 与在浏览器中点击按钮向前相同 
```

-Window Navigator
`window.navigator`对象在编写时可不使用`window`这个前缀。
代表正在使用的浏览器对象
```
<Script>
with (document) {
     write ("你的浏览器信息：<OL>");
     write ("<LI>代码："+navigator.appCodeName);
     write ("<LI>名称："+navigator.appName);
     write ("<LI>版本："+navigator.appVersion);
     write ("<LI>语言："+navigator.language);
     write ("<LI>编译平台："+navigator.platform);
     write ("<LI>用户表头："+navigator.userAgent);
}
</Script>
```

- 计时

```
var t=setTimeout("javascript语句",毫秒)
```
`setTimeout()`方法会返回某个值。在上面的语句中，值被储存在名为`t`的变量中。
假如你希望取消这个`setTimeout()`，你可以使用这个变量名来指定它。
```
clearTimeout(setTimeout_variable)// 取消
```
如:    
```
<html>

<head>
<script type="text/javascript">
var c=0
var t

function timedCount()
 {
 document.getElementById('txt').value=c
 c=c+1
 t=setTimeout("timedCount()",1000)
 }

function stopCount()
 {
 clearTimeout(t)
 }
</script>
</head>

<body>
<form>
<input type="button" value="Start count!" onClick="timedCount()">
<input type="text" id="txt">
<input type="button" value="Stop count!" onClick="stopCount()">
</form>
</body>

</html>
```


- 使用`jQuery`框架库
为了引用某个库，请使用`<script>`标签，其`src`属性设置为库的`URL`
```
<!DOCTYPE html>
<html>
<head>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js">
</script>
</head>
<body>
</body>
</html>

```


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 



	