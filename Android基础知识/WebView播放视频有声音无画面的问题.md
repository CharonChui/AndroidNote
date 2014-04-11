WebView播放视频有声音无画面的问题
===

`WebView`来播放网页中的`Flash`视频，安装到`3.0`以上的版本中只有声音没有画面。后来查了一下资料显示，`3.0`中支持硬件加速，要在`application`节点中配置一下，但是`3.0`一下的版本没有这个配置，但又不想搞两个版本。如何解决呢？也很简单，把`target`版本`3.0`以上(一般都是用最新版做`target`)

- 开启硬件加速
	```xml
	<application
		android:hardwareAccelerated="true"
		android:icon="@drawable/icon"
		android:label="@string/app_name" >
	```
- WebView中设置启用插件
	```java
	webview.getSettings().setPluginsEnabled(true);//支持所有版本
	//webview.getSettings().setPluginState(WebSettings.PluginState.ON);//只支持2.2以上的版本, 2.1及以下版本程序执行到这里会报错
	```

- HTML5 Video support      
    `In order to support inline HTML5 video in your application, you need to have hardware acceleration turned on, and set a WebChromeClient. For full screen support, implementations of onShowCustomView(View,WebChromeClient.CustomViewCallback) and onHideCustomView() are required, getVideoLoadingProgressView() is optional.`
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

