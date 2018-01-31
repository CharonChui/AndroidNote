Android四大组件之ContentProvider
===

ContentProvider    
---

安卓应用程序默认是无法获取到其他程序的数据，这是安卓安全学的基石(沙盒原理)。但是经常我们需要给其他应用分享数据，内容提供者就是一个这种可以分享数据给其他应用的接口。
可以简单的理解为，内容提供者就是一个可以在不同应用程序间共享数据的组件，相当于一个中间人，一个程序把数据暴露给这个中间人，另一个则通过这个中间人获取相应的数据.        

下面的这张图片能更直观的显示:     

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/ContentProvider_1.png)    

- `ContentProvider`中的`getContext`和`AndroidTestCast`中的`getContext`方法一样，都是一个模拟的上下文，必须在该类初始化之后才会调用`setContext`方法将`context`设置成自己的成员变量中记录，
	所以对于获取`getContext`的时候只能放在方法内，不能放到成员位置，因为在成员上时是null，而在方法内调用时该类就会已经初始化完了      

- `ContentProvider`中的`query()`后不能关闭数据库，因为其他的应用在调用该`query`方法时需要继续使用该返回值`Cursor`，所以不能关闭数据库，因为数据库关闭之后`Cursor`就不能用了，
	`Cursor`中保存的数据其实是数据库的一个引用，如果数据库关了`Cursor`就不能找到里面的数据了，`Cursor.close()`只是释放`Cursor`用到的资源。说到这里就多数一句
	`According to Dianne Hackborn (Android framework engineer) there is no need to close the database in a content provider.`以为内容提供者是因为进程启动时便加载，之后就一直存在，当进程销毁
	释放资源时会去关闭数据库。
	
- 如果数据是`SQLiteDatabase`，表中必须有一个`_id`的列，用来表示每条记录的唯一性。

