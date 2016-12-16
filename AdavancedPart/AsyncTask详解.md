AsyncTask详解
===

`AsyncTask`简单的说其实就是`Handler`和`Thread`的结合，就想下面自己写的`MyAsyncTask`一样，这就是它的基本远离，当然它并不止这么简单。

- 经典版异步任务                

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

-  AsyncTask                 

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
类的构造方法中接收三个参数，这里我们不用参数就都给它传Void，new出来AsyncTask类之后然后重写这三个方法，
最后别忘了执行execute方法，其实它的内部和我们写的经典版的异步任务相同，也是里面写了一个在新的线程中去执行耗时的操作，
然后用handler发送Message对象，主线程收到这个Message之后去执行onPostExecute中的内容。


//AsyncTask<Params, Progress, Result> ,params 异步任务执行(doBackgroud方法)需要的参数这个参数的实参可以由execute()方法的参数传入,
// Progess 执行的进度,result是(doBackground方法)执行后的结果 
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

经过上面两部分，我们会发现`AsyncTask`太好了，他帮我们封装了`Handler`和`Thread`，当然他内部肯定会有线程池的管理，所以以后我们在开发中对于耗时的操作可以都用`AsyncTask`来搞定的。其实这种做法是错误的。今天发现公司项目中的网络请求都是用`AsyncTask`来做的(刚换的工作)。这样会有严重的问题。

`AsyncTask`存在的问题:

- `AsyncTask`虽然有`cancel`方法，但是一旦执行了`doInBackground`方法，就算调用取消方法，也会执行完`doInBackground`方法中的内容才会停止。
- 串行还是并行的问题。
    在`1.6`之前，`AsyncTask`是串行执行任务的。`1.6`的时候开始采用线程池并行处理。但是从`3.0`开始为了解决`AsyncTask`的并发问题，`AsyncTask`又采用一个现成来串行执行任务。(串行啊，每个任务10秒，五个任务，最后一个就要到50秒的时候才执行完)
- 线程池的问题。


先从源码的角度分析下:             
打开源码后先看下他的注释，注释把我们所关心的内容说的很明白了。`AsyncTask`并不是设计来处理耗时操作的，耗时的上限最多为几秒钟。

```
AsyncTask enables proper and easy use of the UI thread. This class allows to perform background operations and 
 publish results on the UI thread without having to manipulate threads and/or handlers. 
AsyncTask is designed to be a helper class around Thread and Handler and does not constitute a generic threading 
 framework. ***AsyncTasks should ideally be used for short operations (a few seconds at the most.) If you need to keep 
 threads running for long periods of time, it is highly recommended you use the various APIs provided by the 
 java.util.concurrent package such as Executor, ThreadPoolExecutor and FutureTask. ***
 
There are a few threading rules that must be followed for this class to work properly: 
	- The AsyncTask class must be loaded on the UI thread. This is done automatically as of 
	 android.os.Build.VERSION_CODES.JELLY_BEAN. 
	- The task instance must be created on the UI thread. 
	- execute must be invoked on the UI thread. 
	- Do not call onPreExecute(), onPostExecute, doInBackground, onProgressUpdate manually. 
	- The task can be executed only once (an exception will be thrown if a second execution is attempted.) Memory 
	 observability
	 
    When first introduced, AsyncTasks were executed serially on a single background thread. Starting with 
android.os.Build.VERSION_CODES.DONUT, this was changed to a pool of threads allowing multiple tasks to operate 
in parallel. Starting with android.os.Build.VERSION_CODES.HONEYCOMB, tasks are executed on a single thread to 
avoid common application errors caused by parallel execution. 
If you truly want parallel execution, you can invoke executeOnExecutor(java.util.concurrent.Executor, Object[]) with 
 THREAD_POOL_EXECUTOR.
```

拿到源码我们应该从哪里入手: 使用的时候我们都是 `new AsyncTask<>.execute()`所以我们可以先从构造方法和`execute`方法入手:

