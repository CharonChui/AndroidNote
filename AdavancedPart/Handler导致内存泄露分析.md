Handler导致内存泄露分析
===

有关内存泄露请猛戳[内存泄露][1]

```java
Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
	    // do something.
    }
}
```
当我们这样创建`Handler`的时候`Android Lint`会提示我们这样一个`warning： In Android, Handler classes should be static or leaks might occur.`。       
     
一直以来没有仔细的去分析泄露的原因，先把主要原因列一下：    
- `Android`程序第一次创建的时候，默认会创建一个`Looper`对象，`Looper`去处理`Message Queue`中的每个`Message`,主线程的`Looper`存在整个应用程序的生命周期.
- `Hanlder`在主线程创建时会关联到`Looper`的`Message Queue`,`Message`添加到消息队列中的时候`Message(排队的Message)`会持有当前`Handler`引用，
当`Looper`处理到当前消息的时候，会调用`Handler#handleMessage(Message)`.就是说在`Looper`处理这个`Message`之前，
会有一条链`MessageQueue -> Message -> Handler -> Activity`，由于它的引用导致你的`Activity`被持有引用而无法被回收
- **在java中，no-static的内部类会隐式的持有当前类的一个引用。static的内部类则没有。**

## 具体分析
```java
public class SampleActivity extends Activity {

  private final Handler mHandler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
		  // do something
		}
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
	// 发送一个10分钟后执行的一个消息
	mHandler.postDelayed(new Runnable() {
	  @Override
	  public void run() { }
	}, 600000);

	// 结束当前的Activity
	finish();
}
```
在`finish()`的时候，该`Message`还没有被处理，`Message`持有`Handler`,`Handler`持有`Activity`,这样会导致该`Activity`不会被回收，就发生了内存泄露.

## 解决方法 
- 通过程序逻辑来进行保护。
    - 如果`Handler`中执行的是耗时的操作，在关闭`Activity`的时候停掉你的后台线程。线程停掉了，就相当于切断了`Handler`和外部连接的线，
	`Activity`自然会在合适的时候被回收。 
    - 如果`Handler`是被`delay`的`Message`持有了引用，那么在`Activity`的`onDestroy()`方法要调用`Handler`的`remove*`方法，把消息对象从消息队列移除就行了。 
	    - 关于`Handler.remove*`方法
			- `removeCallbacks(Runnable r)` ——清除r匹配上的Message。    
			- `removeC4allbacks(Runnable r, Object token)` ——清除r匹配且匹配token（Message.obj）的Message，token为空时，只匹配r。
			- `removeCallbacksAndMessages(Object token)` ——清除token匹配上的Message。
			- `removeMessages(int what)` ——按what来匹配     
			- `removeMessages(int what, Object object)` ——按what来匹配      
			我们更多需要的是清除以该`Handler`为`target`的所有`Message(Callback)`就调用如下方法即可`handler.removeCallbacksAndMessages(null)`;
- 将`Handler`声明为静态类。
    静态类不持有外部类的对象，所以你的`Activity`可以随意被回收。但是不持有`Activity`的引用，如何去操作`Activity`中的一些对象？ 这里要用到弱引用
	
```java
public class MyActivity extends Activity {
	private MyHandler mHandler;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		mHandler = new MyHandler(this);
	}

	@Override
	protected void onDestroy() {
		// Remove all Runnable and Message.
		mHandler.removeCallbacksAndMessages(null);
		super.onDestroy();
	}

	static class MyHandler extends Handler {
		// WeakReference to the outer class's instance.
		private WeakReference<MyActivity> mOuter;

		public MyHandler(MyActivity activity) {
			mOuter = new WeakReference<MyActivity>(activity);
		}

		@Override
		public void handleMessage(Message msg) {
			MyActivity outer = mOuter.get();
			if (outer != null) {
				// Do something with outer as your wish.
			}
		}
	}
}
```

[1]:(https://github.com/CharonChui/AndroidNote/blob/master/BasicKnowledge/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F.md)

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 