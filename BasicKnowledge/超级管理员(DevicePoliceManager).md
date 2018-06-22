超级管理员(DevicePoliceManager)
===

DevicePolicyManager    
---

Public interface for managing policies enforced on a device. Most clients of this class must have published a DeviceAdminReceiver that the user 
has currently enabled.       

```java
DevicePolicyManager dpm = (DevicePolicyManager) mContext.getSystemService(Context.DEVICE_POLICY_SERVICE);
if (dpm.isAdminActive(new ComponentName(context, MyAdmin.class))) {
	dpm.resetPassword("321", 0);
	dpm.lockNow();
} 
```

DevicePolicyManager中的方法     

- void lockNow() 
	Make the device lock immediately, as if the lock screen timeout has expired at the point of this call.
	
- boolean resetPassword(String password, int flags) 
	Force a new device unlock password (the password needed to access the entire device, not for individual accounts) on the user.
	
- void wipeData(int flags) 
	Ask the user date be wiped.

- boolean isAdminActive(ComponentName who) 
	Return true if the given administrator component is currently active (enabled) in the system

使用超级管理员DevicePolicyManager的步骤       
1. 要写一个类继承DeviceAdminReceiver
	```java
	public class MyAdmin extends DeviceAdminReceiver {
		//继承就可以不用重写任何内容
	}
	```
    
2. 在清单文件中配置自定义的类
```xml
<receiver
	android:name=".receiver.MyAdmin"
	android:description="@string/admin_des"
	android:label="防卸载"
	android:permission="android.permission.BIND_DEVICE_ADMIN" >
	<meta-data
		android:name="android.app.device_admin"
		android:resource="@xml/device_admin_sample" />

	<intent-filter>
		<action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
	</intent-filter>
</receiver>
```
	
3.  完成第二步中所需的meta-data。在res下新建一个xml文件，device_admin_sample.xml
	```xml
	<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
		<uses-policies>
			<limit-password />
			<watch-login />
			<reset-password />
			<force-lock />
			<wipe-data />
			<expire-password />
			<encrypted-storage />
			<disable-camera />
		</uses-policies>
	</device-admin>
	```
经过上面的三步使用后仍报错，这是因为对于超级管理员，用户必须在手机的应用程序-设备管理员中激活我们的程序才能使用，但是对于一般的人不知道要这样激活。
所以我们要在自己的程序中提供一个按钮让用户点击就能进入这个激活超级管理员的`Activity`
这里可以通过下面的方式来启动这个激活超级管理员的`Activity`
```xml
// 激活设备超级管理员
public void zouni(View view) {
	Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);
	// 初始化要激活的组件
	ComponentName mDeviceAdminSample = new ComponentName(mContext, MyAdmin.class);
	intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, mDeviceAdminSample);
	intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION, "激活可以防止随意卸载应用");
	startActivity(intent);
} 
```
一旦程序有了管理员权限程序就不能卸载了，要想卸载必须在手机上取消该程序的超级管理员，以后如果要防止程序的卸载可以通过这种超级管理员的方式。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
