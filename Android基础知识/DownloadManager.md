DownloadManager
===

`DownloadManager`是`Android 2.3`提供的下载类。功能非常强大，完全没有必要自己实现下载。

一、DownloadManager简单介绍 
---
`DownloadManager`是系统开放给第三方应用使用的类，包含两个静态内部类`DownloadManager.Query`和`DownloadManager.Request`。**`DownloadManager.Request`用来请求一个下载，`DownloadManager.Query`用来查询下载信息**。    

主要方法介绍
```java
**public long enqueue(Request request)**    
执行下载，返回downloadId，downloadId可用于后面查询下载信息。若网络不满足条件、Sdcard挂载中、超过最大并发数等异常会等待下载，正常则直接下载。 

**public int remove(long… ids)**
删除下载，若下载中取消下载。会同时删除下载文件和记录。 

**public Cursor query(Query query)**
查询下载信息。 

public static Long getRecommendedMaxBytesOverMobile(Context context)     
通过移动网络下载的最大字节数

public String getMimeTypeForDownloadedFile(long id)      
得到下载的mimeType，如何设置后面会进行介绍
 
其它：通过查看代码我们可以发现还有个CursorTranslator私有静态内部类。这个类主要对Query做了一层代理。将DownloadProvider和DownloadManager之间做个映射。将DownloadProvider中的十几种状态对应到了DownloadManager中的五种状态，DownloadProvider中的失败、暂停原因转换为了DownloadManager的原因。
```

二、示例代码
---
1. AndroidManifest中添加权限
```xml
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

2. 开始下载
```java
	DownloadManager downloadManager = (DownloadManager)getSystemService(DOWNLOAD_SERVICE);
	String apkUrl = "http://img.meilishuo.net/css/images/AndroidShare/Meilishuo_3.6.1_10006.apk";
	DownloadManager.Request request = new DownloadManager.Request(Uri.parse(apkUrl));
	request.setDestinationInExternalPublicDir("Trinea", "MeiLiShuo.apk");
	// request.setTitle("MeiLiShuo");
	// request.setDescription("MeiLiShuo desc");
	// request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
	// request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);
	// request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_HIDDEN);
	// request.setMimeType("application/com.trinea.download.file");
	long downloadId = downloadManager.enqueue(request);
```
    上面调用downloadManager的enqueue接口进行下载，返回唯一的downloadId。DownloadManager.Request除了构造函数的Uri必须外，其他设置都为可选设置。下面逐个介绍下：     
request.setDestinationInExternalPublicDir(“Trinea”,“MeiLiShuo.apk”);表示设置下载地址为sd卡的Trinea文件夹，文件名为MeiLiShuo.apk。

```java
setDestinationInExternalPublicDir的源码为：
public Request setDestinationInExternalPublicDir(String dirType, String subPath) {
	File file = Environment.getExternalStoragePublicDirectory(dirType);

	Xlog.v(XLOGTAG, "setExternalPublicDir: dirType " + 
			dirType + " subPath " + subPath + 
			"file" + file);

	if (file.exists()) {
		if (!file.isDirectory()) {
			throw new IllegalStateException(file.getAbsolutePath() +
					" already exists and is not a directory");
		}
	} else {
		if (!file.mkdir()) {
			throw new IllegalStateException("Unable to create directory: "+
					file.getAbsolutePath());
		}
	}
	setDestinationFromBase(file, subPath);
	return this;
}
```

从源码中我们可以看出下载完整目录为Environment.getExternalStoragePublicDirectory(dirType)。不过file是通过file.mkdir()创建的，这样如果上级目录不存在就会新建文件夹异常。所以下载前我们最好自己调用File的mkdirs方法递归创建子目录，如下：

```java
File folder = new File(folderName);
return (folder.exists() && folder.isDirectory()) ? true : folder.mkdirs();
```
否则，会报异常
其他设置下载路径接口为setDestinationUri，setDestinationInExternalFilesDir，setDestinationToSystemCache。其中setDestinationToSystemCache仅限系统app使用。

request.allowScanningByMediaScanner();表示允许MediaScanner扫描到这个文件，默认不允许。
request.setTitle(“MeiLiShuo”);设置下载中通知栏提示的标题
request.setDescription(“MeiLiShuo desc”);设置下载中通知栏提示的介绍
request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
表示下载进行中和下载完成的通知栏是否显示。默认只显示下载中通知。VISIBILITY_VISIBLE_NOTIFY_COMPLETED表示下载完成后显示通知栏提示。VISIBILITY_HIDDEN表示不显示任何通知栏提示，这个需要在AndroidMainfest中添加权限android.permission.DOWNLOAD_WITHOUT_NOTIFICATION.
 
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);
表示下载允许的网络类型，默认在任何网络下都允许下载。有NETWORK_MOBILE、NETWORK_WIFI、NETWORK_BLUETOOTH三种及其组合可供选择。如果只允许wifi下载，而当前网络为3g，则下载会等待。
request.setAllowedOverRoaming(boolean allow)移动网络情况下是否允许漫游。
 
request.setMimeType(“application/com.trinea.download.file”);
设置下载文件的mineType。因为下载管理Ui中点击某个已下载完成文件及下载完成点击通知栏提示都会根据mimeType去打开文件，所以我们可以利用这个属性。比如上面设置了mimeType为application/com.trinea.download.file，我们可以同时设置某个Activity的intent-filter为application/com.trinea.download.file，用于响应点击的打开文件。

```xml
<intent-filter>
	<action android:name="android.intent.action.VIEW" />

	<category android:name="android.intent.category.DEFAULT" />

	<data android:mimeType="application/com.trinea.download.file" />