1. 继承`ContentProvider`,并实现相应的方法。
	```java
	public class NoteProvider extends ContentProvider {
		private static final int NOTES = 1;
		private static final int NOTE_ID = 2;

		public static final String AUTHORITY = "com.charon.demo.provider.noteprovider";
		public static final String TABLE_NAME = "note";
		// 定义一个名为`CONTENT_URI`必须为其指定一个唯一的字符串值，最好的方案是以类的全名称
		public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/" + TABLE_NAME);

		// 声明一个路径的检查者，参数为Uri不匹配时的返回值
		// 虽然是中间人，但也不能谁要数据我们都给，所以要检查下，只有符合我们要求的人，我们才会给他数据。
		private static UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

		private NoteSQLiteOpenHelper mSQLiteOpenHelper;
		private SQLiteDatabase mSQLiteDatabase;

		static {
			// 建立匹配规则，例如发现路径为ccom.charon.demo.provider.noteprovider/note/1表示要操作note表中id为1的记录
			sUriMatcher.addURI(AUTHORITY, TABLE_NAME, NOTES);
			sUriMatcher.addURI(AUTHORITY, TABLE_NAME + "/#", NOTE_ID);
		}

		/**
		 * Implement this to initialize your content provider on startup.
		 * This method is called for all registered content providers on the
		 * application main thread at application launch time.  It must not perform
		 * lengthy operations, or application startup will be delayed.
		 * <p/>
		 * <p>You should defer nontrivial initialization (such as opening,
		 * upgrading, and scanning databases) until the content provider is used
		 * (via {@link #query}, {@link #insert}, etc).  Deferred initialization
		 * keeps application startup fast, avoids unnecessary work if the provider
		 * turns out not to be needed, and stops database errors (such as a full
		 * disk) from halting application launch.
		 * <p/>
		 * <p>If you use SQLite, {@link android.database.sqlite.SQLiteOpenHelper}
		 * is a helpful utility class that makes it easy to manage databases,
		 * and will automatically defer opening until first use.  If you do use
		 * SQLiteOpenHelper, make sure to avoid calling
		 * {@link android.database.sqlite.SQLiteOpenHelper#getReadableDatabase} or
		 * {@link android.database.sqlite.SQLiteOpenHelper#getWritableDatabase}
		 * from this method.  (Instead, override
		 * {@link android.database.sqlite.SQLiteOpenHelper#onOpen} to initialize the
		 * database when it is first opened.)
		 *
		 * @return true if the provider was successfully loaded, false otherwise
		 */
		@Override
		public boolean onCreate() {
			mSQLiteOpenHelper = new NoteSQLiteOpenHelper(getContext());
			return true;
		}

		/**
		 * 内容提供者暴露的查询的方法.
		 */
		@Override
		public Cursor query(Uri uri, String[] projection, String selection,
							String[] selectionArgs, String sortOrder) {
			mSQLiteDatabase = mSQLiteOpenHelper.getReadableDatabase();
			Cursor cursor;
			// 1.重要的事情 ,检查 uri的路径.
			switch (sUriMatcher.match(uri)) {
				case NOTES:
					break;
				case NOTE_ID:
					String id = uri.getLastPathSegment();
					if (TextUtils.isEmpty(selection)) {
						selection = selection + "_id = " + id;
					} else {
						selection = selection + " and " + "_id = " + id;
					}
					break;
				default:
					throw new IllegalArgumentException("UnKnown Uri" + uri);
					break;
			}
			cursor = mSQLiteDatabase.query(TABLE_NAME, projection, selection, selectionArgs, null, null, sortOrder);
			if (cursor != null) {
				cursor.setNotificationUri(getContext().getContentResolver(), uri);
			}
			return cursor;
		}

		/**
		 * Implement this to handle requests for the MIME type of the data at the
		 * given URI.  The returned MIME type should start with
		 * <code>vnd.android.cursor.item</code> for a single record,
		 * or <code>vnd.android.cursor.dir/</code> for multiple items.
		 * This method can be called from multiple threads, as described in
		 * <a href="{@docRoot}guide/topics/fundamentals/processes-and-threads.html#Threads">Processes
		 * and Threads</a>.
		 * <p/>
		 * <p>Note that there are no permissions needed for an application to
		 * access this information; if your content provider requires read and/or
		 * write permissions, or is not exported, all applications can still call
		 * this method regardless of their access permissions.  This allows them
		 * to retrieve the MIME type for a URI when dispatching intents.
		 *
		 * @param uri the URI to query.
		 * @return a MIME type string, or {@code null} if there is no type.
		 */
		@Override
		public String getType(Uri uri) {
			// 注释说的很清楚了，下面是常用的格式
			// 单个记录的IMEI类型 vnd.android.cursor.item/vnd.<yourcompanyname>.<contenttype>
			// 多个记录的IMEI类型 vnd.android.cursor.dir/vnd.<yourcompanyname>.<contenttype>
			switch (sUriMatcher.match(uri)) {
				case NOTE_ID:
					// 如果uri为 content://com.charon.demo.noteprovider/note/1
					return "vnd.android.cursor.item/vnd.charon.note";
				case NOTES:
					return "vnd.android.cursor.dir/vnd.charon.note";
				default:
					return null;
			}

			// 这个MIME类型的作用是要匹配AndroidManifest.xml文件<activity>标签下<intent-filter>标签的子标签<data>的属性android:mimeType。
			// 如果不一致，则会导致对应的Activity无法启动。
		}

		@Override
		public Uri insert(Uri uri, ContentValues values) {
			mSQLiteDatabase = mSQLiteOpenHelper.getWritableDatabase();
			switch (sUriMatcher.match(uri)) {
				case NOTES:
					break;

				case NOTE_ID:
					break;

				default:
					throw new IllegalArgumentException("UnKnown Uri" + uri);
					break;
			}

			long rowId = mSQLiteDatabase.insert(TABLE_NAME, null, values);
			if (rowId > 0) {
				Uri noteUri = ContentUris.withAppendedId(CONTENT_URI, rowId);
				getContext().getContentResolver().notifyChange(noteUri, null);
				return noteUri;
			}

			return null;
		}

		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			mSQLiteDatabase = mSQLiteOpenHelper.getWritableDatabase();
			switch (sUriMatcher.match(uri)) {
				case NOTES:
					break;

				case NOTE_ID:
					String id = uri.getLastPathSegment();
					if (TextUtils.isEmpty(selection)) {
						selection = selection + "_id = " + id;
					} else {
						selection = selection + " and " + "_id = " + id;
					}
					break;

				default:
					throw new IllegalArgumentException("UnKnown Uri" + uri);
					break;
			}
			int count = mSQLiteDatabase.delete(TABLE_NAME, selection, selectionArgs);
			getContext().getContentResolver().notifyChange(uri, null);
			return count;
		}

		@Override
		public int update(Uri uri, ContentValues values, String selection,
						  String[] selectionArgs) {
			switch (sUriMatcher.match(uri)) {
				case NOTES:

					break;

				case NOTE_ID:

					break;

				default:
					throw new IllegalArgumentException("UnKnown Uri" + uri);
					break;
			}

			mSQLiteDatabase = mSQLiteOpenHelper.getWritableDatabase();
			int update = mSQLiteDatabase.update(TABLE_NAME, values, selection, selectionArgs);
			return update;
		}
	```

