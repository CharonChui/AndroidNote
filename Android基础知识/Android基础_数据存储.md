Android基础_数据存储
===

1. Android数据存储的几种形式
    1. Internal Storage    
        Store private data on the device memory.    
        通过`mContext.getFilesDir()`来得到`data/data/包名/File`目录
    2. External Storage    
        Store public data on the shared external storage.   
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
    3. SharedPreferences     
        Store private primitive data in key-value pairs.     
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
    4. SQLiteDatabase    
        Store structured data in a private database.
		Android平台中嵌入了一个关系型数据库SQLite，和其他数据库不同的是SQLite存储数据时不区分类型，例如一个字段声明为Integer类型，我们也可以将一个字符串存入，一个字段声明为布尔型，我们也可以存入浮点数。除非是主键被定义为Integer，这时只能存储64位整数创建数据库的表时可以不指定数据类型，例如：     
        `CREATE TABLE person(id INTEGER PRIMARY KEY AUTOINCREMENT, name VARCHAR(20))`    
        `CREATE TABLE person(id INTEGER PRIMARY KEY AUTOINCREMENT, name)`     
        SQLite支持大部分标准SQL语句，增删改查语句都是通用的，分页查询语句和MySQL相同     
        `SELECT * FROM person LIMIT 20 OFFSET 10`        
        `SELECT * FROM person LIMIT 10,20`    
        `select * from reading_history order by _id desc limit 3, 4;`    
        `delete from test where _id in (select _id from test ORDER BY _id DESC limit 3, 20)`
		1. 继承SQLiteOpenHelper
		    ```java
			public class NoteSQLiteOpenHelper extends SQLiteOpenHelper {

				private static final String TAG = "NoteSQLiteOpenHelper";
				/**
				 * context 上下文 name 数据库的名称 cursorfactory 游标工厂 一般设置null 默认游标工厂 version 数据库的版本
				 * 版本号从1开始的
				 * 
				 * @param context
				 */
				public NoteSQLiteOpenHelper(Context context) {
					super(context, "note.db", null, 1);
				}

				/**
				 * oncreate 方法 会在数据库第一创建的时候的是被调用 适合做数据库表结构的初始化
				 */
				@Override
				public void onCreate(SQLiteDatabase db) {
					Log.i(TAG, "oncreate 方法被调用了...");
					db.execSQL("create table account (id integer primary key autoincrement , name  varchar(20), money varchar(20) )");
				}
				
				@Override
				public void onUpgrade(SQLiteDatabase db, int arg1, int arg2) {
					Log.i(TAG,"onupdate 方法被调用了 ,在这个方法里面做更新数据库表结构的操作");
					db.execSQL("DROP TABLE IF EXISTS " + account);
					this.onCreate(db);
				}
			}
			```
			2. 获取
			```java
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
    5. Network    
        Store data on the web with your own network server.

2. 清除缓存&清除数据    
    清除数据会清除/data/data/包名中的所有文件     
    getCacheDir()  /data/data/<当前应用程序包名>/cache/，清除缓存的时候会清除该目录中的内容
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 