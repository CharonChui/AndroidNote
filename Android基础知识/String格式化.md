String格式化
===

- 指定内容替换
	- Int类型    
		经常会遇到这种类型比如"共为您找到几条视频"，我们需要通过代码获取把条数设置进去      
		在`string.xml`中可以这样写，`<string name="video_num_tip">共为您找到%1$d条视频</string>` 
		```java
		String tip = getResources().getString(R.string.video_num_tip);  
		// 将`%1$d`替换为8； 
		tip = String.format(tip, 8);
		```
		`%1$d`的意思是整个`video_num_tip`中第一个整型的替代。如果有两个需要替换的整型内容，则第二个写为：`%2$d`，以此类推
	
	- String类型      
		比如“俺叫某某，俺来自某某地，俺为俺自己代言”这里有两个地方需要替换        
		`<string name="introduction">俺叫%1$s，俺来自%2$s，俺为俺自己代言</string>`
		```java
		String intro = getResources().getString(R.string.introduction);   
		intro = String.format(intro, "张三","火星");
		```
		
	- 混合类型         
		`<string name="friendly_tip">您已看了%1$d个电影,还差%2$d个即可获得美女%3$s一枚!</string>`    
	    ```java
	     String text = String.format(getResources().getString(R.string.friendly_tip), 2,18,"苍老师");
	    ```

- 颜色改变
    ```java
    TextView tv = (TextView) findViewById(R.id.tv);
    String html =
            "<body><p><strong>强调</strong></p>"
                    + "<em>斜体</em>"
                    + "<p><a href=\"http://www.baidu.com\">超链接</a>百度一下，你就知道</p>"
                    + "图片</p><img src=\""+R.drawable.ic_launcher+"\"/>";

    tv.setText(Html.fromHtml("<font color=\"#ff0000\">红色</font>其它颜色"));
    tv.setText(Html.fromHtml("<h1>标题1</h1>"));
    //这样会发现图片显示不出来，因为牵扯到图片的时候必须要使用另外一个构造参数
    tv.setText(Html.fromHtml(html));
    
    //图片要用到该构造参数
    tv.setText(Html.fromHtml(html, new ImageGetter() {
        @Override
        public Drawable getDrawable(String source) {
            int id = Integer.parseInt(source);
            Drawable d = getResources().getDrawable(id);
            d.setBounds(0, 0, d.getIntrinsicWidth(), d.getIntrinsicHeight());
            return d;
        }
    }, null));
	```
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