</intent-filter>
```

request.addRequestHeader(String header, String value)
添加请求下载的网络链接的http头，比如User-Agent，gzip压缩等

3. 下载进度状态监听及查询
```java
	class DownloadChangeObserver extends ContentObserver {

	public DownloadChangeObserver(){
		super(handler);
	}

	@Override
	public void onChange(boolean selfChange) {
		updateView();
	}

}

public void updateView() {
	int[] bytesAndStatus = downloadManagerPro.getBytesAndStatus(downloadId);
	handler.sendMessage(handler.obtainMessage(0, bytesAndStatus[0], bytesAndStatus[1],
											  bytesAndStatus[2]));
}

private DownloadChangeObserver downloadObserver;

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.download_manager_demo);
	……
	downloadObserver = new DownloadChangeObserver();
}

@Override
protected void onResume() {
	super.onResume();
	/** observer download change **/
	getContentResolver().registerContentObserver(DownloadManagerPro.CONTENT_URI, true,
												 downloadObserver);
}

@Override
protected void onPause() {
	super.onPause();
	getContentResolver().unregisterContentObserver(downloadObserver);
}
```
其中我们会监听Uri.parse(“content://downloads/my_downloads”)。然后查询下载状态和进度，发送handler进行更新，handler中处理就是设置进度条和状态等。
其中DownloadManagerPro.getBytesAndStatus的主要代码如下，

```java
public int[] getBytesAndStatus(long downloadId) {
	int[] bytesAndStatus = new int[] { -1, -1, 0 };
	DownloadManager.Query query = new DownloadManager.Query().setFilterById(downloadId);
	Cursor c = null;
	try {
		c = downloadManager.query(query);
		if (c != null && c.moveToFirst()) {
			bytesAndStatus[0] = c.getInt(c.getColumnIndexOrThrow(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR));
			bytesAndStatus[1] = c.getInt(c.getColumnIndexOrThrow(DownloadManager.COLUMN_TOTAL_SIZE_BYTES));
			bytesAndStatus[2] = c.getInt(c.getColumnIndex(DownloadManager.COLUMN_STATUS));
		}
	} finally {
		if (c != null) {
			c.close();
		}
	}
	return bytesAndStatus;
}
```
从上面代码可以看出我们主要调用DownloadManager.Query()进行查询。DownloadManager.Query为下载管理对外开放的信息查询类，主要包括以下接口：
setFilterById(long… ids)根据下载id进行过滤
setFilterByStatus(int flags)根据下载状态进行过滤
setOnlyIncludeVisibleInDownloadsUi(boolean value)根据是否在download ui中可见进行过滤。
 
orderBy(String column, int direction)根据列进行排序，不过目前仅支持DownloadManager.COLUMN_LAST_MODIFIED_TIMESTAMP和DownloadManager.COLUMN_TOTAL_SIZE_BYTES排序。
 
4. 下载成功监听
下载完成后，下载管理会发出DownloadManager.ACTION_DOWNLOAD_COMPLETE这个广播，并传递downloadId作为参数。通过接受广播我们可以打开对下载完成的内容进行操作。代码如下：
```java
class CompleteReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		// get complete download id
		long completeDownloadId = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);
		// to do here
	}
};

