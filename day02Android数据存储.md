day02Android数据存储
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
    5. Network    
        Store data on the web with your own network server.

2. 清除缓存&清除数据    
    清除数据会清除/data/data/包名中的所有文件     
    getCacheDir()  /data/data/<当前应用程序包名>/cache/，清除缓存的时候会清除该目录中的内容
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 