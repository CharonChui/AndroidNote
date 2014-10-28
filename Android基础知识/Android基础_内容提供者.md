Android基础_内容提供者
===

1. ContentProvider    
    一个可以在不同应用程序间共享数据的组件，相当于一个中间人，一边把数据暴露给这个中间人，另一边是别的应用通过这个中间人获取相应的数据.
    **ContentProvider中的getContext和AndroidTestCast中的getContext方法一样，也是必须这个类初始化之后才会调用setContext方法将context设置成自己的成员变量中记录，所以对于获取getContext的时候只能放在方法内，不能放到成员位置，因为在成员上时是null，而只能在方法内调用，在方法内调用时该类就会已经初始化完了，context就不会为null还有一个就是在ContentProvider中的query是不能关闭数据库，因为其他的应用在调用这个ContentResolver的query方法的时候会返回这个Cursor要继续对这个Cursor进行使用，所以不能关闭数据库，不然再用Cusor的时候就会报错，因为数据库关闭之后，Cursor也不能用了，Cursor中保存的数据其实是数据库的一个引用，如果数据库关了Cursor就不能找到里面的数据了，Cursor也有关闭的方法，只是释放Cursor用到的资源。**
    
    1. 继承ContentProvider,并实现相应的方法，由于是一个中间人，我们要检查这个中间人是不是冒牌的，所以就要去验证URI
        ```java
    	public class NoteInfoProvider extends ContentProvider {
    		private static final int QUERY = 1;
    		private static final int INSERT = 2;
    		private static final int DELETE = 3;
    		private static final int UPDATE = 4;
    
    		// 参数code 代表如果uri不匹配的返回值
    		// 在当前应用程序的内部 声明一个路径的检查者
    		private static UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
    		private SQLiteDatabase db;
    
    		static {
    			// 建立一个匹配规则
    			// 1.如果发现一个路径 com.itheima.note.noteprovider/query 查询的操作
    			matcher.addURI("com.itheima.note.noteprovider", "query", QUERY);
    			matcher.addURI("com.itheima.note.noteprovider", "insert", INSERT);
    			matcher.addURI("com.itheima.note.noteprovider", "delete", DELETE);
    			matcher.addURI("com.itheima.note.noteprovider", "update", UPDATE);
    		}
    
    		@Override
    		public boolean onCreate() {
    			NoteSQLiteOpenHelper helper = new NoteSQLiteOpenHelper(getContext());
    			db = helper.getWritableDatabase();
    			return false;
    		}
    
    		/**
    		 * 内容提供者暴露的查询的方法.
    		 */
    		@Override
    		public Cursor query(Uri uri, String[] projection, String selection,
    				String[] selectionArgs, String sortOrder) {
    			// 1.重要的事情 ,检查 uri的路径.
    			int code = matcher.match(uri);
    			if (code == QUERY) {
    				// 查询处理
    				NoteSQLiteOpenHelper helper = new NoteSQLiteOpenHelper(getContext());
    				SQLiteDatabase db = helper.getReadableDatabase();
    				Cursor cursor = db.rawQuery("select * from account", null);
    				return cursor;
    			} else {
    				throw new IllegalArgumentException("路径不能被识别,我不认识你...");
    			}
    		}
    
    		@Override
    		public String getType(Uri uri) {
    			int code = matcher.match(uri);
    			if (code == QUERY) {
    				//返回一条数据
    				return "vnd.android.cursor.item/note";
    				//返回多条数据
    				//return "vnd.android.cursor.dir/note"
    			}
    			return null;
    		}
    
    		@Override
    		public Uri insert(Uri uri, ContentValues values) {
    			int code = matcher.match(uri);
    			if (code == INSERT) {
    				db.insert("account", null, values);
    			} else {
    				throw new IllegalArgumentException("路径不能被识别,我不认识你...");
    			}
    			return null;
    		}
    
    		@Override
    		public int delete(Uri uri, String selection, String[] selectionArgs) {
    			int code = matcher.match(uri);
    			if (code == DELETE) {
    				db.delete("account", selection, selectionArgs);
    			} else {
    				throw new IllegalArgumentException("路径不能被识别,我不认识你...");
    			}
    			return 0;
    		}
    
    		@Override
    		public int update(Uri uri, ContentValues values, String selection,
    				String[] selectionArgs) {
    			int code = matcher.match(uri);
    			if (code == UPDATE) {
    				db.update("account", values, selection, selectionArgs);
    			} else {
    				throw new IllegalArgumentException("路径不能被识别,我不认识你...");
    			}
    			return 0;
    		}
    	}
    	```

	2. 在清单文件中进行注册，并且指定其authorities
    	```xml
    	<provider
    		android:name="com.itheima.note.provider.NoteInfoProvider"
    		android:authorities="com.itheima.note.noteprovider" >
    	```
		
	3. 使用内容提供者获取数据，使用`ContentResolver`去操作`ContentProvider`, `ContentResolver`用于管理`ContentProvider`实例，并且可实现找到指定的`ContentProvider`并获取里面的数据
    	```java
    	public void query(View view){
    		//得到内容提供者的解析器  中间人
    		ContentResolver resolver = getContentResolver();
    		Uri uri = Uri.parse("content://com.itheima.note.noteprovider/queryaa");
    		Cursor cursor = resolver.query(uri, null, null, null, null);
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
    		Uri uri = Uri.parse("content://com.itheima.note.noteprovider/insert");
    		ContentValues values = new ContentValues();
    		values.put("name", "买洗头膏");
    		values.put("money", 22.58f);
    		resolver.insert(uri, values);
    	}
    	public void update(View view){
    		ContentResolver resolver = getContentResolver();
    		Uri uri = Uri.parse("content://com.itheima.note.noteprovider/update");
    		ContentValues values = new ContentValues();
    		values.put("name", "买洗头膏");
    		values.put("money", 42.58f);
    		resolver.update(uri, values, "name=?", new String[]{"买洗头膏"});
    	}
    	public void delete(View view){
    		ContentResolver resolver = getContentResolver();
    		Uri uri = Uri.parse("content://com.itheima.note.noteprovider/delete");
    		resolver.delete(uri, "name=?", new String[]{"买洗头膏"});
    	}
    	```