private CompleteReceiver       completeReceiver;
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.download_manager_demo);

	…
	completeReceiver = new CompleteReceiver();
	/** register download success broadcast **/
	registerReceiver(completeReceiver,
					 new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE));
}

@Override
protected void onDestroy() {
	super.onDestroy();
	unregisterReceiver(completeReceiver);
}
```
5. 响应通知栏点击
(1) 响应下载中通知栏点击
点击下载中通知栏提示，系统会对下载的应用单独发送Action为DownloadManager.ACTION_NOTIFICATION_CLICKED广播。intent.getData为content://downloads/all_downloads/29669，最后一位为downloadId。
如果同时下载多个应用，intent会包含DownloadManager.EXTRA_NOTIFICATION_CLICK_DOWNLOAD_IDS这个key，表示下载的的downloadId数组。这里设计到下载管理通知栏的显示机制，会在下一篇具体介绍。

(2) 响应下载完成通知栏点击
下载完后会调用下面代码进行处理，从中我们可以发现系统会调用View action根据mimeType去查询。所以可以利用我们在介绍的DownloadManager.Request的setMimeType函数。
```java
private void openDownload(Context context, Cursor cursor) {
	String filename = cursor.getString(cursor.getColumnIndexOrThrow(Downloads.Impl._DATA));
	String mimetype = cursor.getString(cursor.getColumnIndexOrThrow(Downloads.Impl.COLUMN_MIME_TYPE));
	Uri path = Uri.parse(filename);
	// If there is no scheme, then it must be a file
	if (path.getScheme() == null) {
		path = Uri.fromFile(new File(filename));
	}
	Intent activityIntent = new Intent(Intent.ACTION_VIEW);
	mimetype = DownloadDrmHelper.getOriginalMimeType(context, filename, mimetype);
	activityIntent.setDataAndType(path, mimetype);
	activityIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	try {
		context.startActivity(activityIntent);
	} catch (ActivityNotFoundException ex) {
		Log.d(Constants.TAG, "no activity for " + mimetype, ex);
	}
}
```
如果界面上过多元素需要更新，且网速较快不断的执行onChange会对页面性能有一定影响。推荐ScheduledExecutorService定期查询，如下:
```java
	public static ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(3);
Runnable command = new Runnable() {

		@Override
		public void run() {
			updateView();
		}
	};
