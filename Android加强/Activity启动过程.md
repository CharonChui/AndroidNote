Activity启动过程
===

前两天面试了天猫的开发，被问到了`Activity`启动过程，不懂啊....
今天就来分析一下，我们开启`Activity`主要有两种方式:    

- 通过桌面图标启动，桌面就是`Launcher`其实他也是一个应用程序，他也是继承`Activity`。
- 在程序内部调用`startActivity()`开启。

而`Launcher`点击图标其实也是调用了`Activity`的`startActivity()`方法，所以我们就从`startActivity()`方法入手了。     
首先看一下`Activity`类中的`startActivity()`方法:    
```java
@Override
public void startActivity(Intent intent) {
	this.startActivity(intent, null);
}
```
继续看`startActivity(intent, null)`:    
```java
/**
 * Launch a new activity.  You will not receive any information about when
 * the activity exits.  This implementation overrides the base version,
 * providing information about
 * the activity performing the launch.  Because of this additional
 * information, the {@link Intent#FLAG_ACTIVITY_NEW_TASK} launch flag is not
 * required; if not specified, the new activity will be added to the
 * task of the caller.
 *
 * <p>This method throws {@link android.content.ActivityNotFoundException}
 * if there was no Activity found to run the given Intent.
 *
 * @param intent The intent to start.
 * @param options Additional options for how the Activity should be started.
 * See {@link android.content.Context#startActivity(Intent, Bundle)
 * Context.startActivity(Intent, Bundle)} for more details.
 *
 * @throws android.content.ActivityNotFoundException
 *
 * @see {@link #startActivity(Intent)}
 * @see #startActivityForResult
 */
@Override
public void startActivity(Intent intent, @Nullable Bundle options) {
	if (options != null) {
		startActivityForResult(intent, -1, options);
	} else {
		// Note we want to go through this call for compatibility with
		// applications that may have overridden the method.
		startActivityForResult(intent, -1);
	}
}
```
接下来会调用`startActivityForResult()`方法(注释也很重要啊，里面也说明了singleTask启动模式时该方法不会走到回调中):     
```java
/**
 * Launch an activity for which you would like a result when it finished.
 * When this activity exits, your
 * onActivityResult() method will be called with the given requestCode.
 * Using a negative requestCode is the same as calling
 * {@link #startActivity} (the activity is not launched as a sub-activity).
 *
 * <p>Note that this method should only be used with Intent protocols
 * that are defined to return a result.  In other protocols (such as
 * {@link Intent#ACTION_MAIN} or {@link Intent#ACTION_VIEW}), you may
 * not get the result when you expect.  For example, if the activity you
 * are launching uses the singleTask launch mode, it will not run in your
 * task and thus you will immediately receive a cancel result.
 *
 * <p>As a special case, if you call startActivityForResult() with a requestCode
 * >= 0 during the initial onCreate(Bundle savedInstanceState)/onResume() of your
 * activity, then your window will not be displayed until a result is
 * returned back from the started activity.  This is to avoid visible
 * flickering when redirecting to another activity.
 *
 * <p>This method throws {@link android.content.ActivityNotFoundException}
 * if there was no Activity found to run the given Intent.
 *
 * @param intent The intent to start.
 * @param requestCode If >= 0, this code will be returned in
 *                    onActivityResult() when the activity exits.
 * @param options Additional options for how the Activity should be started.
 * See {@link android.content.Context#startActivity(Intent, Bundle)
 * Context.startActivity(Intent, Bundle)} for more details.
 *
 * @throws android.content.ActivityNotFoundException
 *
 * @see #startActivity
 */
public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options) {
	// mParent也是Activity类，通过名字就能看明白了。
	if (mParent == null) {
		// 开始了啊
		Instrumentation.ActivityResult ar =
			mInstrumentation.execStartActivity(
				this, mMainThread.getApplicationThread(), mToken, this,
				intent, requestCode, options);
		if (ar != null) {
			mMainThread.sendActivityResult(
				mToken, mEmbeddedID, requestCode, ar.getResultCode(),
				ar.getResultData());
		}
		if (requestCode >= 0) {
			// If this start is requesting a result, we can avoid making
			// the activity visible until the result is received.  Setting
			// this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
			// activity hidden during this time, to avoid flickering.
			// This can only be done when a result is requested because
			// that guarantees we will get information back when the
			// activity is finished, no matter what happens to it.
			mStartedActivity = true;
		}

		cancelInputsAndStartExitTransition(options);
		// TODO Consider clearing/flushing other event sources and events for child windows.
	} else {
		if (options != null) {
			mParent.startActivityFromChild(this, intent, requestCode, options);
		} else {
			// Note we want to go through this method for compatibility with
			// existing applications that may have overridden it.
			mParent.startActivityFromChild(this, intent, requestCode);
		}
	}
}
```
里面调用了`mInstrumentation.execStartActivity`这是啥玩意，先看一下`mInstrumentation`属性，他是`Instrumentation`类，我们看下文档
```java
/**
 * Base class for implementing application instrumentation code.  When running
 * with instrumentation turned on, this class will be instantiated for you
 * before any of the application code, allowing you to monitor all of the
 * interaction the system has with the application.  An Instrumentation
 * implementation is described to the system through an AndroidManifest.xml's
 * &lt;instrumentation&gt; tag.
 */
public class Instrumentation {
	...
}
```
放狗查了下`Instrumentation`的意思是仪器、工具、装置的意思。我就大体翻一下(英语不好- -~，可能不准确)该类是实现应用程序代码的基类，当该类在
启动的状态下运行时，该类会在其他任何应用程序运行前进行初始化，允许你件事所有应用程序与系统的交互。一个`Instrumentation`实例会通过`Manifest`文件
中的`<instrumenttation`标签描述给系统。        
所以继续看一下`mInstrumentation.execStartActivity()`:       
```java
/**
   // 下面的注释说的很明白了默认实现会更新任何活跃的对象并分发给系统的`activity manager`
 * Execute a startActivity call made by the application.  The default 
 * implementation takes care of updating any active {@link ActivityMonitor}
 * objects and dispatches this call to the system activity manager; you can
 * override this to watch for the application to start an activity, and 
 * modify what happens when it does. 
 *
 * <p>This method returns an {@link ActivityResult} object, which you can 
 * use when intercepting application calls to avoid performing the start 
 * activity action but still return the result the application is 
 * expecting.  To do this, override this method to catch the call to start 
 * activity so that it returns a new ActivityResult containing the results 
 * you would like the application to see, and don't call up to the super 
 * class.  Note that an application is only expecting a result if 
 * <var>requestCode</var> is &gt;= 0.
 *
 * <p>This method throws {@link android.content.ActivityNotFoundException}
 * if there was no Activity found to run the given Intent.
 *
 * @param who The Context from which the activity is being started.
 * @param contextThread The main thread of the Context from which the activity
 *                      is being started.
 * @param token Internal token identifying to the system who is starting 
 *              the activity; may be null.
 * @param target Which activity is performing the start (and thus receiving 
 *               any result); may be null if this call is not being made
 *               from an activity.
 * @param intent The actual Intent to start.
 * @param requestCode Identifier for this request's result; less than zero 
 *                    if the caller is not expecting a result.
 * @param options Addition options.
 *
 * @return To force the return of a particular result, return an 
 *         ActivityResult object containing the desired data; otherwise
 *         return null.  The default implementation always returns null.
 *
 * @throws android.content.ActivityNotFoundException
 *
 * @see Activity#startActivity(Intent)
 * @see Activity#startActivityForResult(Intent, int)
 * @see Activity#startActivityFromChild
 *
 * {@hide}
 */
public ActivityResult execStartActivity(
		Context who, IBinder contextThread, IBinder token, Activity target,
		Intent intent, int requestCode, Bundle options) {
	IApplicationThread whoThread = (IApplicationThread) contextThread;
	Uri referrer = target != null ? target.onProvideReferrer() : null;
	if (referrer != null) {
		intent.putExtra(Intent.EXTRA_REFERRER, referrer);
	}
	if (mActivityMonitors != null) {
		synchronized (mSync) {
			final int N = mActivityMonitors.size();
			for (int i=0; i<N; i++) {
				final ActivityMonitor am = mActivityMonitors.get(i);
				if (am.match(who, null, intent)) {
					am.mHits++;
					if (am.isBlocking()) {
						return requestCode >= 0 ? am.getResult() : null;
					}
					break;
				}
			}
		}
	}
	try {
		intent.migrateExtraStreamToClipData();
		intent.prepareToLeaveProcess();
		// 通过注释然后再结合代码一看我们就知道这应该就是分发到系统activity manager的过程
		int result = ActivityManagerNative.getDefault()
			.startActivity(whoThread, who.getBasePackageName(), intent,
					intent.resolveTypeIfNeeded(who.getContentResolver()),
					token, target != null ? target.mEmbeddedID : null,
					requestCode, 0, null, options);
		checkStartActivityResult(result, intent);
	} catch (RemoteException e) {
		throw new RuntimeException("Failure from system", e);
	}
	return null;
}
```
那就直接看下`ActivityManagerNative.getDefault().startActivity()`方法，看之前我们先看看`ActivityManagerNative.getDefault()`是什么鬼:     
```java
/**
 * Retrieve the system's default/global activity manager.
 */
static public IActivityManager getDefault() {
	return gDefault.get();
}
```
注释说的明白的就是拿到系统默认/全局的`activity manager`，通过名字也能看出来`IActivityManager`是个接口，那就继续看`IActivityManager.startActivity()`方法吧:     
```java
public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
            String resolvedType, IBinder resultTo, String resultWho, int requestCode, int flags,
            ProfilerInfo profilerInfo, Bundle options) throws RemoteException;
```
这里就顺便看一下`IActivityManager`接口是神马鬼：  
```
/**
 * System private API for talking with the activity manager service.  This
 * provides calls from the application back to the activity manager.
 *
 * {@hide}
 */
public interface IActivityManager extends IInterface {
	...
}
```
但是看到这里我不知道怎么继续往下分析了啊，既然是接口我们要找到实现类啊，又是通过`ActivityManagerNative.getDefault()`得到的`IActivityManager`实例，
所以这里我们再看下`ActivityManagerNative`类。
```java
/** {@hide} */
public abstract class ActivityManagerNative extends Binder implements IActivityManager {
	...
}
```
啊！ 隐藏的，连个注释也没有，我拖动了下这个类一看`5992`行，不知道从哪下手了，还能从哪下手啊，当然是从`ActivityManagerNative.getDefault()`方法啊
```java
public abstract class ActivityManagerNative extends Binder implements IActivityManager {

	// 继承Binder接口，而且asInterFace方法，我好想明白了点什么
    /**
     * Cast a Binder object into an activity manager interface, generating
     * a proxy if needed.
     */
    static public IActivityManager asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }
        IActivityManager in =
            (IActivityManager)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }
		
        return new ActivityManagerProxy(obj);
    }

    /**
     * Retrieve the system's default/global activity manager.
     */
    static public IActivityManager getDefault() {
		// 使用了getDefaule的get方法
        return gDefault.get();
    }
	....
}	
```
继续看：　　
```java
private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
	protected IActivityManager create() {
	    // 进程间通信了
		IBinder b = ServiceManager.getService("activity");
		if (false) {
			Log.v("ActivityManager", "default service binder = " + b);
		}
		// 看到了吗？用到了刚才我们说的asInterface方法
		IActivityManager am = asInterface(b);
		if (false) {
			Log.v("ActivityManager", "default service = " + am);
		}
		return am;
	}
};
```
好了，我们再看下`asInterface()`方法:　　　　
```java
static public IActivityManager asInterface(IBinder obj) {
	if (obj == null) {
		return null;
	}
	IActivityManager in =
		(IActivityManager)obj.queryLocalInterface(descriptor);
	if (in != null) {
		return in;
	}
	// 在这里
	return new ActivityManagerProxy(obj);
}
```
终于找到了其实就是`ActivityManagerProxy`类，刚才我们找到`IActivityManager`接口的`startActivity()` 方法，现在终于找到了在这里使用的是`IActivityManager`接口实现类的
`ActivityManagerProxy`类，我们来看一下该类实现的`startActivity()`方法:     
```java
class ActivityManagerProxy implements IActivityManager
{
    public ActivityManagerProxy(IBinder remote)
    {
        mRemote = remote;
    }

    public IBinder asBinder()
    {
        return mRemote;
    }
	public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
			String resolvedType, IBinder resultTo, String resultWho, int requestCode,
			int startFlags, ProfilerInfo profilerInfo, Bundle options) throws RemoteException {
		Parcel data = Parcel.obtain();
		Parcel reply = Parcel.obtain();
		data.writeInterfaceToken(IActivityManager.descriptor);
		data.writeStrongBinder(caller != null ? caller.asBinder() : null);
		data.writeString(callingPackage);
		intent.writeToParcel(data, 0);
		data.writeString(resolvedType);
		data.writeStrongBinder(resultTo);
		data.writeString(resultWho);
		data.writeInt(requestCode);
		data.writeInt(startFlags);
		if (profilerInfo != null) {
			data.writeInt(1);
			profilerInfo.writeToParcel(data, Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
		} else {
			data.writeInt(0);
		}
		if (options != null) {
			data.writeInt(1);
			options.writeToParcel(data, 0);
		} else {
			data.writeInt(0);
		}
		mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0);
		reply.readException();
		int result = reply.readInt();
		reply.recycle();
		data.recycle();
		return result;
	}
	...
}	
```
再继续我就不知道走到哪里了....

    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 