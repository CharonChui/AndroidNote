day02Android数据存储
===

1. Android数据存储的几种形式
    1. File
    
        - 内部存储    
            通过`mContext.getFilesDir()`来得到`data/data/包名/Fil`e目录
        - SD卡
            ```java
            TextView tv = (TextView) findViewById(R.id.tv_sdsize);
    		File path = Environment.getExternalStorageDirectory();
    		StatFs stat = new StatFs(path.getPath());
    		long blockSize = stat.getBlockSize();
    		long availableBlocks = stat.getAvailableBlocks();
    		long sizeAvailSize = blockSize * availableBlocks;
    		String str = Formatter.formatFileSize(this, sizeAvailSize);
    		tv.setText(str);
            ```
    2. SharedPreferences    
        会在`data/data/包名/shared_prefes`里面去创建相应的xml文件，根节点是Map，其实内部就是将数据保存到Map集合中，然后将该集合中的数据写到xml文件中进行保存。
        ```xml
        <map>
            <string name="password">123</string>
            <boolean name="isLogin" value="false"/>
        </map>
        ```
        ```java
		//获取系统的一个sharedpreference文件  名字叫 sp
		SharedPreferences sp = context.getSharedPreferences("sp", Context.MODE_WORLD_READABLE+Context.MODE_WORLD_WRITEABLE);
		//创建一个编辑器 可以编辑 sp
		Editor editor = sp.edit();
		editor.putString("name", name);
		editor.putString("password", password);
		editor.putBoolean("boolean", false);
		editor.putInt("int", 8888);
		editor.putFloat("float",3.14159f);
		//注意:调用 commit 提交 数据到文件.
		editor.commit();
		//editor.clear();
	
        ```
    3. SQLite


	
	
	    1.创建数据库
			1. 创建一个数据库打开的帮助类继承SQLiteOpenHelper
			2. 在构造方法中设置数据库文件的名称、游标工厂为null、数据库的版本 1
			3. 重写onCreate()方法，进行数据库表结构的初始化，数据库第一次被创建的时候会调用的方法
			4. helper.getReadabledatabase()或者调用helper.getWriteabledatabase()获取数据库的示例只有在调用这两个方法的时候如果数据库不存在才会去创建该数据库。
			**注意：SQLite数据库中列一旦创建不能修改，如果一定要修改，需要重新创建表，拷贝数据**
			```java
			public class NoteSQLiteOpenHelper extends SQLiteOpenHelper {

				private static final String TAG = "NoteSQLiteOpenHelper";
				/**
				 * context 上下文 name 数据库的名称 cursorfactory 游标工厂 一般设置null 默认游标工厂 version 数据库的版本
				 * 版本号从1开始的
				 */
				public NoteSQLiteOpenHelper(Context context) {
					super(context, "note.db", null, 3);
				}

				/**
				 * oncreate 方法 会在数据库第一创建的时候的是被调用 适合做数据库表结构的初始化
				 */
				@Override
				public void onCreate(SQLiteDatabase db) {
					db.execSQL("create table account (id integer primary key autoincrement , name  varchar(20), money varchar(20) )");
				}
				
				@Override
				public void onUpgrade(SQLiteDatabase db, int arg1, int arg2) {
					Log.i(TAG,"onupdate 方法被调用了 ,在这个方法里面做更新数据库表结构的操作");
					db.execSQL("DROP TABLE IF EXISTS " + "account");
					this.onCreate(db);
				}
			}
			```
		
		2. 增删改查的第一种方式
			1. 增
			insert into account (name,money) values ('买饭','18')
			2. 查
			select * from account 
			3. 改
			update account set money ='19' where id='1'
			4. 删
			delete from account where id='1'
			5. 分页查询
			SELECT * FROM person LIMIT 20 OFFSET 10
			SELECT * FROM person LIMIT 10,20
			
		    ```java
			/**
			 * 记账本的dao
			 * 
			 * @author Administrator
			 * 
			 */
			public class NoteDao {
				// 因为 任何一个操作都是需要 得到 NoteSQLiteOpenHelper helper
				// 把他放在构造方法里面初始化
				private NoteSQLiteOpenHelper helper;

				public NoteDao(Context context) {
					helper = new NoteSQLiteOpenHelper(context);
				}

				/**
				 * 添加一条账目信息 到数据库
				 * 
				 * @param name
				 *            花销的名称
				 * @param money
				 *            金额
				 */
				public void add(String name, float money) {
					SQLiteDatabase db = helper.getWritableDatabase();
					db.execSQL("insert into account (name,money) values (?,?)",
							new Object[] { name, money });
					// 记住 关闭.
					db.close();
				}

				public void delete(int id) {
					SQLiteDatabase db = helper.getWritableDatabase();
					db.execSQL("delete from account where id=?", new Object[] { id });
					db.close();
				}

				public void update(int id, float newmoney) {
					SQLiteDatabase db = helper.getWritableDatabase();
					db.execSQL("update account set money =? where id=?", new Object[] {
							newmoney, id });
					db.close();
				}

				/**
				 * 返回数据库所有的条目
				 * 
				 * @return
				 */
				public List<NoteBean> findAll() {
					// 得到可读的数据库
					SQLiteDatabase db = helper.getReadableDatabase();
					List<NoteBean> noteBeans = new ArrayList<NoteBean>();
					// 获取到数据库查询的结果游标
					Cursor cursor = db.rawQuery("select id,money,name from account ", null);
					while (cursor.moveToNext()) {
						int id = cursor.getInt(cursor.getColumnIndex("id"));
						String name = cursor.getString(cursor.getColumnIndex("name"));
						float money = cursor.getFloat(cursor.getColumnIndex("money"));
						NoteBean bean = new NoteBean(id, money, name);
						noteBeans.add(bean);
						bean = null;
					}

					db.close();
					return noteBeans;
				}

				/**
				 * 模拟一个转账的操作. 使用数据库的事务
				 * 
				 * @throws Exception
				 */
				public void testTransaction() throws Exception {
					// 得到可写的数据库
					SQLiteDatabase db = helper.getWritableDatabase();
					db.beginTransaction(); // 开始事务
					try {
						db.execSQL("update account set money = money - 5 where id=? ",
								new String[] { "2" });
						db.execSQL("update account set money = money + 5 where id=? ",
								new String[] { "3" });
						db.setTransactionSuccessful();
					} catch (Exception e) {
						// TODO: handle exception
					} finally {
						db.endTransaction();//关闭事务.
						db.close();
					}
				}
			}
			```
			
		3. 增删改查第二种方式
		    ```java
			public class NoteDao2 {
				private NoteSQLiteOpenHelper helper;

				public NoteDao2(Context context) {
					helper = new NoteSQLiteOpenHelper(context);
				}

				/**
				 * 添加一条账目信息 到数据库
				 * 
				 * @param name
				 *            花销的名称
				 * @param money
				 *            金额
				 * 
				 * @return true 插入成功 false 失败
				 */
				public boolean add(String name, float money) {
					SQLiteDatabase db = helper.getWritableDatabase();
					ContentValues values = new ContentValues();
					values.put("name", name);
					values.put("money", money);
					long rawid = db.insert("account", null, values);
					db.close();
					if (rawid > 0) {
						return true;
					} else {
						return false;
					}
				}

				public boolean delete(int id) {
					SQLiteDatabase db = helper.getWritableDatabase();
					int result = db.delete("account", "id=?", new String[] { id + "" });
					db.close();
					if (result > 0) {
						return true;
					} else {
						return false;
					}
				}

				public boolean update(int id, float newmoney) {
					SQLiteDatabase db = helper.getWritableDatabase();
					ContentValues values = new ContentValues();
					values.put("id", id);
					values.put("money", newmoney);
					int result = db.update("account", values, "id=?", new String[] { id
							+ "" });
					db.close();
					if (result > 0) {
						return true;
					} else {
						return false;
					}
				}

				/**
				 * 返回数据库所有的条目
				 * 
				 * @return
				 */
				public List<NoteBean> findAll() {
					// 得到可读的数据库
					SQLiteDatabase db = helper.getReadableDatabase();
					List<NoteBean> noteBeans = new ArrayList<NoteBean>();
					Cursor cursor = db.query("account", new String[] { "id", "name",
							"money" }, null, null, null, null, null);
					while (cursor.moveToNext()) {
						int id = cursor.getInt(cursor.getColumnIndex("id"));
						String name = cursor.getString(cursor.getColumnIndex("name"));
						float money = cursor.getFloat(cursor.getColumnIndex("money"));
						NoteBean bean = new NoteBean(id, money, name);
						noteBeans.add(bean);
						bean = null;
					}
					db.close();
					return noteBeans;
				}
			}
			```
    4. ContentProvider
    5. Network

2. 清除缓存&清除数据    
    清除数据会清除/data/data/包名中的所有文件     
    getCacheDir()  /data/data/<当前应用程序包名>/cache/，清除缓存的时候会清除该目录中的内容---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