2. 内容观察者     
    内容观察者的原理：在一个应用程序中是无法直接观察另外一个程序中数据变化的，他们之间的通信是通过一块公共的内存(消息邮箱)，在这个应用的数据发生变化的时候，它会发送一个消息到这块公共的内容中，而另外一个应用会不断的去观察这块公共内存中的消息，这样就能实现观察者的功能。      
	How a content provider actually stores its data under the covers is up to its designer. But all content providers implement a common interface for querying the provider and returning results — as well as for adding, altering, and deleting data.      
    It's an interface that clients use indirectly(间接地), most generally through ContentResolver objects. You get a ContentResolver by calling getContentResolver() from within the implementation of an Activity or other application component: You can then use the ContentResolver's methods to interact with whatever content providers you're interested in.

	1. 一方使用内容观察者去观察变化
		```java
			protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			// 监听系统的短信数据库的变化

			ContentResolver resolver = getContentResolver();
			Uri uri = Uri.parse("content://sms/");
			resolver.registerContentObserver(uri, true, new SmsObserver(new Handler()));

		}

		private class SmsObserver extends ContentObserver {

			public SmsObserver(Handler handler) {
				super(handler);
				
			}
			//当观察到数据发生变化的时候  会执行onchange方法.
			@Override
			public void onChange(boolean selfChange) {
				Log.i(TAG,"发现有新的短信产生了...");
				//1.利用内容提供者  中间人 获取用户的短信数据.
				ContentResolver resolver  = getContentResolver();
				Uri uri = Uri.parse("content://sms/"); //根据分析 代表的是所有的短信的路径
				Cursor cursor = resolver.query(uri, new String[]{"address","date","body","type"}, null, null, null);
				if(cursor.moveToFirst()){
					String address =cursor.getString(0);
					String date  = cursor.getString(1);
					String body = cursor.getString(2);
					String type = cursor.getString(3);
					System.out.println(address+"--"+date+"---"+body+"---"+type);
				}
				cursor.close();
				super.onChange(selfChange);
			}
		}
		```
	2. 一方在发生变化的时候去发送改变的消息
	    对于一些系统的内容提供者内部都实现了该步骤，如果是自己写程序想要暴露就必须要加上该代码。     
		
		```java	
        mContext.getContentResolver().notifyChange(uri, null);
		```
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 