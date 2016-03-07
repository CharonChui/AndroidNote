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
这一块其实是用了`Binder`机制，上面的`IActivityManager.getDefault()`方法返回的是`ActivityManagerService`的远程接口，所以接下来
我们应该看一下`ActivityManagerService.startActivity()`类。(具体`Binder`机制我们后续会专门写一篇文章，这里就不说了，不然就说不完了)
```java
@Override
public final int startActivity(IApplicationThread caller, String callingPackage,
		Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
		int startFlags, ProfilerInfo profilerInfo, Bundle options) {
	return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
		resultWho, requestCode, startFlags, profilerInfo, options,
		UserHandle.getCallingUserId());
}
```
继续看下`startActivityAsUser()`方法:    
```java
@Override
public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
		Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
		int startFlags, ProfilerInfo profilerInfo, Bundle options, int userId) {
	enforceNotIsolatedCaller("startActivity");
	userId = handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId,
			false, ALLOW_FULL_ONLY, "startActivity", null);
	// TODO: Switch to user app stacks here.
	return mStackSupervisor.startActivityMayWait(caller, -1, callingPackage, intent,
			resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
			profilerInfo, null, null, options, false, userId, null, null);
}
```
继续看一下`mStackSupervisor.startActivityMayWait()`方法,这里`mStackSupervisor`是`ActivityStackSupervisor`类:       
```java
final int startActivityMayWait(IApplicationThread caller, int callingUid,
		String callingPackage, Intent intent, String resolvedType,
		IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
		IBinder resultTo, String resultWho, int requestCode, int startFlags,
		ProfilerInfo profilerInfo, WaitResult outResult, Configuration config,
		Bundle options, boolean ignoreTargetSecurity, int userId,
		IActivityContainer iContainer, TaskRecord inTask) {
	// Refuse possible leaked file descriptors
	if (intent != null && intent.hasFileDescriptors()) {
		throw new IllegalArgumentException("File descriptors passed in Intent");
	}
	boolean componentSpecified = intent.getComponent() != null;

	// Don't modify the client's object!
	intent = new Intent(intent);

	// 解析出要开启的Activity的信息，包名、类名、参数等。
	// Collect information about the target of the Intent.
	ActivityInfo aInfo =
			resolveActivity(intent, resolvedType, startFlags, profilerInfo, userId);

	ActivityContainer container = (ActivityContainer)iContainer;
	synchronized (mService) {
		if (container != null && container.mParentActivity != null &&
				container.mParentActivity.state != RESUMED) {
			// Cannot start a child activity if the parent is not resumed.
			return ActivityManager.START_CANCELED;
		}
		final int realCallingPid = Binder.getCallingPid();
		final int realCallingUid = Binder.getCallingUid();
		int callingPid;
		if (callingUid >= 0) {
			callingPid = -1;
		} else if (caller == null) {
			callingPid = realCallingPid;
			callingUid = realCallingUid;
		} else {
			callingPid = callingUid = -1;
		}

		final ActivityStack stack;
		if (container == null || container.mStack.isOnHomeDisplay()) {
			stack = mFocusedStack;
		} else {
			stack = container.mStack;
		}
		stack.mConfigWillChange = config != null && mService.mConfiguration.diff(config) != 0;
		if (DEBUG_CONFIGURATION) Slog.v(TAG_CONFIGURATION,
				"Starting activity when config will change = " + stack.mConfigWillChange);

		final long origId = Binder.clearCallingIdentity();

		if (aInfo != null &&
				(aInfo.applicationInfo.privateFlags
						&ApplicationInfo.PRIVATE_FLAG_CANT_SAVE_STATE) != 0) {
			// This may be a heavy-weight process!  Check to see if we already
			// have another, different heavy-weight process running.
			if (aInfo.processName.equals(aInfo.applicationInfo.packageName)) {
				if (mService.mHeavyWeightProcess != null &&
						(mService.mHeavyWeightProcess.info.uid != aInfo.applicationInfo.uid ||
						!mService.mHeavyWeightProcess.processName.equals(aInfo.processName))) {
					int appCallingUid = callingUid;
					if (caller != null) {
						ProcessRecord callerApp = mService.getRecordForAppLocked(caller);
						if (callerApp != null) {
							appCallingUid = callerApp.info.uid;
						} else {
							Slog.w(TAG, "Unable to find app for caller " + caller
								  + " (pid=" + callingPid + ") when starting: "
								  + intent.toString());
							ActivityOptions.abort(options);
							return ActivityManager.START_PERMISSION_DENIED;
						}
					}

					IIntentSender target = mService.getIntentSenderLocked(
							ActivityManager.INTENT_SENDER_ACTIVITY, "android",
							appCallingUid, userId, null, null, 0, new Intent[] { intent },
							new String[] { resolvedType }, PendingIntent.FLAG_CANCEL_CURRENT
							| PendingIntent.FLAG_ONE_SHOT, null);

					Intent newIntent = new Intent();
					if (requestCode >= 0) {
						// Caller is requesting a result.
						newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_HAS_RESULT, true);
					}
					newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_INTENT,
							new IntentSender(target));
					if (mService.mHeavyWeightProcess.activities.size() > 0) {
						ActivityRecord hist = mService.mHeavyWeightProcess.activities.get(0);
						newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_CUR_APP,
								hist.packageName);
						newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_CUR_TASK,
								hist.task.taskId);
					}
					newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_NEW_APP,
							aInfo.packageName);
					newIntent.setFlags(intent.getFlags());
					newIntent.setClassName("android",
							HeavyWeightSwitcherActivity.class.getName());
					intent = newIntent;
					resolvedType = null;
					caller = null;
					callingUid = Binder.getCallingUid();
					callingPid = Binder.getCallingPid();
					componentSpecified = true;
					try {
						ResolveInfo rInfo =
							AppGlobals.getPackageManager().resolveIntent(
									intent, null,
									PackageManager.MATCH_DEFAULT_ONLY
									| ActivityManagerService.STOCK_PM_FLAGS, userId);
						aInfo = rInfo != null ? rInfo.activityInfo : null;
						aInfo = mService.getActivityInfoForUser(aInfo, userId);
					} catch (RemoteException e) {
						aInfo = null;
					}
				}
			}
		}
		// 根据上面解析的内容去开启了。。。
		int res = startActivityLocked(caller, intent, resolvedType, aInfo,
				voiceSession, voiceInteractor, resultTo, resultWho,
				requestCode, callingPid, callingUid, callingPackage,
				realCallingPid, realCallingUid, startFlags, options, ignoreTargetSecurity,
				componentSpecified, null, container, inTask);

		Binder.restoreCallingIdentity(origId);

		if (stack.mConfigWillChange) {
			// If the caller also wants to switch to a new configuration,
			// do so now.  This allows a clean switch, as we are waiting
			// for the current activity to pause (so we will not destroy
			// it), and have not yet started the next activity.
			mService.enforceCallingPermission(android.Manifest.permission.CHANGE_CONFIGURATION,
					"updateConfiguration()");
			stack.mConfigWillChange = false;
			if (DEBUG_CONFIGURATION) Slog.v(TAG_CONFIGURATION,
					"Updating to new configuration after starting activity.");
			mService.updateConfigurationLocked(config, null, false, false);
		}

		if (outResult != null) {
			outResult.result = res;
			if (res == ActivityManager.START_SUCCESS) {
				mWaitingActivityLaunched.add(outResult);
				do {
					try {
						mService.wait();
					} catch (InterruptedException e) {
					}
				} while (!outResult.timeout && outResult.who == null);
			} else if (res == ActivityManager.START_TASK_TO_FRONT) {
				ActivityRecord r = stack.topRunningActivityLocked(null);
				if (r.nowVisible && r.state == RESUMED) {
					outResult.timeout = false;
					outResult.who = new ComponentName(r.info.packageName, r.info.name);
					outResult.totalTime = 0;
					outResult.thisTime = 0;
				} else {
					outResult.thisTime = SystemClock.uptimeMillis();
					mWaitingActivityVisible.add(outResult);
					do {
						try {
							mService.wait();
						} catch (InterruptedException e) {
						}
					} while (!outResult.timeout && outResult.who == null);
				}
			}
		}

		return res;
	}
}
```
那就继续看一下`startActivityLocked()`方法:        
```java
final int startActivityLocked(IApplicationThread caller,
		Intent intent, String resolvedType, ActivityInfo aInfo,
		IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
		IBinder resultTo, String resultWho, int requestCode,
		int callingPid, int callingUid, String callingPackage,
		int realCallingPid, int realCallingUid, int startFlags, Bundle options,
		boolean ignoreTargetSecurity, boolean componentSpecified, ActivityRecord[] outActivity,
		ActivityContainer container, TaskRecord inTask) {
	int err = ActivityManager.START_SUCCESS;

	ProcessRecord callerApp = null;
	if (caller != null) {
		// mService就是ActivityManagerService类
		callerApp = mService.getRecordForAppLocked(caller);
		if (callerApp != null) {
			callingPid = callerApp.pid;
			callingUid = callerApp.info.uid;
		} else {
			Slog.w(TAG, "Unable to find app for caller " + caller
				  + " (pid=" + callingPid + ") when starting: "
				  + intent.toString());
			err = ActivityManager.START_PERMISSION_DENIED;
		}
	}

	final int userId = aInfo != null ? UserHandle.getUserId(aInfo.applicationInfo.uid) : 0;

	if (err == ActivityManager.START_SUCCESS) {
		Slog.i(TAG, "START u" + userId + " {" + intent.toShortString(true, true, true, false)
				+ "} from uid " + callingUid
				+ " on display " + (container == null ? (mFocusedStack == null ?
						Display.DEFAULT_DISPLAY : mFocusedStack.mDisplayId) :
						(container.mActivityDisplay == null ? Display.DEFAULT_DISPLAY :
								container.mActivityDisplay.mDisplayId)));
	}

	ActivityRecord sourceRecord = null;
	ActivityRecord resultRecord = null;
	if (resultTo != null) {
		sourceRecord = isInAnyStackLocked(resultTo);
		if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
				"Will send result to " + resultTo + " " + sourceRecord);
		if (sourceRecord != null) {
			if (requestCode >= 0 && !sourceRecord.finishing) {
				resultRecord = sourceRecord;
			}
		}
	}

	final int launchFlags = intent.getFlags();

	if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
		// Transfer the result target from the source activity to the new
		// one being started, including any failures.
		if (requestCode >= 0) {
			ActivityOptions.abort(options);
			return ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT;
		}
		resultRecord = sourceRecord.resultTo;
		if (resultRecord != null && !resultRecord.isInStackLocked()) {
			resultRecord = null;
		}
		resultWho = sourceRecord.resultWho;
		requestCode = sourceRecord.requestCode;
		sourceRecord.resultTo = null;
		if (resultRecord != null) {
			resultRecord.removeResultsLocked(sourceRecord, resultWho, requestCode);
		}
		if (sourceRecord.launchedFromUid == callingUid) {
			// The new activity is being launched from the same uid as the previous
			// activity in the flow, and asking to forward its result back to the
			// previous.  In this case the activity is serving as a trampoline between
			// the two, so we also want to update its launchedFromPackage to be the
			// same as the previous activity.  Note that this is safe, since we know
			// these two packages come from the same uid; the caller could just as
			// well have supplied that same package name itself.  This specifially
			// deals with the case of an intent picker/chooser being launched in the app
			// flow to redirect to an activity picked by the user, where we want the final
			// activity to consider it to have been launched by the previous app activity.
			callingPackage = sourceRecord.launchedFromPackage;
		}
	}

	if (err == ActivityManager.START_SUCCESS && intent.getComponent() == null) {
		// We couldn't find a class that can handle the given Intent.
		// That's the end of that!
		err = ActivityManager.START_INTENT_NOT_RESOLVED;
	}

	if (err == ActivityManager.START_SUCCESS && aInfo == null) {
		// We couldn't find the specific class specified in the Intent.
		// Also the end of the line.
		err = ActivityManager.START_CLASS_NOT_FOUND;
	}

	if (err == ActivityManager.START_SUCCESS
			&& !isCurrentProfileLocked(userId)
			&& (aInfo.flags & FLAG_SHOW_FOR_ALL_USERS) == 0) {
		// Trying to launch a background activity that doesn't show for all users.
		err = ActivityManager.START_NOT_CURRENT_USER_ACTIVITY;
	}

	if (err == ActivityManager.START_SUCCESS && sourceRecord != null
			&& sourceRecord.task.voiceSession != null) {
		// If this activity is being launched as part of a voice session, we need
		// to ensure that it is safe to do so.  If the upcoming activity will also
		// be part of the voice session, we can only launch it if it has explicitly
		// said it supports the VOICE category, or it is a part of the calling app.
		if ((launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) == 0
				&& sourceRecord.info.applicationInfo.uid != aInfo.applicationInfo.uid) {
			try {
				intent.addCategory(Intent.CATEGORY_VOICE);
				if (!AppGlobals.getPackageManager().activitySupportsIntent(
						intent.getComponent(), intent, resolvedType)) {
					Slog.w(TAG,
							"Activity being started in current voice task does not support voice: "
							+ intent);
					err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
				}
			} catch (RemoteException e) {
				Slog.w(TAG, "Failure checking voice capabilities", e);
				err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
			}
		}
	}

	if (err == ActivityManager.START_SUCCESS && voiceSession != null) {
		// If the caller is starting a new voice session, just make sure the target
		// is actually allowing it to run this way.
		try {
			if (!AppGlobals.getPackageManager().activitySupportsIntent(intent.getComponent(),
					intent, resolvedType)) {
				Slog.w(TAG,
						"Activity being started in new voice task does not support: "
						+ intent);
				err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
			}
		} catch (RemoteException e) {
			Slog.w(TAG, "Failure checking voice capabilities", e);
			err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
		}
	}
	// Activity栈
	final ActivityStack resultStack = resultRecord == null ? null : resultRecord.task.stack;

	if (err != ActivityManager.START_SUCCESS) {
		if (resultRecord != null) {
			resultStack.sendActivityResultLocked(-1,
				resultRecord, resultWho, requestCode,
				Activity.RESULT_CANCELED, null);
		}
		ActivityOptions.abort(options);
		return err;
	}

	boolean abort = false;

	final int startAnyPerm = mService.checkPermission(
			START_ANY_ACTIVITY, callingPid, callingUid);

	if (startAnyPerm != PERMISSION_GRANTED) {
		final int componentRestriction = getComponentRestrictionForCallingPackage(
				aInfo, callingPackage, callingPid, callingUid, ignoreTargetSecurity);
		final int actionRestriction = getActionRestrictionForCallingPackage(
				intent.getAction(), callingPackage, callingPid, callingUid);

		if (componentRestriction == ACTIVITY_RESTRICTION_PERMISSION
				|| actionRestriction == ACTIVITY_RESTRICTION_PERMISSION) {
			if (resultRecord != null) {
				resultStack.sendActivityResultLocked(-1,
						resultRecord, resultWho, requestCode,
						Activity.RESULT_CANCELED, null);
			}
			String msg;
			if (actionRestriction == ACTIVITY_RESTRICTION_PERMISSION) {
				msg = "Permission Denial: starting " + intent.toString()
						+ " from " + callerApp + " (pid=" + callingPid
						+ ", uid=" + callingUid + ")" + " with revoked permission "
						+ ACTION_TO_RUNTIME_PERMISSION.get(intent.getAction());
			} else if (!aInfo.exported) {
				msg = "Permission Denial: starting " + intent.toString()
						+ " from " + callerApp + " (pid=" + callingPid
						+ ", uid=" + callingUid + ")"
						+ " not exported from uid " + aInfo.applicationInfo.uid;
			} else {
				msg = "Permission Denial: starting " + intent.toString()
						+ " from " + callerApp + " (pid=" + callingPid
						+ ", uid=" + callingUid + ")"
						+ " requires " + aInfo.permission;
			}
			Slog.w(TAG, msg);
			throw new SecurityException(msg);
		}

		if (actionRestriction == ACTIVITY_RESTRICTION_APPOP) {
			String message = "Appop Denial: starting " + intent.toString()
					+ " from " + callerApp + " (pid=" + callingPid
					+ ", uid=" + callingUid + ")"
					+ " requires " + AppOpsManager.permissionToOp(
							ACTION_TO_RUNTIME_PERMISSION.get(intent.getAction()));
			Slog.w(TAG, message);
			abort = true;
		} else if (componentRestriction == ACTIVITY_RESTRICTION_APPOP) {
			String message = "Appop Denial: starting " + intent.toString()
					+ " from " + callerApp + " (pid=" + callingPid
					+ ", uid=" + callingUid + ")"
					+ " requires appop " + AppOpsManager.permissionToOp(aInfo.permission);
			Slog.w(TAG, message);
			abort = true;
		}
	}

	abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid,
			callingPid, resolvedType, aInfo.applicationInfo);

	if (mService.mController != null) {
		try {
			// The Intent we give to the watcher has the extra data
			// stripped off, since it can contain private information.
			Intent watchIntent = intent.cloneFilter();
			abort |= !mService.mController.activityStarting(watchIntent,
					aInfo.applicationInfo.packageName);
		} catch (RemoteException e) {
			mService.mController = null;
		}
	}

	if (abort) {
		if (resultRecord != null) {
			resultStack.sendActivityResultLocked(-1, resultRecord, resultWho, requestCode,
					Activity.RESULT_CANCELED, null);
		}
		// We pretend to the caller that it was really started, but
		// they will just get a cancel result.
		ActivityOptions.abort(options);
		return ActivityManager.START_SUCCESS;
	}
	// ActivityRecord: An entry in the history stack, representing an activity.
	ActivityRecord r = new ActivityRecord(mService, callerApp, callingUid, callingPackage,
			intent, resolvedType, aInfo, mService.mConfiguration, resultRecord, resultWho,
			requestCode, componentSpecified, voiceSession != null, this, container, options);
	if (outActivity != null) {
		outActivity[0] = r;
	}

	if (r.appTimeTracker == null && sourceRecord != null) {
		// If the caller didn't specify an explicit time tracker, we want to continue
		// tracking under any it has.
		r.appTimeTracker = sourceRecord.appTimeTracker;
	}

	final ActivityStack stack = mFocusedStack;
	if (voiceSession == null && (stack.mResumedActivity == null
			|| stack.mResumedActivity.info.applicationInfo.uid != callingUid)) {
		if (!mService.checkAppSwitchAllowedLocked(callingPid, callingUid,
				realCallingPid, realCallingUid, "Activity start")) {
			PendingActivityLaunch pal =
					new PendingActivityLaunch(r, sourceRecord, startFlags, stack);
			mPendingActivityLaunches.add(pal);
			ActivityOptions.abort(options);
			return ActivityManager.START_SWITCHES_CANCELED;
		}
	}

	if (mService.mDidAppSwitch) {
		// This is the second allowed switch since we stopped switches,
		// so now just generally allow switches.  Use case: user presses
		// home (switches disabled, switch to home, mDidAppSwitch now true);
		// user taps a home icon (coming from home so allowed, we hit here
		// and now allow anyone to switch again).
		mService.mAppSwitchesAllowedTime = 0;
	} else {
		mService.mDidAppSwitch = true;
	}

	doPendingActivityLaunchesLocked(false);
	// 继续调用方法
	err = startActivityUncheckedLocked(r, sourceRecord, voiceSession, voiceInteractor,
			startFlags, true, options, inTask);

	if (err < 0) {
		// If someone asked to have the keyguard dismissed on the next
		// activity start, but we are not actually doing an activity
		// switch...  just dismiss the keyguard now, because we
		// probably want to see whatever is behind it.
		notifyActivityDrawnForKeyguard();
	}
	return err;
}
```
再继续看`startActivityUncheckedLocked`方法：　　　　
```java
final int startActivityUncheckedLocked(final ActivityRecord r, ActivityRecord sourceRecord,
		IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor, int startFlags,
		boolean doResume, Bundle options, TaskRecord inTask) {
	final Intent intent = r.intent;
	final int callingUid = r.launchedFromUid;

	// In some flows in to this function, we retrieve the task record and hold on to it
	// without a lock before calling back in to here...  so the task at this point may
	// not actually be in recents.  Check for that, and if it isn't in recents just
	// consider it invalid.
	if (inTask != null && !inTask.inRecents) {
		Slog.w(TAG, "Starting activity in task not in recents: " + inTask);
		inTask = null;
	}

	// 判断不同的启动模式
	final boolean launchSingleTop = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TOP;
	final boolean launchSingleInstance = r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE;
	final boolean launchSingleTask = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK;

	.....

	// We may want to try to place the new activity in to an existing task.  We always
	// do this if the target activity is singleTask or singleInstance; we will also do
	// this if NEW_TASK has been requested, and there is not an additional qualifier telling
	// us to still place it in a new task: multi task, always doc mode, or being asked to
	// launch this as a new task behind the current one.
	if (((launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0 &&
			(launchFlags & Intent.FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
			|| launchSingleInstance || launchSingleTask) {
		// If bring to front is requested, and no result is requested and we have not
		// been given an explicit task to launch in to, and
		// we can find a task that was started with this same
		// component, then instead of launching bring that one to the front.
		if (inTask == null && r.resultTo == null) {
			// See if there is a task to bring to the front.  If this is
			// a SINGLE_INSTANCE activity, there can be one and only one
			// instance of it in the history, and it is always in its own
			// unique task, so we do a special search.
			ActivityRecord intentActivity = !launchSingleInstance ?
					findTaskLocked(r) : findActivityLocked(intent, r.info);
			if (intentActivity != null) {
				// When the flags NEW_TASK and CLEAR_TASK are set, then the task gets reused
				// but still needs to be a lock task mode violation since the task gets
				// cleared out and the device would otherwise leave the locked task.
				if (isLockTaskModeViolation(intentActivity.task,
						(launchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
						== (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))) {
					showLockTaskToast();
					Slog.e(TAG, "startActivityUnchecked: Attempt to violate Lock Task Mode");
					return ActivityManager.START_RETURN_LOCK_TASK_MODE_VIOLATION;
				}
				if (r.task == null) {
					r.task = intentActivity.task;
				}
				if (intentActivity.task.intent == null) {
					// This task was started because of movement of
					// the activity based on affinity...  now that we
					// are actually launching it, we can assign the
					// base intent.
					intentActivity.task.setIntent(r);
				}
				targetStack = intentActivity.task.stack;
				targetStack.mLastPausedActivity = null;
				// If the target task is not in the front, then we need
				// to bring it to the front...  except...  well, with
				// SINGLE_TASK_LAUNCH it's not entirely clear.  We'd like
				// to have the same behavior as if a new instance was
				// being started, which means not bringing it to the front
				// if the caller is not itself in the front.
				final ActivityStack focusStack = getFocusedStack();
				ActivityRecord curTop = (focusStack == null)
						? null : focusStack.topRunningNonDelayedActivityLocked(notTop);
				boolean movedToFront = false;
				if (curTop != null && (curTop.task != intentActivity.task ||
						curTop.task != focusStack.topTask())) {
					r.intent.addFlags(Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT);
					if (sourceRecord == null || (sourceStack.topActivity() != null &&
							sourceStack.topActivity().task == sourceRecord.task)) {
						// We really do want to push this one into the user's face, right now.
						if (launchTaskBehind && sourceRecord != null) {
							intentActivity.setTaskToAffiliateWith(sourceRecord.task);
						}
						movedHome = true;
						targetStack.moveTaskToFrontLocked(intentActivity.task, noAnimation,
								options, r.appTimeTracker, "bringingFoundTaskToFront");
						movedToFront = true;
						if ((launchFlags &
								(FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME))
								== (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME)) {
							// Caller wants to appear on home activity.
							intentActivity.task.setTaskToReturnTo(HOME_ACTIVITY_TYPE);
						}
						options = null;
					}
				}
				if (!movedToFront) {
					if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Bring to front target: " + targetStack
							+ " from " + intentActivity);
					targetStack.moveToFront("intentActivityFound");
				}

				// If the caller has requested that the target task be
				// reset, then do so.
				if ((launchFlags&Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) != 0) {
					intentActivity = targetStack.resetTaskIfNeededLocked(intentActivity, r);
				}
				if ((startFlags & ActivityManager.START_FLAG_ONLY_IF_NEEDED) != 0) {
					// We don't need to start a new activity, and
					// the client said not to do anything if that
					// is the case, so this is it!  And for paranoia, make
					// sure we have correctly resumed the top activity.
					if (doResume) {
						resumeTopActivitiesLocked(targetStack, null, options);

						// Make sure to notify Keyguard as well if we are not running an app
						// transition later.
						if (!movedToFront) {
							notifyActivityDrawnForKeyguard();
						}
					} else {
						ActivityOptions.abort(options);
					}
					return ActivityManager.START_RETURN_INTENT_TO_CALLER;
				}
				if ((launchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
						== (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK)) {
					// The caller has requested to completely replace any
					// existing task with its new activity.  Well that should
					// not be too hard...
					reuseTask = intentActivity.task;
					reuseTask.performClearTaskLocked();
					reuseTask.setIntent(r);
				} else if ((launchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0
						|| launchSingleInstance || launchSingleTask) {
					// In this situation we want to remove all activities
					// from the task up to the one being started.  In most
					// cases this means we are resetting the task to its
					// initial state.
					ActivityRecord top =
							intentActivity.task.performClearTaskLocked(r, launchFlags);
					if (top != null) {
						if (top.frontOfTask) {
							// Activity aliases may mean we use different
							// intents for the top activity, so make sure
							// the task now has the identity of the new
							// intent.
							top.task.setIntent(r);
						}
						ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT,
								r, top.task);
						top.deliverNewIntentLocked(callingUid, r.intent, r.launchedFromPackage);
					} else {
						// A special case: we need to start the activity because it is not
						// currently running, and the caller has asked to clear the current
						// task to have this activity at the top.
						addingToTask = true;
						// Now pretend like this activity is being started by the top of its
						// task, so it is put in the right place.
						sourceRecord = intentActivity;
						TaskRecord task = sourceRecord.task;
						if (task != null && task.stack == null) {
							// Target stack got cleared when we all activities were removed
							// above. Go ahead and reset it.
							targetStack = computeStackFocus(sourceRecord, false /* newTask */);
							targetStack.addTask(
									task, !launchTaskBehind /* toTop */, false /* moving */);
						}

					}
				} else if (r.realActivity.equals(intentActivity.task.realActivity)) {
					// In this case the top activity on the task is the
					// same as the one being launched, so we take that
					// as a request to bring the task to the foreground.
					// If the top activity in the task is the root
					// activity, deliver this new intent to it if it
					// desires.
					if (((launchFlags&Intent.FLAG_ACTIVITY_SINGLE_TOP) != 0 || launchSingleTop)
							&& intentActivity.realActivity.equals(r.realActivity)) {
						ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT, r,
								intentActivity.task);
						if (intentActivity.frontOfTask) {
							intentActivity.task.setIntent(r);
						}
						intentActivity.deliverNewIntentLocked(callingUid, r.intent,
								r.launchedFromPackage);
					} else if (!r.intent.filterEquals(intentActivity.task.intent)) {
						// In this case we are launching the root activity
						// of the task, but with a different intent.  We
						// should start a new instance on top.
						addingToTask = true;
						sourceRecord = intentActivity;
					}
				} else if ((launchFlags&Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) == 0) {
					// In this case an activity is being launched in to an
					// existing task, without resetting that task.  This
					// is typically the situation of launching an activity
					// from a notification or shortcut.  We want to place
					// the new activity on top of the current task.
					addingToTask = true;
					sourceRecord = intentActivity;
				} else if (!intentActivity.task.rootWasReset) {
					// In this case we are launching in to an existing task
					// that has not yet been started from its front door.
					// The current task has been brought to the front.
					// Ideally, we'd probably like to place this new task
					// at the bottom of its stack, but that's a little hard
					// to do with the current organization of the code so
					// for now we'll just drop it.
					intentActivity.task.setIntent(r);
				}
				if (!addingToTask && reuseTask == null) {
					// We didn't do anything...  but it was needed (a.k.a., client
					// don't use that intent!)  And for paranoia, make
					// sure we have correctly resumed the top activity.
					if (doResume) {
						targetStack.resumeTopActivityLocked(null, options);
						if (!movedToFront) {
							// Make sure to notify Keyguard as well if we are not running an app
							// transition later.
							notifyActivityDrawnForKeyguard();
						}
					} else {
						ActivityOptions.abort(options);
					}
					return ActivityManager.START_TASK_TO_FRONT;
				}
			}
		}
	}

	//String uri = r.intent.toURI();
	//Intent intent2 = new Intent(uri);
	//Slog.i(TAG, "Given intent: " + r.intent);
	//Slog.i(TAG, "URI is: " + uri);
	//Slog.i(TAG, "To intent: " + intent2);

	if (r.packageName != null) {
		// If the activity being launched is the same as the one currently
		// at the top, then we need to check if it should only be launched
		// once.
		ActivityStack topStack = mFocusedStack;
		ActivityRecord top = topStack.topRunningNonDelayedActivityLocked(notTop);
		if (top != null && r.resultTo == null) {
			if (top.realActivity.equals(r.realActivity) && top.userId == r.userId) {
				if (top.app != null && top.app.thread != null) {
					if ((launchFlags & Intent.FLAG_ACTIVITY_SINGLE_TOP) != 0
						|| launchSingleTop || launchSingleTask) {
						ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT, top,
								top.task);
						// For paranoia, make sure we have correctly
						// resumed the top activity.
						topStack.mLastPausedActivity = null;
						if (doResume) {
							resumeTopActivitiesLocked();
						}
						ActivityOptions.abort(options);
						if ((startFlags&ActivityManager.START_FLAG_ONLY_IF_NEEDED) != 0) {
							// We don't need to start a new activity, and
							// the client said not to do anything if that
							// is the case, so this is it!
							return ActivityManager.START_RETURN_INTENT_TO_CALLER;
						}
						top.deliverNewIntentLocked(callingUid, r.intent, r.launchedFromPackage);
						return ActivityManager.START_DELIVERED_TO_TOP;
					}
				}
			}
		}

	} else {
		if (r.resultTo != null && r.resultTo.task.stack != null) {
			r.resultTo.task.stack.sendActivityResultLocked(-1, r.resultTo, r.resultWho,
					r.requestCode, Activity.RESULT_CANCELED, null);
		}
		ActivityOptions.abort(options);
		return ActivityManager.START_CLASS_NOT_FOUND;
	}

	boolean newTask = false;
	boolean keepCurTransition = false;

	TaskRecord taskToAffiliate = launchTaskBehind && sourceRecord != null ?
			sourceRecord.task : null;

	// Should this be considered a new task?
	if (r.resultTo == null && inTask == null && !addingToTask
			&& (launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
		newTask = true;
		targetStack = computeStackFocus(r, newTask);
		targetStack.moveToFront("startingNewTask");

		if (reuseTask == null) {
			r.setTask(targetStack.createTaskRecord(getNextTaskId(),
					newTaskInfo != null ? newTaskInfo : r.info,
					newTaskIntent != null ? newTaskIntent : intent,
					voiceSession, voiceInteractor, !launchTaskBehind /* toTop */),
					taskToAffiliate);
			if (DEBUG_TASKS) Slog.v(TAG_TASKS,
					"Starting new activity " + r + " in new task " + r.task);
		} else {
			r.setTask(reuseTask, taskToAffiliate);
		}
		if (isLockTaskModeViolation(r.task)) {
			Slog.e(TAG, "Attempted Lock Task Mode violation r=" + r);
			return ActivityManager.START_RETURN_LOCK_TASK_MODE_VIOLATION;
		}
		if (!movedHome) {
			if ((launchFlags &
					(FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME))
					== (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME)) {
				// Caller wants to appear on home activity, so before starting
				// their own activity we will bring home to the front.
				r.task.setTaskToReturnTo(HOME_ACTIVITY_TYPE);
			}
		}
	} else if (sourceRecord != null) {
		final TaskRecord sourceTask = sourceRecord.task;
		if (isLockTaskModeViolation(sourceTask)) {
			Slog.e(TAG, "Attempted Lock Task Mode violation r=" + r);
			return ActivityManager.START_RETURN_LOCK_TASK_MODE_VIOLATION;
		}
		targetStack = sourceTask.stack;
		targetStack.moveToFront("sourceStackToFront");
		final TaskRecord topTask = targetStack.topTask();
		if (topTask != sourceTask) {
			targetStack.moveTaskToFrontLocked(sourceTask, noAnimation, options,
					r.appTimeTracker, "sourceTaskToFront");
		}
		if (!addingToTask && (launchFlags&Intent.FLAG_ACTIVITY_CLEAR_TOP) != 0) {
			// In this case, we are adding the activity to an existing
			// task, but the caller has asked to clear that task if the
			// activity is already running.
			ActivityRecord top = sourceTask.performClearTaskLocked(r, launchFlags);
			keepCurTransition = true;
			if (top != null) {
				ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT, r, top.task);
				top.deliverNewIntentLocked(callingUid, r.intent, r.launchedFromPackage);
				// For paranoia, make sure we have correctly
				// resumed the top activity.
				targetStack.mLastPausedActivity = null;
				if (doResume) {
					targetStack.resumeTopActivityLocked(null);
				}
				ActivityOptions.abort(options);
				return ActivityManager.START_DELIVERED_TO_TOP;
			}
		} else if (!addingToTask &&
				(launchFlags&Intent.FLAG_ACTIVITY_REORDER_TO_FRONT) != 0) {
			// In this case, we are launching an activity in our own task
			// that may already be running somewhere in the history, and
			// we want to shuffle it to the front of the stack if so.
			final ActivityRecord top = sourceTask.findActivityInHistoryLocked(r);
			if (top != null) {
				final TaskRecord task = top.task;
				task.moveActivityToFrontLocked(top);
				ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT, r, task);
				top.updateOptionsLocked(options);
				top.deliverNewIntentLocked(callingUid, r.intent, r.launchedFromPackage);
				targetStack.mLastPausedActivity = null;
				if (doResume) {
					targetStack.resumeTopActivityLocked(null);
				}
				return ActivityManager.START_DELIVERED_TO_TOP;
			}
		}
		// An existing activity is starting this new activity, so we want
		// to keep the new one in the same task as the one that is starting
		// it.
		r.setTask(sourceTask, null);
		if (DEBUG_TASKS) Slog.v(TAG_TASKS, "Starting new activity " + r
				+ " in existing task " + r.task + " from source " + sourceRecord);

	} else if (inTask != null) {
		// The caller is asking that the new activity be started in an explicit
		// task it has provided to us.
		if (isLockTaskModeViolation(inTask)) {
			Slog.e(TAG, "Attempted Lock Task Mode violation r=" + r);
			return ActivityManager.START_RETURN_LOCK_TASK_MODE_VIOLATION;
		}
		targetStack = inTask.stack;
		targetStack.moveTaskToFrontLocked(inTask, noAnimation, options, r.appTimeTracker,
				"inTaskToFront");

		// Check whether we should actually launch the new activity in to the task,
		// or just reuse the current activity on top.
		ActivityRecord top = inTask.getTopActivity();
		if (top != null && top.realActivity.equals(r.realActivity) && top.userId == r.userId) {
			if ((launchFlags & Intent.FLAG_ACTIVITY_SINGLE_TOP) != 0
					|| launchSingleTop || launchSingleTask) {
				ActivityStack.logStartActivity(EventLogTags.AM_NEW_INTENT, top, top.task);
				if ((startFlags&ActivityManager.START_FLAG_ONLY_IF_NEEDED) != 0) {
					// We don't need to start a new activity, and
					// the client said not to do anything if that
					// is the case, so this is it!
					return ActivityManager.START_RETURN_INTENT_TO_CALLER;
				}
				top.deliverNewIntentLocked(callingUid, r.intent, r.launchedFromPackage);
				return ActivityManager.START_DELIVERED_TO_TOP;
			}
		}

		if (!addingToTask) {
			// We don't actually want to have this activity added to the task, so just
			// stop here but still tell the caller that we consumed the intent.
			ActivityOptions.abort(options);
			return ActivityManager.START_TASK_TO_FRONT;
		}

		r.setTask(inTask, null);
		if (DEBUG_TASKS) Slog.v(TAG_TASKS, "Starting new activity " + r
				+ " in explicit task " + r.task);

	} else {
		// This not being started from an existing activity, and not part
		// of a new task...  just put it in the top task, though these days
		// this case should never happen.
		targetStack = computeStackFocus(r, newTask);
		targetStack.moveToFront("addingToTopTask");
		ActivityRecord prev = targetStack.topActivity();
		r.setTask(prev != null ? prev.task : targetStack.createTaskRecord(getNextTaskId(),
						r.info, intent, null, null, true), null);
		mWindowManager.moveTaskToTop(r.task.taskId);
		if (DEBUG_TASKS) Slog.v(TAG_TASKS, "Starting new activity " + r
				+ " in new guessed " + r.task);
	}

	mService.grantUriPermissionFromIntentLocked(callingUid, r.packageName,
			intent, r.getUriPermissionsLocked(), r.userId);

	if (sourceRecord != null && sourceRecord.isRecentsActivity()) {
		r.task.setTaskToReturnTo(RECENTS_ACTIVITY_TYPE);
	}
	if (newTask) {
		EventLog.writeEvent(EventLogTags.AM_CREATE_TASK, r.userId, r.task.taskId);
	}
	ActivityStack.logStartActivity(EventLogTags.AM_CREATE_ACTIVITY, r, r.task);
	targetStack.mLastPausedActivity = null;
	
	// 上面这部分代码很多，各种启动模式、栈的处理啊，我们就不继续看了，接着看下面的启动。
	targetStack.startActivityLocked(r, newTask, doResume, keepCurTransition, options);
	if (!launchTaskBehind) {
		// Don't set focus on an activity that's going to the back.
		mService.setFocusedActivityLocked(r, "startedActivity");
	}
	return ActivityManager.START_SUCCESS;
}
```
继续看`targetStack.startActivityLocked()`方法,这里`targetStack`就是`ActivityStack`类:      
```java
final void startActivityLocked(ActivityRecord r, boolean newTask,
		boolean doResume, boolean keepCurTransition, Bundle options) {
	TaskRecord rTask = r.task;
	final int taskId = rTask.taskId;
	// mLaunchTaskBehind tasks get placed at the back of the task stack.
	if (!r.mLaunchTaskBehind && (taskForIdLocked(taskId) == null || newTask)) {
		// Last activity in task had been removed or ActivityManagerService is reusing task.
		// Insert or replace.
		// Might not even be in.
		insertTaskAtTop(rTask, r);
		mWindowManager.moveTaskToTop(taskId);
	}
	TaskRecord task = null;
	if (!newTask) {
		// If starting in an existing task, find where that is...
		boolean startIt = true;
		for (int taskNdx = mTaskHistory.size() - 1; taskNdx >= 0; --taskNdx) {
			task = mTaskHistory.get(taskNdx);
			if (task.getTopActivity() == null) {
				// All activities in task are finishing.
				continue;
			}
			if (task == r.task) {
				// Here it is!  Now, if this is not yet visible to the
				// user, then just add it without starting; it will
				// get started when the user navigates back to it.
				if (!startIt) {
					if (DEBUG_ADD_REMOVE) Slog.i(TAG, "Adding activity " + r + " to task "
							+ task, new RuntimeException("here").fillInStackTrace());
					task.addActivityToTop(r);
					r.putInHistory();
					mWindowManager.addAppToken(task.mActivities.indexOf(r), r.appToken,
							r.task.taskId, mStackId, r.info.screenOrientation, r.fullscreen,
							(r.info.flags & ActivityInfo.FLAG_SHOW_FOR_ALL_USERS) != 0,
							r.userId, r.info.configChanges, task.voiceSession != null,
							r.mLaunchTaskBehind);
					if (VALIDATE_TOKENS) {
						validateAppTokensLocked();
					}
					ActivityOptions.abort(options);
					return;
				}
				break;
			} else if (task.numFullscreen > 0) {
				startIt = false;
			}
		}
	}

	// Place a new activity at top of stack, so it is next to interact
	// with the user.

	// If we are not placing the new activity frontmost, we do not want
	// to deliver the onUserLeaving callback to the actual frontmost
	// activity
	if (task == r.task && mTaskHistory.indexOf(task) != (mTaskHistory.size() - 1)) {
		mStackSupervisor.mUserLeaving = false;
		if (DEBUG_USER_LEAVING) Slog.v(TAG_USER_LEAVING,
				"startActivity() behind front, mUserLeaving=false");
	}

	task = r.task;

	// Slot the activity into the history stack and proceed
	if (DEBUG_ADD_REMOVE) Slog.i(TAG, "Adding activity " + r + " to stack to task " + task,
			new RuntimeException("here").fillInStackTrace());
	task.addActivityToTop(r);
	task.setFrontOfTask();

	r.putInHistory();
	if (!isHomeStack() || numActivities() > 0) {
		// We want to show the starting preview window if we are
		// switching to a new task, or the next activity's process is
		// not currently running.
		boolean showStartingIcon = newTask;
		ProcessRecord proc = r.app;
		if (proc == null) {
			proc = mService.mProcessNames.get(r.processName, r.info.applicationInfo.uid);
		}
		if (proc == null || proc.thread == null) {
			showStartingIcon = true;
		}
		if (DEBUG_TRANSITION) Slog.v(TAG_TRANSITION,
				"Prepare open transition: starting " + r);
		if ((r.intent.getFlags() & Intent.FLAG_ACTIVITY_NO_ANIMATION) != 0) {
			mWindowManager.prepareAppTransition(AppTransition.TRANSIT_NONE, keepCurTransition);
			mNoAnimActivities.add(r);
		} else {
			mWindowManager.prepareAppTransition(newTask
					? r.mLaunchTaskBehind
							? AppTransition.TRANSIT_TASK_OPEN_BEHIND
							: AppTransition.TRANSIT_TASK_OPEN
					: AppTransition.TRANSIT_ACTIVITY_OPEN, keepCurTransition);
			mNoAnimActivities.remove(r);
		}
		mWindowManager.addAppToken(task.mActivities.indexOf(r),
				r.appToken, r.task.taskId, mStackId, r.info.screenOrientation, r.fullscreen,
				(r.info.flags & ActivityInfo.FLAG_SHOW_FOR_ALL_USERS) != 0, r.userId,
				r.info.configChanges, task.voiceSession != null, r.mLaunchTaskBehind);
		boolean doShow = true;
		if (newTask) {
			// Even though this activity is starting fresh, we still need
			// to reset it to make sure we apply affinities to move any
			// existing activities from other tasks in to it.
			// If the caller has requested that the target task be
			// reset, then do so.
			if ((r.intent.getFlags() & Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) != 0) {
				resetTaskIfNeededLocked(r, r);
				doShow = topRunningNonDelayedActivityLocked(null) == r;
			}
		} else if (options != null && new ActivityOptions(options).getAnimationType()
				== ActivityOptions.ANIM_SCENE_TRANSITION) {
			doShow = false;
		}
		if (r.mLaunchTaskBehind) {
			// Don't do a starting window for mLaunchTaskBehind. More importantly make sure we
			// tell WindowManager that r is visible even though it is at the back of the stack.
			mWindowManager.setAppVisibility(r.appToken, true);
			ensureActivitiesVisibleLocked(null, 0);
		} else if (SHOW_APP_STARTING_PREVIEW && doShow) {
			// Figure out if we are transitioning from another activity that is
			// "has the same starting icon" as the next one.  This allows the
			// window manager to keep the previous window it had previously
			// created, if it still had one.
			ActivityRecord prev = mResumedActivity;
			if (prev != null) {
				// We don't want to reuse the previous starting preview if:
				// (1) The current activity is in a different task.
				if (prev.task != r.task) {
					prev = null;
				}
				// (2) The current activity is already displayed.
				else if (prev.nowVisible) {
					prev = null;
				}
			}
			mWindowManager.setAppStartingWindow(
					r.appToken, r.packageName, r.theme,
					mService.compatibilityInfoForPackageLocked(
							r.info.applicationInfo), r.nonLocalizedLabel,
					r.labelRes, r.icon, r.logo, r.windowFlags,
					prev != null ? prev.appToken : null, showStartingIcon);
			r.mStartingWindowShown = true;
		}
	} else {
		// If this is the first activity, don't do any fancy animations,
		// because there is nothing for it to animate on top of.
		mWindowManager.addAppToken(task.mActivities.indexOf(r), r.appToken,
				r.task.taskId, mStackId, r.info.screenOrientation, r.fullscreen,
				(r.info.flags & ActivityInfo.FLAG_SHOW_FOR_ALL_USERS) != 0, r.userId,
				r.info.configChanges, task.voiceSession != null, r.mLaunchTaskBehind);
		ActivityOptions.abort(options);
		options = null;
	}
	if (VALIDATE_TOKENS) {
		validateAppTokensLocked();
	}

	if (doResume) {
		mStackSupervisor.resumeTopActivitiesLocked(this, r, options);
	}
}
```

 在Android应用程序框架层中，是由ActivityManagerService组件负责为Android应用程序创建新的进程的，它本来也是运行在一个独立的进程之中，不过这个进程是在系统启动的过程中创建的。
 ActivityManagerService组件一般会在什么情况下会为应用程序创建一个新的进程呢？当系统决定要在一个新的进程中启动一个Activity或者Service时，它就会创建一个新的进程了，
 然后在这个新的进程中启动这个Activity或者Service

    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 