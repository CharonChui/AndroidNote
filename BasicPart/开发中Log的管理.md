开发中Log的管理
===

**LogUtil**是一个管理`Log`打印的工具类。在开发的不同阶段中通过对该类的控制来实现不同级别`Log`的打印。        
```java    
public class LogUtil {
	public static final int VERBOSE = 5;
	public static final int DEBUG = 4;
	public static final int INFO = 3;
	public static final int WARN = 2;
	public static final int ERROR = 1;

	/** 通过改变该值来进行不同级别的Log打印，上线的时将该值改为0来关闭所有log打印 */
	public final static int LOG_LEVEL = 6;

	public static void v(String tag, String msg) {
		if (LOG_LEVEL > VERBOSE) {
			Log.v(tag, msg);
		}
	}

	public static void d(String tag, String msg) {
		if (LOG_LEVEL > DEBUG) {
			Log.d(tag, msg);
		}
	}

	public static void i(String tag, String msg) {
		if (LOG_LEVEL > INFO) {
			Log.i(tag, msg);
		}
	}

	public static void w(String tag, String msg) {
		if (LOG_LEVEL > WARN) {
			Log.w(tag, msg);
		}
	}

	public static void e(String tag, String msg) {
		if (LOG_LEVEL > ERROR) {
			Log.e(tag, msg);
		}
	}
}
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