```java
/**
 * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
 */
public AsyncTask() {
    // 初始化mWorker
	mWorker = new WorkerRunnable<Params, Result>() {
		public Result call() throws Exception {
			// 修改该变量值
			mTaskInvoked.set(true);

			Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
			// 熟悉的doInBackground方法，并且返回该方法的返回值。
			//noinspection unchecked
			return postResult(doInBackground(mParams));
		}
	};
	// 初始化mFuture并且将mWorker作为参数。这个FutureTask是什么...我也不知道，放狗查了一下。FutureTask是一种可以取消的异步的计算任务实现了Runnable接口，
	// 它可以让程序员准确地知道线程什么时候执行完成并获得到线程执行完成后返回的结果。其实就是FutureTask就是个子线程，会去执行mWorker回调中的耗时的操作
	// 然后在执行完后执行done回调方法。
	mFuture = new FutureTask<Result>(mWorker) {
		@Override
		protected void done() {
			try {
			    // 执行完成后的操作
				postResultIfNotInvoked(get());
			} catch (InterruptedException e) {
				android.util.Log.w(LOG_TAG, e);
			} catch (ExecutionException e) {
				throw new RuntimeException("An error occured while executing doInBackground()",
						e.getCause());
			} catch (CancellationException e) {
				postResultIfNotInvoked(null);
			}
		}
	};
}
```

`WorkerRunnable`是`Callable`接口的抽象实现类:

```java
private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
	Params[] mParams;
}
```

下面上`postResultIfNotInvoked()`源码:      
```java
private void postResultIfNotInvoked(Result result) {
	final boolean wasTaskInvoked = mTaskInvoked.get();
	if (!wasTaskInvoked) {
		postResult(result);
	}
}
// 通过Handler和Message将结果发布出去
private Result postResult(Result result) {
	@SuppressWarnings("unchecked")
	// 调用getHandler去发送Message
	Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
			new AsyncTaskResult<Result>(this, result));
	message.sendToTarget();
	return result;
}
```

那我们再看一下`getHandler()`方法得到的是哪个`Handler`:
```java
private static Handler getHandler() {
	synchronized (AsyncTask.class) {
		if (sHandler == null) {
			sHandler = new InternalHandler();
		}
		return sHandler;
	}
}
```
那接下来再看一下`InternalHandler`的实现:
```java
private static class InternalHandler extends Handler {
	public InternalHandler() {
		super(Looper.getMainLooper());
	}

	@SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
	@Override
	public void handleMessage(Message msg) {
		AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
		switch (msg.what) {
			case MESSAGE_POST_RESULT:
				// There is only one result
				result.mTask.finish(result.mData[0]);
				break;
			case MESSAGE_POST_PROGRESS:
				result.mTask.onProgressUpdate(result.mData);
				break;
		}
	}
}
```
我们看到如果判断消息类型为`MESSAGE_POST_RESULT`时，回去执行`finish()`方法，接着看一下`result.mTask.finish()`方法的源码:
```java
private void finish(Result result) {
	if (isCancelled()) {
	    // 如果被取消了就执行onCancelled方法，这就是为什么虽然AsyncTask可以取消，但是doInBackground方法还是会执行完的原因。
		onCancelled(result);
	} else {
	    // 没被取消就执行oPostExecute方法
		onPostExecute(result);
	}
	mStatus = Status.FINISHED;
}
```

到这里我们会发现已经分析完了`doInBackground`方法执行完后的一系列操作。那`onPreExecute`方法是在哪里?