scheduledExecutorService.scheduleAtFixedRate(command, 0, 3, TimeUnit.SECONDS);
```
表示3秒定时刷新

Android下载管理DownloadManager功能扩展和bug修改
===

本文主要介绍如何修改Android系统下载管理，以支持更多的功能及部分bug修改和如何编译生效。目前内容包括暂停下载、继续下载、通知设置NotiExtra和NotiClass、wifi切换到3g自动暂停、Bug修改。
下面需要修改的DownloadManager.java所在目录为frameworks/base/core/java/android/app
DownloadInfo.java, DownloadProvider.java,DownloadThread.java文件所在目录为packages/providers/DownloadProvider/src/com/android/providers/downloads
 
1、暂停、继续下载功能
(1) DownloadProvider类修改
```java
public int update(final Uri uri, final ContentValues values, final String where, final String[] whereArgs)
```
函数，修改后代码如下(只增加了一行有效代码)：
    if (Binder.getCallingPid() != Process.myPid()) {
		filteredValues = new ContentValues();
		copyString(Downloads.Impl.COLUMN_APP_DATA, values, filteredValues);
		copyInteger(Downloads.Impl.COLUMN_VISIBILITY, values, filteredValues);
		Integer i = values.getAsInteger(Downloads.Impl.COLUMN_CONTROL);
		if (i != null) {
			filteredValues.put(Downloads.Impl.COLUMN_CONTROL, i);
			startService = true;
		}

		// trinea BEGIN, added by trinea@trinea.cn 2013/03/01
		copyInteger(Downloads.Impl.COLUMN_STATUS, values, filteredValues);
		// trinea END
		copyInteger(Downloads.Impl.COLUMN_CONTROL, values, filteredValues);
		copyString(Downloads.Impl.COLUMN_TITLE, values, filteredValues);
		copyString(Downloads.Impl.COLUMN_MEDIAPROVIDER_URI, values, filteredValues);
		copyString(Downloads.Impl.COLUMN_DESCRIPTION, values, filteredValues);
		copyInteger(Downloads.Impl.COLUMN_DELETED, values, filteredValues);
    } else {

其中以// trinea BEGIN开头，// trinea END结尾为修改部分，下面代码示例同样如此。因为DownloadProvider安全策略对非该进程id的修改会过滤掉COLUMN_STATUS状态，所以我们需要添加该行。

(2) DownloadThread类修改
	private void setupDestinationFile(State state, InnerState innerState)

函数中这个注释掉一个else if，如下：
	// trinea BEGIN, noted by trinea@trinea.cn 2013/03/01
	//} else if (mInfo.mETag == null && !mInfo.mNoIntegrity) {
	//    // This should've been caught upon failure
	//    if (Constants.LOGVV) {
	//        Log.d(TAG, "setupDestinationFile() unable to resume download, deleting "
	//                + state.mFilename);
	//    }
	//    f.delete();
	//    throw new StopRequestException(Downloads.Impl.STATUS_CANNOT_RESUME,
	//            "Trying to resume a download that can't be resumed");
	// trinea END
	
上面一段代码表示一个验证过程，可以去掉。
mETag为数据库中的etag字段值，代码中没有解释，感觉是一个验证值，类似hashcode。
mNoIntegrity为数据中no_integrity字段值，表示启动下载的应用程序能否验证下载的文件的完整性。不过坑爹的是对于etag和no_integrity都没有提供设置的接口
 
(3) DownloadManager类中添加暂停和继续的对外接口
/**
 * pause download, added by trinea@trinea.cn 2013/03/01
 * 
 * @param ids the IDs of the downloads to be paused
 * @return the number of downloads actually paused
 */
public int pauseDownload(long... ids) {
	if (ids == null || ids.length == 0) {
		// called with nothing to remove!
		throw new IllegalArgumentException("input param 'ids' can't be null");
	}

	ContentValues values = new ContentValues();
	values.put(Downloads.Impl.COLUMN_CONTROL, Downloads.Impl.CONTROL_PAUSED);
	values.put(Downloads.Impl.COLUMN_STATUS, Downloads.Impl.STATUS_PAUSED_BY_APP);
	if (ids.length == 1) {
		return mResolver.update(ContentUris.withAppendedId(mBaseUri, ids[0]), values,
				null, null);
	} 
	return mResolver.update(mBaseUri, values, getWhereClauseForIds(ids),
			getWhereArgsForIds(ids));
}

/**
 * resume download, added by trinea@trinea.cn 2013/03/01
 * 
 * @param ids the IDs of the downloads to be resumed
 * @return the number of downloads actually resumed
 */
public int resumeDownload(long... ids) {
	if (ids == null || ids.length == 0) {
		// called with nothing to remove!
		throw new IllegalArgumentException("input param 'ids' can't be null");
	}

	ContentValues values = new ContentValues();
	values.put(Downloads.Impl.COLUMN_CONTROL, Downloads.Impl.CONTROL_RUN);
	values.put(Downloads.Impl.COLUMN_STATUS, Downloads.Impl.STATUS_RUNNING);
	if (ids.length == 1) {
		return mResolver.update(ContentUris.withAppendedId(mBaseUri, ids[0]), values,
				null, null);
	} 
	return mResolver.update(mBaseUri, values, getWhereClauseForIds(ids),
			getWhereArgsForIds(ids));
}

无论是暂停还是继续我们都是同时把Downloads.Impl.COLUMN_CONTROL和Downloads.Impl.COLUMN_STATUS字段进行修改，因为在DownloadInfo的private boolean isReadyToStart(long now)函数中,会对COLUMN_CONTROL字段进行判断，如果是用户手动暂停的话，是不会自动继续的，部分代码如下：
```java
private boolean isReadyToStart(long now) {
	if (DownloadHandler.getInstance().hasDownloadInQueue(mId)) {
		// already running
		return false;
	}
	if (mControl == Downloads.Impl.CONTROL_PAUSED) {
		// the download is paused, so it's not going to start
		Xlog.i(Constants.DL_ENHANCE, "Download is paused " +
				"then no need to start");
		return false;
	}
	……
}
```

之后我们直接调用DownloadManager的pauseDownload和resumeDownload接口即可
 
PS：也可以试试不做第二步的修改，而将第一步DownloadProvider的update函数修改变为
	// trinea BEGIN, added by trinea@trinea.cn 2013/03/01
	copyInteger(Downloads.Impl.COLUMN_STATUS, values, filteredValues);
	copyInteger(Downloads.Impl.COLUMN_NO_INTEGRITY, values, filteredValues);
	// trinea END
	
第二步修改变为在public int resumeDownload(long… ids)加入
```java
values.put(Downloads.Impl.COLUMN_CONTROL, Downloads.Impl.CONTROL_RUN);
values.put(Downloads.Impl.COLUMN_STATUS, Downloads.Impl.STATUS_RUNNING);
values.put(Downloads.Impl.COLUMN_NO_INTEGRITY, true);
```

2、通知栏可以设置NotiExtra和NotiClass
(1) DownloadProvider类中修改private void checkInsertPermissions(ContentValues values)函数
```java
values.remove(Downloads.Impl.COLUMN_IS_VISIBLE_IN_DOWNLOADS_UI);
values.remove(Downloads.Impl.COLUMN_MEDIA_SCANNED);
// BEGIN, added by trinea@trinea.cn 2013/03/01
values.remove(Downloads.Impl.COLUMN_NOTIFICATION_CLASS);
values.remove(Downloads.Impl.COLUMN_NOTIFICATION_EXTRAS);
// trinea END
```
在DownloadProvider insert之前会调用checkInsertPermissions检查不能被插入的字段插入，这里我们需要允许这两个字段存在。
 
(2) DownloadManager.Request添加对外接口
```java
// BEGIN, added by trinea@trinea.cn 2013/03/01
private CharSequence mNotiClass;
private CharSequence mNotiExtras;
// trinea END

