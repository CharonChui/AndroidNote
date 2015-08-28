DOM
===

`DOM`（Document Object Model）文档对象模型
当网页被加载时，浏览器会创建页面的文档对象模型。

通过对象模型,`JavaScript`获得了足够的能力来创建动态的`HTML`。
- `JavaScript`能够改变页面中的所有`HTML`元素 
- `JavaScript`能够改变页面中的所有`HTML`属性 
- `JavaScript`能够改变页面中的所有`CSS`样式 
- `JavaScript`能够对页面中的所有事件做出反应 

- 查找HTML元素

为了做到这件事情，您必须首先找到该元素。有三种方法来做这件事：

- 通过`id`找到`HTML`元素 
- 通过标签名找到`HTML`元素 
- 通过类名找到`HTML`元素 

- 通过`id查找`HTML`元素
	```
	<!DOCTYPE html>
	<html>
	<body>

	<p id="intro">Hello World!</p>

	<script>
	x=document.getElementById("intro");
	document.write('<p>id="intro" 的段落中的文本是：' + x.innerHTML + '</p>');
	</script>

	</body>
	</html>
	```

- 通过标签名查找`HTML`元素	
	```
	<div id="main">
	<p>The DOM is very useful.</p>
	</div>

	<script>
	var x=document.getElementById("main");
	var y=x.getElementsByTagName("p");
	document.write('id 为 "main" 的 div 中的第一段文本是：' + y[0].innerHTML);
	</script>

	</body>
	</html>
	```

- 向HTML输出内容
`document.write()`

```
<!DOCTYPE html>
<html>
<body>

<script>
document.write(Date());
</script>

</body>
</html>
```

- 修改HTML内容
修改`HTML`内容的最简单的方法时使用`innerHTML`属性。
```
<!DOCTYPE html>
<html>
<body>

<h1 id="header">Old Header</h1>

<script>
var element=document.getElementById("header");
element.innerHTML="New Header";
</script>

</body>
</html>
```

- 改变HTML属性
使用`属性名`来获取`HTML`元素的属性
例如改变`<img>`元素的`src`属性:
```
<!DOCTYPE html>
<html>
<body>

<img id="image" src="smiley.gif">

<script>
document.getElementById("image").src="landscape.jpg";
</script>

</body>
</html>
```

- 改变HTML样式
通过`.style.属性名`来获取元素样式,例如下面的改变文字颜色值:   
```
<p id="p2">Hello World!</p>

<script>
document.getElementById("p2").style.color="blue";
</script>
```

- 响应事件
通过对当前元素设置`onclick`属性.例如下面这个请点击该文本的文字，当我们点击时，文字就会变成谢谢。

```
<!DOCTYPE html>
<html>
<body>

<h1 onclick="this.innerHTML='谢谢!'">请点击该文本</h1>

</body>
</html>
```
或
```
<!DOCTYPE html>
<html>
<head>
<script>
function changetext(id)
{
id.innerHTML="谢谢!";
}
</script>
</head>
<body>
<h1 onclick="changetext(this)">请点击该文本</h1>
</body>
</html>

```

- Button 
```
<button onclick="displayDate()">点击这里</button>
```

或者
```
<!DOCTYPE html>
<html>
<head>
</head>
<body>

<p>点击按钮就可以执行 <em>displayDate()</em> 函数。</p>

<button id="myBtn">点击这里</button>

<script>
document.getElementById("myBtn").onclick=function(){displayDate()};
function displayDate()
{
document.getElementById("demo").innerHTML=Date();
}
</script>

<p id="demo"></p>

</body>
</html> 
```

- 创建和删除节点
```
<!DOCTYPE html>
<html>
<body>

<div id="div1">
<p id="p1">这是一个段落。</p>
<p id="p2">这是另一个段落。</p>
</div>

<script>
var para=document.createElement("p");
var node=document.createTextNode("这是新段落。");
para.appendChild(node);

var element=document.getElementById("div1");
element.appendChild(para);
</script>

</body>
</html>
```

- 删除节点
```
<!DOCTYPE html>
<html>
<body>

<div id="div1">
<p id="p1">这是一个段落。</p>
<p id="p2">这是另一个段落。</p>
</div>

<script>
var parent=document.getElementById("div1");
var child=document.getElementById("p1");
parent.removeChild(child);
</script>

</body>
</html>

```



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 



	