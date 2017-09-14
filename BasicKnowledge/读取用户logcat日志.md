读取用户logcat日志
===

1. 读取用户日志需要权限`android.permission.READ_LOGS`

2. 在一个服务中开启logcat程序，然后读取
	```java
	public void onCreate() {
	        super.onCreate();
	        new Thread(){
	            public void run() {
	                try {
	                    File file = new File(Environment.getExternalStorageDirectory(),"log.txt");
	                    FileOutputStream fos = new FileOutputStream(file);
	                    Process process = Runtime.getRuntime().exec("logcat");
	                    InputStream is = process.getInputStream();
	                    BufferedReader br = new BufferedReader(new InputStreamReader(is));
	                    String line;
	                    while((line = br.readLine())!=null){
	                        if(line.contains("I/ActivityManager")){
	                            fos.write(line.getBytes());
	                            fos.flush();
	                        }
	                    }
	                    fos.close();
	                } catch (Exception e) {
	                    e.printStackTrace();
	                }
	            };
	        }.start();
    	}
	```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 