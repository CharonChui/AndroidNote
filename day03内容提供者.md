day02Android数据存储
===

1. ContentProvider
    一个可以在不同应用程序间共享数据的组件，相当于一个中间人，一边把数据暴露给这个中间人，另一边是别的应用通过这个中间人获取相应的数据。
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
	3. 在另外一个应用程序中使用内容提供者获取数据,使用ContentResolver去操作ContentProvider,ContentResolver用于管理ContentProvider实例，并且可实现找到指定的Contentprovider并获取到Contentprovider的数据。
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
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 