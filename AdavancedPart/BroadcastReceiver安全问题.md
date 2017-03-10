BroadcastReceiver安全问题
===

`BroadcastReceiver`设计的初衷是从全局考虑可以方便应用程序和系统、应用程序之间、应用程序内的通信，所以对单个应用程序而言`BroadcastReceiver`是存在安全性问题的(恶意程序脚本不断的去发送你所接收的广播)
- 保证发送的广播要发送给指定的对象      
    当应用程序发送某个广播时系统会将发送的`Intent`与系统中所有注册的`BroadcastReceiver`的`IntentFilter`进行匹配，若匹配成功则执行相应的`onReceive`函数。可以通过类似`sendBroadcast(Intent, String)`的接口在发送广播时指定接收者必须具备的`permission`或通过`Intent.setPackage`设置广播仅对某个程序有效。

- 保证我接收到的广播室指定对象发送过来的    
    当应用程序注册了某个广播时，即便设置了`IntentFilter`还是会接收到来自其他应用程序的广播进行匹配判断。对于动态注册的广播可以通过类似`registerReceiver(BroadcastReceiver, IntentFilter, String, android.os.Handler)`的接口指定发送者必须具备的`permission`，对于静态注册的广播可以通过`android:exported="false"`属性表示接收者对外部应用程序不可用，即不接受来自外部的广播。

`android.support.v4.content.LocalBroadcastManager`工具类，可以实现在自己的进程内进行局部广播发送与注册，使用它比直接通过sendBroadcast(Intent)发送系统全局广播有以下几个好处：
- 因广播数据在本应用范围内传播，你不用担心隐私数据泄露的问题。
- 不用担心别的应用伪造广播，造成安全隐患。
- 相比在系统内发送全局广播，它更高效。

```java
 LocalBroadcastManager mLocalBroadcastManager;  
 BroadcastReceiver mReceiver;  

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	 IntentFilter filter = new IntentFilter();  
	 filter.addAction("test");  

	 mReceiver = new BroadcastReceiver() {  
		@Override  
		public void onReceive(Context context, Intent intent) {  
			if (intent.getAction().equals("test")) {  
				//Do Something
			} 
		}  
	};  
	mLocalBroadcastManager = LocalBroadcastManager.getInstance(this);
	mLocalBroadcastManager.registerReceiver(mReceiver, filter);
}


@Override
protected void onDestroy() {
   mLocalBroadcastManager.unregisterReceiver(mReceiver);
   super.onDestroy();
} 
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 