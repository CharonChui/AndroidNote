获取手机及SD卡可用存储空间
===
 
存储设备都是分块的，获取一共有多少块，然后算出来每一块的大小就能得到总的大小   
```java 
File file = Environment.getExternalStorageDirectory();//获取SD卡的目录
StatFs statf = new StatFs(file.getAbsolutePath());
long count = statf.getAvailableBlocks();
long size = statf.getBlockSize();
System.out.println("sd卡可用空间:"+ Formatter.formatFileSize(this, count*size));

File path = Environment.getDataDirectory();//获取手机存储设备的目录
StatFs stat = new StatFs(path.getPath());
long blockSize = stat.getBlockSize();
long availableBlocks = stat.getAvailableBlocks();
System.out.println("手机内部空间:"+ Formatter.formatFileSize(this, availableBlocks*blockSize));
```
Formatter类的formatFileSize内部能自动将一个比特大小的值转换成M,G,K等
```
static String formatFileSize(Context context, long number)
Formats a content size to be in the form of bytes, kilobytes, megabytes, etc
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
