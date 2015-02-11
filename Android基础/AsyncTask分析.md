AsyncTask分析
===

1. 经典版异步任务
	```java
	public abstract class MyAsyncTask {
		private Handler handler = new Handler(){
			public void handleMessage(android.os.Message msg) {
				onPostExecute();
			};
		};
		
		/**
		 * 后台任务执行之前 提示用户的界面操作.
		 */
		public abstract void onPreExecute();
		
		/**
		 * 后台任务执行之后 更新界面的操作.
		 */
		public abstract void onPostExecute();
		
		/**
		 * 在后台执行的一个耗时的操作.
		 */
		public abstract void doInBackground();
		
		
		public void execute(){
			//1. 耗时任务执行之前通知界面更新
			onPreExecute();
			new Thread(){
				public void run() {
					doInBackground();
					handler.sendEmptyMessage(0);
				};
			}.start();
			
		} 
	}
	```
2. 谷歌异步任务
	```java
		new AsyncTask<Void, Void, Void>() {
				@Override
				protected Void doInBackground(Void... params) {
					blackNumberInfos = dao.findByPage(startIndex, maxNumber);
					return null;
				}
				@Override
				protected void onPreExecute() {
					loading.setVisibility(View.VISIBLE);
					super.onPreExecute();
				}
				@Override
				protected void onPostExecute(Void result) {
					loading.setVisibility(View.INVISIBLE);
					if (adapter == null) {// 第一次加载数据 数据适配器还不存在
						adapter = new CallSmsAdapter();
						lv_callsms_safe.setAdapter(adapter);
					} else {// 有新的数据被添加进来.
						adapter.notifyDataSetChanged();// 通知数据适配器 数据变化了.
					}
					super.onPostExecute(result);
				}
			}.execute(); 

	类的构造方法中接收三个参数，这里我们不用参数就都给它传Void，new出来AsyncTask类之后然后重写这三个方法，最后别忘了执行execute方法，其实它的内部和我们写的经典版的异步任务相同，也是里面写了一个在新的线程中去执行耗时的操作，然后用handler发送Message对象，主线程收到这个Message之后去执行onPostExecute中的内容。


		//AsyncTask<Params, Progress, Result> ,params 异步任务执行(doBackgroud方法)需要的参数这个参数的实参可以由execute()方法的参数传入,Progess 执行的进度,result是(doBackground方法)执行后的结果 

		new AsyncTask<String, Void, Boolean>() { 
				ProgressDialog pd;
				@Override
				protected Boolean doInBackground(String... params) { //这里返回的就是执行的接口，这个返回的结果会传递给onPostExecute的参数
					try {
						String filename = params[0];//得到execute传入的参数
						File file = new File(Environment.getExternalStorageDirectory(),filename);
						FileOutputStream fos = new FileOutputStream(file);
						SmsUtils.backUp(getApplicationContext(), fos, new BackUpStatusListener() {
							public void onBackUpProcess(int process) {
								pd.setProgress(process);
							}
							
							public void beforeBackup(int max) {
								pd.setMax(max);
							}
						});
						return true;
					} catch (Exception e) {
						e.printStackTrace();
						return false;
					}    
				}
				@Override
				protected void onPreExecute() {
					pd = new ProgressDialog(AtoolsActivity.this);
					pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
					pd.setMessage("正在备份短信");
					pd.show();
					super.onPreExecute();
				}
				@Override
				protected void onPostExecute(Boolean result) {
					pd.dismiss();
					if(result){
						Toast.makeText(getApplicationContext(), "备份成功", 0).show();
					}else{
						Toast.makeText(getApplicationContext(), "备份失败", 0).show();
					}
					super.onPostExecute(result);
				}
				
			}.execute("backup.xml"); //这里传入的参数就是doInBackgound中的参数，会传入到doInBackground中
	 
	ProgressDialog有个方法
	incrementProgressBy(int num);方法，这个方法能够让进度条自动增加，如果参数为1就是进度条累加1。
	 
	可以给ProgressDialog添加一个监听dismiss的监听器。pd.setOnDismisListener(DismisListener listener);让其在取消显示后做什么事
	```

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 