好了，接着看`execute()`方法	:
```java
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
	return executeOnExecutor(sDefaultExecutor, params);
}
```
里面调用了`executeOnExecutor()`，我们看一下`executeOnExecutor()`方法:
```java
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
		Params... params) {
	if (mStatus != Status.PENDING) {
		switch (mStatus) {
			case RUNNING:
				throw new IllegalStateException("Cannot execute task:"
						+ " the task is already running.");
			case FINISHED:
				throw new IllegalStateException("Cannot execute task:"
						+ " the task has already been executed "
						+ "(a task can be executed only once)");
		}
	}

	mStatus = Status.RUNNING;
	// 看到我们熟悉的onPreExecute()方法。
	onPreExecute();
    // 将参数设置给mWorker变量
	mWorker.mParams = params;
	// 执行了Executor的execute方法并用mFuture为参数，这个exec就是上面的sDefaultExecutor
	exec.execute(mFuture);

	return this;
}
```
我们看一下`sDefaultExecutor`是什么: 
```java
/**
 * An {@link Executor} that executes tasks one at a time in serial
 * order.  This serialization is global to a particular process.
 */
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static final int MESSAGE_POST_RESULT = 0x1;
private static final int MESSAGE_POST_PROGRESS = 0x2;

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
```
从上面的部分能够看出`sDefaultExecutor`是一个`SerialExecutor`对象，好了，接下来看一下`SerialExecutor`类:
```java
private static class SerialExecutor implements Executor {
    // 用一个队列来管理所有的runnable。offer是把要执行的添加进来，在scheduleNext中取出来去执行。
	final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
	Runnable mActive;

	public synchronized void execute(final Runnable r) {
	    // 终于找到了sDefaultExecutor.execute()所真正执行的部分。
		mTasks.offer(new Runnable() {
			public void run() {
				try {
				    // 就是mFuture的run方法，他会去调用mWorker.call方法，这样就会执行doInBackground方法，执行完后会把返回值用Handler发送出去
					r.run();
				} finally {
					scheduleNext();
				}
			}
		});
		if (mActive == null) {
			scheduleNext();
		}
	}

	protected synchronized void scheduleNext() {
		if ((mActive = mTasks.poll()) != null) {
		    // 去取队列中的runnable去执行，这个mActive其实就是mFuture对象。
			THREAD_POOL_EXECUTOR.execute(mActive);
		}
	}
}
```

所以从`SerialExecutor`中我们能看到这就是为什么会串行的去执行了。因为他只会取队列的第一个去执行，其他的都在队列中等待。

但是这里`THREAD_POOL_EXECUTOR`是什么呢？　　　　
```java
/**
 * An {@link Executor} that can be used to execute tasks in parallel.
 */
public static final Executor THREAD_POOL_EXECUTOR
		= new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
				TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```
静态常量，也就是所不管你用多少个`AsyncTask`都会用这同一个线程池。

接着我们重点看一下`THREAD_POOL_EXECUTOR.execute(mActive)`:
因为`mActive`就是`mFuture = new FutureTask<Result>(mWorker)`。所以在执行`execute`方法时会执行`FutureTask`的`run`方法:
```java
public void run() {
	if (state != NEW ||
		!UNSAFE.compareAndSwapObject(this, runnerOffset,
									 null, Thread.currentThread()))
		return;
	try {
		Callable<V> c = callable;
		if (c != null && state == NEW) {
			V result;
			boolean ran;
			try {
			    // 他会去调用 Callable的call()方法，而上面传入的Callable参数是mWorker。所以这里就会调用mWorker的call方法。
				// 通过这里就和之前我们讲的doInBackground方法联系上了.
				result = c.call();
				ran = true;
			} catch (Throwable ex) {
				result = null;
				ran = false;
				setException(ex);
			}
			if (ran)
				set(result);
		}
	} finally {
		// runner must be non-null until state is settled to
		// prevent concurrent calls to run()
		runner = null;
		// state must be re-read after nulling runner to prevent
		// leaked interrupts
		int s = state;
		if (s >= INTERRUPTING)
			handlePossibleCancellationInterrupt(s);
	}
}
```

到这里就分析完了。

