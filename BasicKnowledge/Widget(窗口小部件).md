Widget简介
===

可以使用`AppWidgetManager`更新`Widget`中的数据，但这样最短也要半个小时才能更新一次，一般不用他更新，而是自己定义一个服务去更新`Widget`中的数据。

## Widget的创建步骤

1. 写一个类继承AppWidgetProvider,这个是一个广播接收者,所以要在清单文件中进行配置
	```java
	public class MyWidget extends AppWidgetProvider {
		@Override
		public void onEnabled(Context context) {
			//开启服务定期的更新界面.
			Intent intent = new Intent(context,UpdateWidgetService.class);
			context.startService(intent);
			super.onEnabled(context);
		}
		
		@Override
		public void onDisabled(Context context) {
			//关闭掉服务
			Intent intent = new Intent(context,UpdateWidgetService.class);
			context.stopService(intent);
			super.onDisabled(context);
		}
					
		@Override
		public void onUpdate(Context context, AppWidgetManager appWidgetManager,
				int[] appWidgetIds) {      

			//Widget布局中定义的更新时间到了。检查下 服务是否还活着.
			if(!ServiceStatusUtil.isServiceRunning(context, "com.itheima.mobilesafe.service.UpdateWidgetService")){
				Intent intent = new Intent(context,UpdateWidgetService.class);
				context.startService(intent);
			}
			super.onUpdate(context, appWidgetManager, appWidgetIds);
		}
	}
	```
	
2. 在清单文件中进行配置,内容如下:
	```xml
	<receiver android:name=".receiver.MyWidget" >
		<intent-filter>
			<action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
		</intent-filter>
		<meta-data
			android:name="android.appwidget.provider"
			android:resource="@xml/example_appwidget_info" />//这里使用到了一个xml文件,所以要创建这个文件
	</receiver> 
	```
	
3. 在res下面新建一个名为xml的文件件，然后新建example_appwidget_info.xml内容如下
	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
		android:minWidth="294dp"
		android:minHeight="72dp"
		android:updatePeriodMillis="86400000" //指定更新的间隔时间，最小为半个小时，一般不用它更新，都是自己更新
		android:previewImage="@drawable/preview"//指定小控件的图标,如果不要这个选项就是程序的图标
		android:initialLayout="@layout/example_appwidget"//设置这个小控件的布局文件
		android:configure="com.example.android.ExampleAppWidgetConfigure" //有些复杂的Widget在点击的时候会去开启一个Activity，
		// 这时候就是通过这个参数来配置要开启的Activity，用不着就删除这一行	android:resizeMode="horizontal|vertical">
		//这个是Android3.0的一个新特性，是可以让widget改变大小，在2.3时候创建出来的Widget多大就是多大，不能改变，可以把这个去掉
	</appwidget-provider>
	```
	
4. 更新Widget数据的服务      
	```java
	public class UpdateWidgetService extends Service {
		private Timer timer;
		private TimerTask task;
		@Override
		public IBinder onBind(Intent intent) {
			return null;
		}
		@Override
		public void onCreate() {
			super.onCreate();
			// 开启定期的任务更新widget.
			timer = new Timer();
			task = new TimerTask() {
				@Override
				public void run() {
					AppWidgetManager awm = AppWidgetManager
							.getInstance(getApplicationContext());
					ComponentName component = new ComponentName(
							getApplicationContext(), MyWidget.class);
					RemoteViews views = new RemoteViews(getPackageName(),
							R.layout.process_widget);
					views.setTextViewText(
							R.id.process_count,
							"正在运行:"
									+ ProcessStatusUtils.getProcessCount(getApplicationContext())
									+ "个");
					views.setTextViewText(
							R.id.process_memory,
							"可用内存:"
									+ Formatter
											.formatFileSize(
													getApplicationContext(), ProcessStatusUtils.getAvailRAM(getApplicationContext())));
					Intent intent = new Intent();
					intent.setAction("com.itheima.killall");
					//设置一个自定义的广播事件 动作  com.itheima.killall
					PendingIntent pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), 0, intent, 0);
					views.setOnClickPendingIntent(R.id.btn_clear, pendingIntent);
					awm.updateAppWidget(component, views);
				}
			};
			timer.schedule(task, 1000, 2000);
		}
		@Override
		public void onDestroy() {
			timer.cancel();
			task.cancel();
			timer = null;
			task = null;
			super.onDestroy();
		}
	}
	```

## Widget的声明周期

`Widget`就是一个特殊的广播接收者
1. 当界面上第一个`widget`被创建的时候
	```
	01-14 02:17:14.348: INFO/System.out(1853): onEnabled   当`widget`第一次被创建的时候调用. 非常适合做应用程序的初始化.
	01-14 02:17:14.348: INFO/System.out(1853): onReceive
	01-14 02:17:14.357: INFO/System.out(1853): onUpdate     当有新的`widget`被创建的时候 更新界面的操作. 当时间片到的时候`onupdate()`调用.
	01-14 02:17:14.357: INFO/System.out(1853): onReceive
	```

2. 当界面上第二个`widget`被创建的时候 
	```
	01-14 02:18:10.148: INFO/System.out(1853): onUpdate
	01-14 02:18:10.148: INFO/System.out(1853): onReceive
	```
3. 再创建新的`widget`
	```
	01-14 02:18:10.148: INFO/System.out(1853): onUpdate
	01-14 02:18:10.148: INFO/System.out(1853): onReceive
	```
4. 从界面上移除一个`widget`
    ```
	01-14 02:19:11.709: INFO/System.out(1853): onDeleted
	01-14 02:19:11.709: INFO/System.out(1853): onReceive
    ```
5. 最后一个`widget`被移除
    ```
	01-14 02:19:37.509: INFO/System.out(1853): onDeleted
	01-14 02:19:37.509: INFO/System.out(1853): onReceive
	01-14 02:19:37.509: INFO/System.out(1853): onDisabled  当`widget`从界面上全部移除的时候调用的方法. 非常适合删除临时文件停止后台服务.
	01-14 02:19:37.509: INFO/System.out(1853): onReceive
    ```
6. `widget`就是一个特殊的广播接受者 当有新的事件产生的是 肯定会调用 `onReceive()`;
	

注意: 在不同的手机上  widget的生命周期调用方法 可能有细微的不同.
360桌面 go桌面 awt桌面 腾讯桌面 小米桌面
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 