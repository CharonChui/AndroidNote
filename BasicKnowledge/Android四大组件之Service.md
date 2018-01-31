Android四大组件之Service
===

服务的两种开启方式：
---

1. `startService()`:开启服务.      
	开启服务后 服务就会长期的后台运行,即使调用者退出了.服务仍然在后台继续运行.服务和调用者没有什么关系, 调用者是不可以访问服务里面的方法.
2. `bindService()`:绑定服务.           
	服务开启后,生命周期与调用者相关联.调用者挂了,服务也会跟着挂掉.不求同时生,但求同时死.调用者和服务绑定在一起,调用者可以间接的调用到服务里面的方法.

AIDL
---

本地服务:服务代码在本应用中     
远程服务:服务在另外一个应用里面(另外一个进程里面)      
`aidl`: `android interface defination language`
`IPC implementation` : `inter process communication`        

服务混合调用的生命周期      
---

开启服务后再去绑定服务然后再去停止服务，这时服务是无法停止了.必须先解绑服务然后再停止服务，在实际开发中会经常采用这种模式，
开启服务(保证服务长期后台运行) --> 绑定服务(调用服务的方法) --> 解绑服务(服务继续在后台运行) --> 停止服务(服务停止),服务只会被开启一次，
如果已经开启后再去执行开启操作是没有效果的。    

绑定服务调用方法的原理
---

1. 定义一个接口，里面定义一个方法
	```java
	public interface IService {
		public void callMethodInService();//通过该类中提供一个方法，让自定 
	义的类实现这个接口
	}
	```
	
2. 在服务中自定义一个IBinder的实现类，让这个类继承Binder(Binder是IBinder的默认适配器)，由于这个自定义类是私有的，为了其他类中能拿到该类，
	我们要定义一个接口，提供一个方法，让IBinder类去实现该接口，并在相应方法中调用自己要供别人调用的方法。   
	```java
	public class TestService extends Service {
		@Override
		public IBinder onBind(Intent intent) {
			System.out.println("onbind");
			return new MyBinder();
		}
		
		private class MyBinder extends Binder implements IService{
			public void callMethodInService(){
				//实现该方法，去调用服务中的方法
				methodInService();
			}
		}
		//服务中的方法
		public void methodInService(){
			Toast.makeText(this, "我是服务里面的春哥,巴拉布拉!", 0).show();
		}
	}
	```
	
3. 服务的调用类中将`onServiceConnected`方法中的第二个参数强转成接口
	```java
	public class DemoActivity extends Activity {
		private Intent intent;
		private Myconn conn;
		private IService iService;
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			intent = new Intent(this,TestService.class);
			setContentView(R.layout.main);
		}
		public void start(View view) {
			startService(intent);
		}
		public void stop(View view) {
			stopService(intent);
		}
		public void bind(View view) {
			conn = new Myconn();
			//1.绑定服务 传递一个conn对象.这个conn就是一个回调接口
			bindService(intent, conn, Context.BIND_AUTO_CREATE);
		}
		public void unbind(View view) {
			unbindService(conn);
		}
		//调用服务中的方法
		public void call(View view){
			iService.callMethodInService();
		}
		private class Myconn implements ServiceConnection{
			//当服务被成功绑定的时候调用的方法.
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {//第二个参数就是服务中的onBind方法的返回值
				iService = (IService) service;
			}
			@Override
			public void onServiceDisconnected(ComponentName name) {
			}
		}
	} 
	```

远程服务aidl
---
    