下面把完整的代码粘贴上(5.1.1):
```java
public abstract class AsyncTask<Params, Progress, Result> {
    private static final String LOG_TAG = "AsyncTask";

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

    /**
     * An {@link Executor} that executes tasks one at a time in serial
     * order.  This serialization is global to a particular process.
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    private static final int MESSAGE_POST_RESULT = 0x1;
    private static final int MESSAGE_POST_PROGRESS = 0x2;

    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    private static InternalHandler sHandler;

    private final WorkerRunnable<Params, Result> mWorker;
    private final FutureTask<Result> mFuture;

    private volatile Status mStatus = Status.PENDING;
    
    private final AtomicBoolean mCancelled = new AtomicBoolean();
    private final AtomicBoolean mTaskInvoked = new AtomicBoolean();

    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

    /**
     * Indicates the current status of the task. Each status will be set only once
     * during the lifetime of a task.
     */
    public enum Status {
        /**
         * Indicates that the task has not been executed yet.
         */
        PENDING,
        /**
         * Indicates that the task is running.
         */
        RUNNING,
        /**
         * Indicates that {@link AsyncTask#onPostExecute} has finished.
         */
        FINISHED,
    }

    private static Handler getHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler();
            }
            return sHandler;
        }
    }

    /** @hide */
    public static void setDefaultExecutor(Executor exec) {
        sDefaultExecutor = exec;
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     */
    public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                return postResult(doInBackground(mParams));
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }

    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }

    /**
     * Returns the current status of this task.
     *
     * @return The current status.
     */
    public final Status getStatus() {
        return mStatus;
    }

    /**
     * Override this method to perform a computation on a background thread. The
     * specified parameters are the parameters passed to {@link #execute}
     * by the caller of this task.
     *
     * This method can call {@link #publishProgress} to publish updates
     * on the UI thread.
     *
     * @param params The parameters of the task.
     *
     * @return A result, defined by the subclass of this task.
     *
     * @see #onPreExecute()
     * @see #onPostExecute
     * @see #publishProgress
     */
    protected abstract Result doInBackground(Params... params);

    /**
     * Runs on the UI thread before {@link #doInBackground}.
     *
     * @see #onPostExecute
     * @see #doInBackground
     */
    protected void onPreExecute() {
    }

    /**
     * <p>Runs on the UI thread after {@link #doInBackground}. The
     * specified result is the value returned by {@link #doInBackground}.</p>
     * 
     * <p>This method won't be invoked if the task was cancelled.</p>
     *
     * @param result The result of the operation computed by {@link #doInBackground}.
     *
     * @see #onPreExecute
     * @see #doInBackground
     * @see #onCancelled(Object) 
     */
    @SuppressWarnings({"UnusedDeclaration"})
    protected void onPostExecute(Result result) {
    }

    /**
     * Runs on the UI thread after {@link #publishProgress} is invoked.
     * The specified values are the values passed to {@link #publishProgress}.
     *
     * @param values The values indicating progress.
     *
     * @see #publishProgress
     * @see #doInBackground
     */
    @SuppressWarnings({"UnusedDeclaration"})
    protected void onProgressUpdate(Progress... values) {
    }

    /**
     * <p>Runs on the UI thread after {@link #cancel(boolean)} is invoked and
     * {@link #doInBackground(Object[])} has finished.</p>
     * 
     * <p>The default implementation simply invokes {@link #onCancelled()} and
     * ignores the result. If you write your own implementation, do not call
     * <code>super.onCancelled(result)</code>.</p>
     *
     * @param result The result, if any, computed in
     *               {@link #doInBackground(Object[])}, can be null
     * 
     * @see #cancel(boolean)
     * @see #isCancelled()
     */
    @SuppressWarnings({"UnusedParameters"})
    protected void onCancelled(Result result) {
        onCancelled();
    }    
    
    /**
     * <p>Applications should preferably override {@link #onCancelled(Object)}.
     * This method is invoked by the default implementation of
     * {@link #onCancelled(Object)}.</p>
     * 
     * <p>Runs on the UI thread after {@link #cancel(boolean)} is invoked and
     * {@link #doInBackground(Object[])} has finished.</p>
     *
     * @see #onCancelled(Object) 
     * @see #cancel(boolean)
     * @see #isCancelled()
     */
    protected void onCancelled() {
    }

    /**
     * Returns <tt>true</tt> if this task was cancelled before it completed
     * normally. If you are calling {@link #cancel(boolean)} on the task,
     * the value returned by this method should be checked periodically from
     * {@link #doInBackground(Object[])} to end the task as soon as possible.
     *
     * @return <tt>true</tt> if task was cancelled before it completed
     *
     * @see #cancel(boolean)
     */
    public final boolean isCancelled() {
        return mCancelled.get();
    }

    /**
     * <p>Attempts to cancel execution of this task.  This attempt will
     * fail if the task has already completed, already been cancelled,
     * or could not be cancelled for some other reason. If successful,
     * and this task has not started when <tt>cancel</tt> is called,
     * this task should never run. If the task has already started,
     * then the <tt>mayInterruptIfRunning</tt> parameter determines
     * whether the thread executing this task should be interrupted in
     * an attempt to stop the task.</p>
     * 
     * <p>Calling this method will result in {@link #onCancelled(Object)} being
     * invoked on the UI thread after {@link #doInBackground(Object[])}
     * returns. Calling this method guarantees that {@link #onPostExecute(Object)}
     * is never invoked. After invoking this method, you should check the
     * value returned by {@link #isCancelled()} periodically from
     * {@link #doInBackground(Object[])} to finish the task as early as
     * possible.</p>
     *
     * @param mayInterruptIfRunning <tt>true</tt> if the thread executing this
     *        task should be interrupted; otherwise, in-progress tasks are allowed
     *        to complete.
     *
     * @return <tt>false</tt> if the task could not be cancelled,
     *         typically because it has already completed normally;
     *         <tt>true</tt> otherwise
     *
     * @see #isCancelled()
     * @see #onCancelled(Object)
     */
    public final boolean cancel(boolean mayInterruptIfRunning) {
        mCancelled.set(true);
        return mFuture.cancel(mayInterruptIfRunning);
    }

    /**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return The computed result.
     *
     * @throws CancellationException If the computation was cancelled.
     * @throws ExecutionException If the computation threw an exception.
     * @throws InterruptedException If the current thread was interrupted
     *         while waiting.
     */
    public final Result get() throws InterruptedException, ExecutionException {
        return mFuture.get();
    }

    /**
     * Waits if necessary for at most the given time for the computation
     * to complete, and then retrieves its result.
     *
     * @param timeout Time to wait before cancelling the operation.
     * @param unit The time unit for the timeout.
     *
     * @return The computed result.
     *
     * @throws CancellationException If the computation was cancelled.
     * @throws ExecutionException If the computation threw an exception.
     * @throws InterruptedException If the current thread was interrupted
     *         while waiting.
     * @throws TimeoutException If the wait timed out.
     */
    public final Result get(long timeout, TimeUnit unit) throws InterruptedException,
            ExecutionException, TimeoutException {
        return mFuture.get(timeout, unit);
    }

    /**
     * Executes the task with the specified parameters. The task returns
     * itself (this) so that the caller can keep a reference to it.
     * 
     * <p>Note: this function schedules the task on a queue for a single background
     * thread or pool of threads depending on the platform version.  When first
     * introduced, AsyncTasks were executed serially on a single background thread.
     * Starting with {@link android.os.Build.VERSION_CODES#DONUT}, this was changed
     * to a pool of threads allowing multiple tasks to operate in parallel. Starting
     * {@link android.os.Build.VERSION_CODES#HONEYCOMB}, tasks are back to being
     * executed on a single thread to avoid common application errors caused
     * by parallel execution.  If you truly want parallel execution, you can use
     * the {@link #executeOnExecutor} version of this method
     * with {@link #THREAD_POOL_EXECUTOR}; however, see commentary there for warnings
     * on its use.
     *
     * <p>This method must be invoked on the UI thread.
     *
     * @param params The parameters of the task.
     *
     * @return This instance of AsyncTask.
     *
     * @throws IllegalStateException If {@link #getStatus()} returns either
     *         {@link AsyncTask.Status#RUNNING} or {@link AsyncTask.Status#FINISHED}.
     *
     * @see #executeOnExecutor(java.util.concurrent.Executor, Object[])
     * @see #execute(Runnable)
     */
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
		// 这里就多插一嘴了。 sDefaultExecutor在上面我们分析过了，就是一个队列来保证串行进行。从3.0开始都是这样。
		// 那在1.6到3.0之间是怎么并行执行的呢？　按照下面的方式改就可以了
		// return executeOnExecutor(THREAD_POOL_EXECUTOR, params);
		// 就是将sDefaultExecutor改成THREAD_POOL_EXECUTOR， THREAD_POOL_EXECUTOR就是线程池。
		// new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE, TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
		// 原来的CORE_POOL_SIZE是5, KEEP_ALIVE是10, MAXIMUM_POOL_SIZE是128
		// 也就是说可以同时执行的线程是5个，如果超过5个后，超过的部分就会放到缓存队列中，如果超过了128那就挂了
    }

    /**
     * Executes the task with the specified parameters. The task returns
     * itself (this) so that the caller can keep a reference to it.
     * 
     * <p>This method is typically used with {@link #THREAD_POOL_EXECUTOR} to
     * allow multiple tasks to run in parallel on a pool of threads managed by
     * AsyncTask, however you can also use your own {@link Executor} for custom
     * behavior.
     * 
     * <p><em>Warning:</em> Allowing multiple tasks to run in parallel from
     * a thread pool is generally <em>not</em> what one wants, because the order
     * of their operation is not defined.  For example, if these tasks are used
     * to modify any state in common (such as writing a file due to a button click),
     * there are no guarantees on the order of the modifications.
     * Without careful work it is possible in rare cases for the newer version
     * of the data to be over-written by an older one, leading to obscure data
     * loss and stability issues.  Such changes are best
     * executed in serial; to guarantee such work is serialized regardless of
     * platform version you can use this function with {@link #SERIAL_EXECUTOR}.
     *
     * <p>This method must be invoked on the UI thread.
     *
     * @param exec The executor to use.  {@link #THREAD_POOL_EXECUTOR} is available as a
     *              convenient process-wide thread pool for tasks that are loosely coupled.
     * @param params The parameters of the task.
     *
     * @return This instance of AsyncTask.
     *
     * @throws IllegalStateException If {@link #getStatus()} returns either
     *         {@link AsyncTask.Status#RUNNING} or {@link AsyncTask.Status#FINISHED}.
     *
     * @see #execute(Object[])
     */
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }

    /**
     * Convenience version of {@link #execute(Object...)} for use with
     * a simple Runnable object. See {@link #execute(Object[])} for more
     * information on the order of execution.
     *
     * @see #execute(Object[])
     * @see #executeOnExecutor(java.util.concurrent.Executor, Object[])
     */
    public static void execute(Runnable runnable) {
        sDefaultExecutor.execute(runnable);
    }

    /**
     * This method can be invoked from {@link #doInBackground} to
     * publish updates on the UI thread while the background computation is
     * still running. Each call to this method will trigger the execution of
     * {@link #onProgressUpdate} on the UI thread.
     *
     * {@link #onProgressUpdate} will not be called if the task has been
     * canceled.
     *
     * @param values The progress values to update the UI with.
     *
     * @see #onProgressUpdate
     * @see #doInBackground
     */
    protected final void publishProgress(Progress... values) {
        if (!isCancelled()) {
            getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<Progress>(this, values)).sendToTarget();
        }
    }

    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }

    private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }

    private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }

    @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
}

```
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 