2. 在清单文件中进行注册，并且指定其authorities
	```xml
	<provider
		android:name="com.charon.demo.provider.NoteProvider"
		android:authorities="com.charon.demo.provider.noteprovider" >
	```
	
3. 使用内容提供者获取数据，使用`ContentResolver`去操作`ContentProvider`, `ContentResolver`用于管理`ContentProvider`实例，
	并且可实现找到指定的`ContentProvider`并获取里面的数据
	```java
	public void query(View view){
		//得到内容提供者的解析器  中间人
		ContentResolver resolver = getContentResolver();
		Cursor cursor = resolver.query(NoteProvider.CONTENT_URI, null, null, null, null);
		while(cursor.moveToNext()){
			String name = cursor.getString(cursor.getColumnIndex("name"));
			int id = cursor.getInt(cursor.getColumnIndex("id"));
			float money = cursor.getFloat(cursor.getColumnIndex("money"));
			System.out.println("id="+id+",name="+name+",money="+money);
		}
		cursor.close();
	}
	public void insert(View view){
		ContentResolver resolver = getContentResolver();
		ContentValues values = new ContentValues();
		values.put("name", "买洗头膏");
		values.put("money", 22.58f);
		resolver.insert(NoteProvider.CONTENT_URI, values);
	}
	public void update(View view){
		ContentResolver resolver = getContentResolver();
		ContentValues values = new ContentValues();
		values.put("name", "买洗头膏");
		values.put("money", 42.58f);
		resolver.update(NoteProvider.CONTENT_URI, values, "name=?", new String[]{"买洗头膏"});
	}
	public void delete(View view){
		ContentResolver resolver = getContentResolver();
		resolver.delete(NoteProvider.CONTENT_URI, "name=?", new String[]{"买洗头膏"});
	}
	```

内容观察者     
---

内容观察者的原理：     
`How a content provider actually stores its data under the covers is up to its designer. But all content providers implement a common interface for 
querying the provider and returning results — as well as for adding, altering, and deleting data.            
It's an interface that clients use indirectly, most generally through ContentResolver objects. 
You get a ContentResolver by calling getContentResolver() from within the implementation of an Activity or other application component: 
You can then use the ContentResolver's methods to interact with whatever content providers you're interested in.`

1. 一方使用内容观察者去观察变化
	```java
		protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		ContentResolver resolver = getContentResolver();
		resolver.registerContentObserver(NoteProvider.CONTENT_URI, true, new NoteObserver(new Handler()));

	}

	private class NoteObserver extends ContentObserver {

		public NoteObserver(Handler handler) {
			super(handler);
			
		}
		//当观察到数据发生变化的时候  会执行onchange方法.
		@Override
		public void onChange(boolean selfChange) {
			super.onChange(selfChange);
			Log.i(TAG,"发现有新的短信产生了...");
			//1.利用内容提供者  中间人 获取用户的短信数据.
			ContentResolver resolver  = getContentResolver();
			// .. 重新查询
			cursor = ...;
			cursor.close();
		}
	}
	```
	
2. 一方在发生变化的时候去发送改变的消息
	对于一些系统的内容提供者内部都实现了该步骤，如果是自己写程序想要暴露就必须要加上该代码。     
	
	```java	
	getContext().getContentResolver().notifyChange(uri, null);
	```
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 