HttpURLConnection与HttpClient
===

- `Java`的`HttpURLConnection`     
    请求默认带`Gzip`压缩。
- `Apache`的`HttpClient`       
    请求默认不带`Gzip`压缩。
	
一般对于`API`请求返回的数据大多是`Json`类的字符串，`Gzip`压缩可以使数据大小大幅降低。
`Retrofit`及`Volley`框架默认在`Android Gingerbread(API 9)`及以上都是用`HttpURLConnection`，9以下用`HttpClient`。       

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 