上面介绍了绑定服务调用服务中方法的原理，对于远程服务的绑定也是这样，
但是这个远程服务是在另外一个程序中的，在另外一个程序中定 义的这个接口，
在另外一个程序中是拿不到的，就算是我们在自己的应用 中也定义一个一模一样
的接口，但是由于两个程序的报名不同，这两个接口也是不一样的，为了解决这个
问题，谷歌的工程师给提供了aidl，我们将定义的这个接口的`.java`改成 `.aidl`，
然后将这个接口中的`权限修饰符`都**去掉**，在另一个程序中拷贝这个aidl文 
件，然后放到同一个包名中，由于`Android`中通过包名来区分应用程序，这两个 
`aidl`的包名一样，系统会认为两个程序中的接口是同一个，这样就能够在另一
个程序中将参数强转成这个接口,在使用`aidl`文件拷贝到自己的工程之后会自动
生成一个接口类，这个接口类中有 一个内部类`Stub`该类继承了`Binder`并实现了
这个接口，所以我们在自定义 `IBinder的实现类`时只需让自定义的类继承`Stub类`
即可.

1. 远程服务中定义一个接口，改成`aidl`
	```java
	package com.seal.test.service;

	interface IService {
		 void callMethodInService();
	}
	```

2. 远程服务中定义一个`Ibinder的实现类`，让这个实现类继承上面接口的`Stub类`，
并在`onBind`方法中返回这个自定义类对象 
	```java
	public class RemoteService extends Service {
		@Override
		public IBinder onBind(Intent intent) {
			System.out.println("远程服务被绑定");
			return new MyBinder();
		}
		private class MyBinder extends IService.Stub{
			@Override
			public void callMethodInService() {
				methodInService();
			}
		}
		public void methodInService(){
			System.out.println("我是远程服务里面的方法");
		}
	}
	```

3. 在其他程序中想要绑定这个服务并且调用这个服务中的方法的时候首先要拷贝 
这个`aidl`文件到自己的工程，然后再`ServiceConnection`的实现类中将这个参数使 
用`asInterface`方法转成接口，通过这样来得到接口，从而调用接口中的方法     
	```java
	public class CallRemoteActivity extends Activity {
		private Intent service;
		private IService iService;
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.main);
			service = new Intent();
			service.setAction("com.itheima.xxxx");
		}
		public void bind(View veiw){
			bindService(service, new MyConn(), BIND_AUTO_CREATE);
		}
		public void call(View view){
			try {
				iService.callMethodInService();
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
		private class MyConn implements ServiceConnection{
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {
				iService = IService.Stub.asInterface(service);
			}
			@Override
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub
			}
		}
	}
	```

最后说一下`IntentService`:

`IntentService`是`Service`的子类，用来处理异步请求。客户端可以通过`startService(Intent)`方法将请求的`Intent`传递请求给`IntentService`，
`IntentService`会将该`Intent`加入到队列中，然后对每一个`Intent`开启一个`worker thread`来进行处理，执行完所有的工作之后自动停止`Service`。
每一个请求都会在一个单独的`worker thread`中处理，不会阻塞应用程序的主线程。`IntentService` 实际上是`Looper`、`Handler`、`Service` 的集合体,
他不仅有服务的功能,还有处理和循环消息的功能.

- Service：    
    1. A Service is not a separate process. The Service object itself does not imply it is running in its own process; unless otherwise specified, 
		it runs in the same process as the application it is part of.
    2. A Service is not a thread. It is not a means itself to do work off of the main thread (to avoid Application Not Responding errors).
所以在`Service`中进行耗时的操作时必须要新开一个线程。    

至于为什么要使用`Service`而不是`Thread`，这个主要的区别就是生命周期不同，`Service`是Android系统的一个组件，Android系统会尽量保持`Service`的长期后台运行，
即使内存不足杀死了该服务(很少会出现内存不足杀死服务的情况)也会在内存可用的时候去复活该服务，而`Thread`随后都会被杀死
- IntentService
    1. IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. 
		Clients send requests throughstartService(Intent) calls; 
	    the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.    
    2. This "work queue processor" pattern is commonly used to offload tasks from an application's main thread. 
		The IntentService class exists to simplify this pattern and take care of the mechanics. 
	    To use it, extend IntentService and implement onHandleIntent(Intent). IntentService will receive the Intents, launch a worker thread, 
		and stop the service as appropriate.
    3. All requests are handled on a single worker thread -- they may take as long as necessary (and will not block the application's main loop), 
		but only one request will be processed at a time.

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