/**
 * Set notiClass, to be used as destination when click downloading in download manager ui
 * 
 * @return this object
 */
public Request setNotiClass(CharSequence notiClass) {
	mNotiClass = notiClass;
	return this;
}

/**
 * Set notiExtras, to be sended to notiClass when click downloading in download manager ui
 * 
 * @return this object
 */
public Request setNotiExtras(CharSequence notiExtras) {
	mNotiExtras = notiExtras;
	return this;
}

ContentValues toContentValues(String packageName)中
	putIfNonNull(values, Downloads.Impl.COLUMN_TITLE, mTitle);
	putIfNonNull(values, Downloads.Impl.COLUMN_DESCRIPTION, mDescription);
	putIfNonNull(values, Downloads.Impl.COLUMN_MIME_TYPE, mMimeType);
	// trinea BEGIN
	putIfNonNull(values, Downloads.Impl.COLUMN_NOTIFICATION_CLASS, mNotiClass);
	putIfNonNull(values, Downloads.Impl.COLUMN_NOTIFICATION_EXTRAS, mNotiExtras);
	// trinea END
```java
在Request中添加接口以及允许字段修改。通过允许设置NotiExtra和NotiClass，我们可以给系统传递更丰富的参数，在通知栏点击相应或是DownloadUi中通过broadcast将这些参数传递出来方便应用调用。

3、wifi切换到3g自动暂停
(1) 修改DownloadInfo.java
```java
private int checkIsNetworkTypeAllowed(int networkType) {
	if (mIsPublicApi) {
		final int flag = translateNetworkTypeToApiFlag(networkType);
		final boolean allowAllNetworkTypes = mAllowedNetworkTypes == ~0;
		if (!allowAllNetworkTypes && (flag & mAllowedNetworkTypes) == 0) {
			return NETWORK_TYPE_DISALLOWED_BY_REQUESTOR;
		}
		// trinea BEGIN
		if (mStatus == Downloads.Impl.STATUS_WAITING_FOR_NETWORK 
				&& flag != DownloadManager.Request.NETWORK_WIFI) {
			return NETWORK_TYPE_DISALLOWED_BY_REQUESTOR;
		}
		// trinea END
	}
	return checkSizeAllowedForNetwork(networkType);
}
```
表示等待网络时始终只等待wifi
 
(2) 修改DownloadReceiver.java
```java
} else if (action.equals(ConnectivityManager.CONNECTIVITY_ACTION)) {
	NetworkInfo info = (NetworkInfo)
			intent.getParcelableExtra(ConnectivityManager.EXTRA_NETWORK_INFO);
	if (info != null && info.isConnected()) {
		startService(context);
	}
} else if (action.equals(Constants.ACTION_RETRY)) {
```
修改为：
```java
} else if (action.equals(ConnectivityManager.CONNECTIVITY_ACTION)) {
	NetworkInfo info = (NetworkInfo)
			intent.getParcelableExtra(ConnectivityManager.EXTRA_NETWORK_INFO);
	// trinea BEGIN
	/** 
	 * modified by trinea@trinea.cn @2013/04/01, resume download only when network type is wifi
	 */
	if (info != null && info.isConnected() && info.getType() == ConnectivityManager.TYPE_WIFI) {
		// trinea END
		startService(context);
	}
} else if (action.equals(Constants.ACTION_RETRY)) {
```
表示只有连接wifi时才唤醒service去检查是否下载

(3) 修改DownloadThread.java
```java
/**
 * Check if the download has been paused or canceled, stopping the request appropriately if it
 * has been.
 */
private void checkPausedOrCanceled(State state) throws StopRequestException {
	synchronized (mInfo) {
		if (mInfo.mControl == Downloads.Impl.CONTROL_PAUSED) {
			Xlog.i(Constants.DL_ENHANCE, "DownloadThread: checkPausedOrCanceled: user pause download");
			throw new StopRequestException(
					Downloads.Impl.STATUS_PAUSED_BY_APP, "download paused by owner");
		}
		if (mInfo.mStatus == Downloads.Impl.STATUS_CANCELED) {
			throw new StopRequestException(Downloads.Impl.STATUS_CANCELED, "download canceled");
		}
	}

	// if policy has been changed, trigger connectivity check
	if (mPolicyDirty) {
		// trinea BEGIN
		/** 
		 * add by trinea@trinea.cn @2013/04/01, when switched from wifi to 3g, pause all download
		 */
		NetworkInfo info = mSystemFacade.getActiveNetworkInfo(mInfo.mUid);
		if (info != null && !info.isConnected()) {
			mInfo.mStatus = Downloads.Impl.STATUS_WAITING_FOR_NETWORK;
		}
		// trinea END
		checkConnectivity();
	}
}
```
表示如果网络变化并且表示网络断开时，下载状态变为等待网络。

4、Bug修改
(1) 当存储空间不足时，利用DownloadManager下载，只显示通知栏提示，在下载管理UI中不显示
DownloadManager的Cursor runQuery(ContentResolver resolver, String[] projection, Uri baseUri)函数修改如下：
```java
if ((mStatusFlags & STATUS_RUNNING) != 0) {
	parts.add(statusClause("=", Downloads.Impl.STATUS_RUNNING));
	// trinea BEGIN
	parts.add(statusClause("=", Downloads.Impl.STATUS_INSUFFICIENT_SPACE_ERROR));
	// trinea END
}
```
DownloadManager的CursorTranslator类的private int translateStatus(int status) 函数修改如下：
```java
// trinea BEGIN
case Downloads.Impl.STATUS_INSUFFICIENT_SPACE_ERROR:
// trinea END
case Downloads.Impl.STATUS_RUNNING:
	return STATUS_RUNNING;
```

5、编译安装
修改后是需要重新编译的，需同时编译framweork和DownloadProvider。
framework编译命令为：./makeMtk model mm frameworks/base/core/
编译后apk所在路径为out\target\product\model\system\framework\secondary_framework.jar，之后push到system/framework重启即可。编译命令中model为机型，非mtk平台命令有所不同
 
DownloadProvider编译命令为./makeMtk model mm packages/providers/DownloadProvider/
编译后apk所在路径为out\target\product\model\system\app\DownloadProvider.apk，之后push到system/app即可(可能需要先删除/system/app/目录下的DownloadProvider.odex)