Volley源码分析
---

- Volley简介    

    [volley官方地址](https://android.googlesource.com/platform/frameworks/volley)                                                   
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/volley.png?raw=true)                   

    在`Google I/0 2013`中发布了`Volley`.`Volley`是`Android`平台上的网络通信库，能使网络通信更快，更简单，更健壮。

    这是`Volley`名称的由来:`a burst or emission of many things or a large amount at once`.`Volley`特别适合数据量不大但是通信频繁的场景。   

    `Github`上面已经有大神做了镜像，使用`Gradle`更方便。[Volley On Github](https://github.com/mcxiaoke/android-volley)            

- Volley使用                   
    `Volley`的使用非常方便。
    - 首先就是要构建一个`RequestQueue`的请求队列，它可以缓存所有`Http`请求，内部处理的非常完善，通常我们整个应用只需要一个`RequestQueue`对象即可。
    - 创建`StringRequest`对象，并且传入相应的请求地址以及添加请求成功和失败的回调方法。这一步的意思就是创建一个新的网络请求。
    - 将刚才新创建的`StringRequest`对象加入到`RequestQueue`队列中。这样就相当于会去执行该请求。等到执行成功后就可以在`StringRequest`中设置的回调方法里面获取到相应的结果。      
    总结一下，其实挺像我们浏览器的操作，第一步打开浏览器，第二步输入`URL`地址第三步按回车去执行。    

    ```java
    public class HttpUtil {
        private static HttpUtil instance;
        private RequestQueue mQueue;
    
        private HttpUtil(Context context) {
            mQueue = Volley.newRequestQueue(context);
        }
    
        public static synchronized HttpUtil getInstance(Context context) {
            if (instance == null) {
                instance = new HttpUtil(context.getApplicationContext());
            }
            return instance;
        }
    
        public Request sendGetRequest(String url, final HttpListener listener) {
            StringRequest stringRequest = new StringRequest(url, new Response.Listener<String>() {
                @Override
                public void onResponse(String response) {
                    if (listener != null) {
                        listener.onResponse(response);
                    }
                }
            }, new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    if (listener != null) {
                        listener.onErrorResponse(error);
                    }
                }
            });
            mQueue.add(stringRequest);
            return stringRequest;
        }
    
        public Request sendPostRequest(String url, final Map<String, String> map, final HttpListener listener) {
            final StringRequest stringRequest = new StringRequest(Request.Method.POST, url, new Response.Listener<String>() {
                @Override
                public void onResponse(String response) {
                    if (listener != null) {
                        listener.onResponse(response);
                    }
    
                }
            }, new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    if (listener != null) {
                        listener.onErrorResponse(error);
                    }
                }
            }) {
                @Override
                protected Map<String, String> getParams() throws AuthFailureError {
                    return map;
                }
            };
            mQueue.add(stringRequest);
            return stringRequest;
        }
    
        /**
         * Response listener of HttpUtil.
         */
        public interface HttpListener {
    
            void onResponse(String response);
    
            void onErrorResponse(VolleyError error);
        }
    }
    ```

到这里我们就简单的介绍了它的使用，当然还有一些其他的`Request`对象例如`JsonRequest`等，他们的使用方法都是一样的，这里就不再说明了。当然`Volley`里面还提供了对图片的处理，例如`NetworkImageView`空间和`ImageRequest`等，因为这里图片用到的不太多，所以暂时不去分析了。


接下来我们就从源码的角度去分析一下：      
这里我们都知道使用的时候最新要初始化一个`RequestQueue`所以，我们首先看一下`Volley.newRequestQueue`方法。
```java

public class Volley {

    /** Default on-disk cache directory. */
    private static final String DEFAULT_CACHE_DIR = "volley";

    /**
     * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
     * You may set a maximum size of the disk cache in bytes.
     *
     * @param context A {@link Context} to use for creating the cache dir.
     * @param stack An {@link HttpStack} to use for the network, or null for default.
     * @param maxDiskCacheBytes the maximum size of the disk cache, in bytes. Use -1 for default size.
     * @return A started {@link RequestQueue} instance.
     */
    public static RequestQueue newRequestQueue(Context context, HttpStack stack, int maxDiskCacheBytes) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
        try {
            String packageName = context.getPackageName();
            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
            // 使用包名和versionCode作为userAgent
            userAgent = packageName + "/" + info.versionCode;
        } catch (NameNotFoundException e) {
        }

        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                // 在9及以上Volleys使用HttpURLConnection，9一下使用HttpClient。这里是有原因的，因为在9之前HttpURLConnection有Bug。
                // 那HttpClient那么好为什么不一直使用它，要在9及以后使用HttpURLConnection呢？这里也是有原因的。从9开始HttpURLConnection
                // 将自动添加`Accept-Encoding:gzip`头字段到`request`请求中，并做相应处理，一般我们请求都是字符串，所以压缩可以使数据大小大幅降低。
                // 但是这也会带来问题的，之前开发中就遇到过。因为启用了压缩，所以`Content-Lenght`字段返回的是压缩后的大小。使用`getContentLength()`
                // 方法去分配解压缩后数据大小是错误的。应该从response中读取字节直到`InputStream.read()`返回-1为止。当时我们在开发下载时就遇到过这个
                // 问题，在下载视频时没有问题，但是在下载小说的时候就会发现`content-length`返回值不对。
                // 总结一下就是在9之前HttpClient的bug更少，而HttpURLConnection存在严重的Bug。但是从9开始HttpURLConnection更小巧，API更简单，压缩以及
                // response cache的使用减少了网络流量，提高了网络速度，也就更省电，所以更适合在Android中使用
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }
        // 创建BasicNetwork对象，下面会介绍BasicNetwork(HttpStack)方法的内部实现
        Network network = new BasicNetwork(stack);
        
        RequestQueue queue;
        if (maxDiskCacheBytes <= -1)
        {
                // 如果不指定大小的话，默认大小为5M
            // No maximum size specified
        	queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        }
        else
        {
        	// Disk cache size specified
        	queue = new RequestQueue(new DiskBasedCache(cacheDir, maxDiskCacheBytes), network);
        }

        queue.start();

        return queue;
    }
    
    /**
     * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
     * You may set a maximum size of the disk cache in bytes.
     *
     * @param context A {@link Context} to use for creating the cache dir.
     * @param maxDiskCacheBytes the maximum size of the disk cache, in bytes. Use -1 for default size.
     * @return A started {@link RequestQueue} instance.
     */
    public static RequestQueue newRequestQueue(Context context, int maxDiskCacheBytes) {
        return newRequestQueue(context, null, maxDiskCacheBytes);
    }
    
    /**
     * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
     *
     * @param context A {@link Context} to use for creating the cache dir.
     * @param stack An {@link HttpStack} to use for the network, or null for default.
     * @return A started {@link RequestQueue} instance.
     */
    public static RequestQueue newRequestQueue(Context context, HttpStack stack)
    {
    	return newRequestQueue(context, stack, -1);
    }
    
    /**
     * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
     *
     * @param context A {@link Context} to use for creating the cache dir.
     * @return A started {@link RequestQueue} instance.
     */
    public static RequestQueue newRequestQueue(Context context) {
        return newRequestQueue(context, null);
    }

}

```

接着来看一上上面提到的new BasicNetwork(HttpStack)方法的实现,可以看到他内部的缓存大小是4k
```java
private static int DEFAULT_POOL_SIZE = 4096;

protected final HttpStack mHttpStack;

protected final ByteArrayPool mPool;

/**
 * @param httpStack HTTP stack to be used
 */
public BasicNetwork(HttpStack httpStack) {
    // If a pool isn't passed in, then build a small default pool that will give us a lot of
    // benefit and not use too much memory.
    this(httpStack, new ByteArrayPool(DEFAULT_POOL_SIZE));
}

/**
 * @param httpStack HTTP stack to be used
 * @param pool a buffer pool that improves GC performance in copy operations
 */
public BasicNetwork(HttpStack httpStack, ByteArrayPool pool) {
    mHttpStack = httpStack;
    mPool = pool;
}
```

接着我们还要分析一下`RequestQueue`的构造方法以及`start()`方法:      
```java
/**
 * A request dispatch queue with a thread pool of dispatchers.
 *
 * Calling {@link #add(Request)} will enqueue the given Request for dispatch,
 * resolving from either cache or network on a worker thread, and then delivering
 * a parsed response on the main thread.
 *//**
 * A request dispatch queue with a thread pool of dispatchers.
 *
 * Calling {@link #add(Request)} will enqueue the given Request for dispatch,
 * resolving from either cache or network on a worker thread, and then delivering
 * a parsed response on the main thread.
 */
public class RequestQueue {

    /** Used for generating monotonically-increasing sequence numbers for requests. */
    private AtomicInteger mSequenceGenerator = new AtomicInteger();

    /**
     * Staging area for requests that already have a duplicate request in flight.
     *
     * <ul>
     *     <li>containsKey(cacheKey) indicates that there is a request in flight for the given cache
     *          key.</li>
     *     <li>get(cacheKey) returns waiting requests for the given cache key. The in flight request
     *          is <em>not</em> contained in that list. Is null if no requests are staged.</li>
     * </ul>
     */
    private final Map<String, Queue<Request<?>>> mWaitingRequests =
            new HashMap<String, Queue<Request<?>>>();

    /**
     * The set of all requests currently being processed by this RequestQueue. A Request
     * will be in this set if it is waiting in any queue or currently being processed by
     * any dispatcher.
     */
    private final Set<Request<?>> mCurrentRequests = new HashSet<Request<?>>();

	// cache队列
    /** The cache triage queue. */
    private final PriorityBlockingQueue<Request<?>> mCacheQueue =
        new PriorityBlockingQueue<Request<?>>();

	// 网络请求队列
    /** The queue of requests that are actually going out to the network. */
    private final PriorityBlockingQueue<Request<?>> mNetworkQueue =
        new PriorityBlockingQueue<Request<?>>();

    /** Number of network request dispatcher threads to start. */
    private static final int DEFAULT_NETWORK_THREAD_POOL_SIZE = 4;

    /** Cache interface for retrieving and storing responses. */
    private final Cache mCache;

    /** Network interface for performing requests. */
    private final Network mNetwork;

    /** Response delivery mechanism. */
    private final ResponseDelivery mDelivery;

    /** The network dispatchers. */
    private NetworkDispatcher[] mDispatchers;

    /** The cache dispatcher. */
    private CacheDispatcher mCacheDispatcher;

    /**
     * Creates the worker pool. Processing will not begin until {@link #start()} is called.
     *
     * @param cache A Cache to use for persisting responses to disk
     * @param network A Network interface for performing HTTP requests
     * @param threadPoolSize Number of network dispatcher threads to create
     * @param delivery A ResponseDelivery interface for posting responses and errors
     */
    public RequestQueue(Cache cache, Network network, int threadPoolSize,
            ResponseDelivery delivery) {
        mCache = cache;
        mNetwork = network;
        mDispatchers = new NetworkDispatcher[threadPoolSize];
        mDelivery = delivery;
    }

    /**
     * Creates the worker pool. Processing will not begin until {@link #start()} is called.
     *
     * @param cache A Cache to use for persisting responses to disk
     * @param network A Network interface for performing HTTP requests
     * @param threadPoolSize Number of network dispatcher threads to create
     */
    public RequestQueue(Cache cache, Network network, int threadPoolSize) {
        this(cache, network, threadPoolSize,
                new ExecutorDelivery(new Handler(Looper.getMainLooper())));
    }

    /**
     * Creates the worker pool. Processing will not begin until {@link #start()} is called.
     *
     * @param cache A Cache to use for persisting responses to disk
     * @param network A Network interface for performing HTTP requests
     */
    public RequestQueue(Cache cache, Network network) {
        // 默认大小为4
        this(cache, network, DEFAULT_NETWORK_THREAD_POOL_SIZE);
    }

    /**
     * Starts the dispatchers in this queue.
     */
    public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        // 初始化RequestQueue之后就会调用start方法，内部会开启CacheDispatcher,也是Thread的子类，后面再看里面具体的run方法
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i < mDispatchers.length; i++) {
            // 创建4个(默认是4个)NetworkDispatcher一直去执行, NetworkDispatcher是Thread的子类，他会不断的去从mNetworkQueue中取出Requet并用
            // 并用mNetwork去执行，执行完成后再使用mDelivery去分发相应的结果
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
        // 就好像一个工厂一启动，里面就分配了5个搬运工，一个负责搬运cache里面的的请求，4个负责搬运network中的。启动后他们就开始待命
        // 一旦有活来了，就开始去取出活开始干。

    }

    /**
     * Stops the cache and network dispatchers.
     */
    public void stop() {
        if (mCacheDispatcher != null) {
            mCacheDispatcher.quit();
        }
        for (int i = 0; i < mDispatchers.length; i++) {
            if (mDispatchers[i] != null) {
                mDispatchers[i].quit();
            }
        }
    }

    /**
     * Gets a sequence number.
     */
    public int getSequenceNumber() {
        return mSequenceGenerator.incrementAndGet();
    }

    /**
     * Gets the {@link Cache} instance being used.
     */
    public Cache getCache() {
        return mCache;
    }

    /**
     * A simple predicate or filter interface for Requests, for use by
     * {@link RequestQueue#cancelAll(RequestFilter)}.
     */
    public interface RequestFilter {
        public boolean apply(Request<?> request);
    }

    /**
     * Cancels all requests in this queue for which the given filter applies.
     * @param filter The filtering function to use
     */
    public void cancelAll(RequestFilter filter) {
        synchronized (mCurrentRequests) {
            for (Request<?> request : mCurrentRequests) {
                if (filter.apply(request)) {
                    request.cancel();
                }
            }
        }
    }

    /**
     * Cancels all requests in this queue with the given tag. Tag must be non-null
     * and equality is by identity.
     */
    public void cancelAll(final Object tag) {
        if (tag == null) {
            throw new IllegalArgumentException("Cannot cancelAll with a null tag");
        }
        cancelAll(new RequestFilter() {
            @Override
            public boolean apply(Request<?> request) {
                return request.getTag() == tag;
            }
        });
    }

    /**
     * Adds a Request to the dispatch queue.
     * @param request The request to service
     * @return The passed-in request
     */
    public <T> Request<T> add(Request<T> request) {
        // Tag the request as belonging to this queue and add it to the set of current requests.
        request.setRequestQueue(this);
        synchronized (mCurrentRequests) {
            // 添加到mCurrentRequests中，在执行完后的finish方法中会去移除该请求。
            mCurrentRequests.add(request);
        }

        // Process requests in the order they are added.
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");

        // 判断一下该请求能否进行缓存，如果不能缓存就直接添加到网络请求的队列中。这个能不能缓存是怎么判断的？其实就是根据Request中的一个变量来判断。
　　　　// 默认情况下所有的请求都是可以缓存的，可以通过Request.setShouldCache(false)方法，来将其设置为不可缓存状态。
        // If the request is uncacheable, skip the cache queue and go straight to the network.
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }

        // Insert request into stage if there's already a request with the same cache key in flight.
        synchronized (mWaitingRequests) {
            String cacheKey = request.getCacheKey();
            if (mWaitingRequests.containsKey(cacheKey)) {
                // There is already a request in flight. Queue up.
                Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
                if (stagedRequests == null) {
                    stagedRequests = new LinkedList<Request<?>>();
                }
                stagedRequests.add(request);
                mWaitingRequests.put(cacheKey, stagedRequests);
                if (VolleyLog.DEBUG) {
                    VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
                }
            } else {
                // 如果能缓存，并且缓存线程中没有的时候就讲该请求添加到缓存队列中
                // Insert 'null' queue for this cacheKey, indicating there is now a request in
                // flight.
                mWaitingRequests.put(cacheKey, null);
                mCacheQueue.add(request);
            }
            return request;
        }
    }

    /**
     * Called from {@link Request#finish(String)}, indicating that processing of the given request
     * has finished.
     *
     * <p>Releases waiting requests for <code>request.getCacheKey()</code> if
     *      <code>request.shouldCache()</code>.</p>
     */
    void finish(Request<?> request) {
        // Remove from the set of requests currently being processed.
        synchronized (mCurrentRequests) {
            mCurrentRequests.remove(request);
        }

        if (request.shouldCache()) {
            synchronized (mWaitingRequests) {
                String cacheKey = request.getCacheKey();
                Queue<Request<?>> waitingRequests = mWaitingRequests.remove(cacheKey);
                if (waitingRequests != null) {
                    if (VolleyLog.DEBUG) {
                        VolleyLog.v("Releasing %d waiting requests for cacheKey=%s.",
                                waitingRequests.size(), cacheKey);
                    }
                    // Process all queued up requests. They won't be considered as in flight, but
                    // that's not a problem as the cache has been primed by 'request'.
                    mCacheQueue.addAll(waitingRequests);
                }
            }
        }
    }
}
```

看到这里基本都能看的差不多了。官方文档中有句话说的很好，这里用他来总结一下`A RequestQueue needs two things to do its job: a network to perform transport of the requests, and a cache to handle caching. `
顺便再上一张图:       
 
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/volley-request.png?raw=true)                   

总结完之后我们接着进行分析。因为默认情况下请求都是可缓存的，所以都会被添加到mCacheQueue中。添加该队列之后，就会被开始`start`方法所制定的cache搬运工去执行，所以我们要看一下CacheDispatcher的实现。
```java
/**
 * Provides a thread for performing cache triage on a queue of requests.
 *
 * Requests added to the specified cache queue are resolved from cache.
 * Any deliverable response is posted back to the caller via a
 * {@link ResponseDelivery}.  Cache misses and responses that require
 * refresh are enqueued on the specified network queue for processing
 * by a {@link NetworkDispatcher}.
 */
public class CacheDispatcher extends Thread {

    private static final boolean DEBUG = VolleyLog.DEBUG;

    /** The queue of requests coming in for triage. */
    private final BlockingQueue<Request<?>> mCacheQueue;

    /** The queue of requests going out to the network. */
    private final BlockingQueue<Request<?>> mNetworkQueue;

    /** The cache to read from. */
    private final Cache mCache;

    /** For posting responses. */
    private final ResponseDelivery mDelivery;

    /** Used for telling us to die. */
    private volatile boolean mQuit = false;

    /**
     * Creates a new cache triage dispatcher thread.  You must call {@link #start()}
     * in order to begin processing.
     *
     * @param cacheQueue Queue of incoming requests for triage
     * @param networkQueue Queue to post requests that require network to
     * @param cache Cache interface to use for resolution
     * @param delivery Delivery interface to use for posting responses
     */
    public CacheDispatcher(
            BlockingQueue<Request<?>> cacheQueue, BlockingQueue<Request<?>> networkQueue,
            Cache cache, ResponseDelivery delivery) {
        // 将CacheQueue以及networkQueue等传递进来。
        mCacheQueue = cacheQueue;
        mNetworkQueue = networkQueue;
        mCache = cache;
        mDelivery = delivery;
    }

    /**
     * Forces this dispatcher to quit immediately.  If any requests are still in
     * the queue, they are not guaranteed to be processed.
     */
    public void quit() {
        mQuit = true;
        interrupt();
    }

    @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        // Make a blocking call to initialize the cache.
    /**
     * Performs any potentially long-running actions needed to initialize the cache;
     * will be called from a worker thread.
     */
        mCache.initialize();

        while (true) {
            try {
                // 从缓存队列中取出第一个请求
                // Get a request from the cache triage queue, blocking until
                // at least one is available.
                final Request<?> request = mCacheQueue.take();
                request.addMarker("cache-queue-take");

                // If the request has been canceled, don't bother dispatching it.
                if (request.isCanceled()) {
                    request.finish("cache-discard-canceled");
                    continue;
                }

                // Attempt to retrieve this item from cache.
                Cache.Entry entry = mCache.get(request.getCacheKey());
                if (entry == null) {
                    // 如果缓存中找不到该请求，就把该请求添加到网络请求队列中。
                    request.addMarker("cache-miss");
                    // Cache miss; send off to the network dispatcher.
                    mNetworkQueue.put(request);
                    continue;
                }
                // 如果缓存中找到了该请求，接下来就判断该缓存是否过期。
                // If it is completely expired, just send it to the network.
                if (entry.isExpired()) {
                    // 过期了也重新添加到网络请求中
                    request.addMarker("cache-hit-expired");
                    request.setCacheEntry(entry);
                    mNetworkQueue.put(request);
                    continue;
                }
                // 没有过期，就不用再请求了，直接从缓存中取出数据返回即可。
                // We have a cache hit; parse its data for delivery back to the request.
                request.addMarker("cache-hit");
                // 这里的意思就是对数据进行解析，后面再看Request.parseNetworkResponse方法。
                Response<?> response = request.parseNetworkResponse(
                        new NetworkResponse(entry.data, entry.responseHeaders));
                request.addMarker("cache-hit-parsed");
		// 判断缓存数据是否需要刷新，以便使用mDelivery分发结果或者添加到网络请求队列中
                if (!entry.refreshNeeded()) {
                    // Completely unexpired cache hit. Just deliver the response.
                    mDelivery.postResponse(request, response);
                } else {
                    // Soft-expired cache hit. We can deliver the cached response,
                    // but we need to also send the request to the network for
                    // refreshing.
                    request.addMarker("cache-hit-refresh-needed");
                    request.setCacheEntry(entry);

                    // Mark the response as intermediate.
                    response.intermediate = true;

                    // Post the intermediate response back to the user and have
                    // the delivery then forward the request along to the network.
                    mDelivery.postResponse(request, response, new Runnable() {
                        @Override
                        public void run() {
                            try {
                                mNetworkQueue.put(request);
                            } catch (InterruptedException e) {
                                // Not much we can do about this.
                            }
                        }
                    });
                }

            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
        }
    }
}

```

而对于网络请求队列中的任务该如何执行，这里就要看`NetworkDispatcher`的具体实现:        

```java
/**
 * Provides a thread for performing network dispatch from a queue of requests.
 *
 * Requests added to the specified queue are processed from the network via a
 * specified {@link Network} interface. Responses are committed to cache, if
 * eligible, using a specified {@link Cache} interface. Valid responses and
 * errors are posted back to the caller via a {@link ResponseDelivery}.
 */
public class NetworkDispatcher extends Thread {
    /** The queue of requests to service. */
    private final BlockingQueue<Request<?>> mQueue;
    /** The network interface for processing requests. */
    private final Network mNetwork;
    /** The cache to write to. */
    private final Cache mCache;
    /** For posting responses and errors. */
    private final ResponseDelivery mDelivery;
    /** Used for telling us to die. */
    private volatile boolean mQuit = false;

    /**
     * Creates a new network dispatcher thread.  You must call {@link #start()}
     * in order to begin processing.
     *
     * @param queue Queue of incoming requests for triage
     * @param network Network interface to use for performing requests
     * @param cache Cache interface to use for writing responses to cache
     * @param delivery Delivery interface to use for posting responses
     */
    public NetworkDispatcher(BlockingQueue<Request<?>> queue,
            Network network, Cache cache,
            ResponseDelivery delivery) {
        mQueue = queue;
        mNetwork = network;
        mCache = cache;
        mDelivery = delivery;
    }

    /**
     * Forces this dispatcher to quit immediately.  If any requests are still in
     * the queue, they are not guaranteed to be processed.
     */
    public void quit() {
        mQuit = true;
        interrupt();
    }

    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    private void addTrafficStatsTag(Request<?> request) {
        // Tag the request (if API >= 14)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            TrafficStats.setThreadStatsTag(request.getTrafficStatsTag());
        }
    }

    @Override
    public void run() {
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        while (true) {
            long startTimeMs = SystemClock.elapsedRealtime();
            Request<?> request;
            try {
                // Take a request from the queue.
                request = mQueue.take();
            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }

            try {
                request.addMarker("network-queue-take");

                // If the request was cancelled already, do not perform the
                // network request.
                if (request.isCanceled()) {
                    request.finish("network-discard-cancelled");
                    continue;
                }

                addTrafficStatsTag(request);
                // mNetwork会去执行对应的request请求，后面再看里面的具体实现
                // Perform the network request.
                NetworkResponse networkResponse = mNetwork.performRequest(request);
                request.addMarker("network-http-complete");

                // If the server returned 304 AND we delivered a response already,
                // we're done -- don't deliver a second identical response.
                if (networkResponse.notModified && request.hasHadResponseDelivered()) {
                    request.finish("not-modified");
                    continue;
                }
                // 将网络请求返回的NetworkResponse交给request.parseNetworkResponse进行处理
                // Parse the response here on the worker thread.
                Response<?> response = request.parseNetworkResponse(networkResponse);
                request.addMarker("network-parse-complete");

                // Write to cache if applicable.
                // TODO: Only update cache metadata instead of entire record for 304s.
                if (request.shouldCache() && response.cacheEntry != null) {
                    mCache.put(request.getCacheKey(), response.cacheEntry);
                    request.addMarker("network-cache-written");
                }

                // Post the response back.
                request.markDelivered();
                // mDelivery进行分发
                mDelivery.postResponse(request, response);
            } catch (VolleyError volleyError) {
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                parseAndDeliverNetworkError(request, volleyError);
            } catch (Exception e) {
                VolleyLog.e(e, "Unhandled exception %s", e.toString());
                VolleyError volleyError = new VolleyError(e);
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                mDelivery.postError(request, volleyError);
            }
        }
    }

    private void parseAndDeliverNetworkError(Request<?> request, VolleyError error) {
        error = request.parseNetworkError(error);
        mDelivery.postError(request, error);
    }
}
```
上面会执行到`mNetwork.performRequest`方法，而`Network`是一个接口，具体的实现要看`BaseNetwork`中的实现：      
```java
    @Override
    public NetworkResponse performRequest(Request<?> request) throws VolleyError {
        long requestStart = SystemClock.elapsedRealtime();
        while (true) {
            HttpResponse httpResponse = null;
            byte[] responseContents = null;
            Map<String, String> responseHeaders = Collections.emptyMap();
            try {
                // Gather headers.
                Map<String, String> headers = new HashMap<String, String>();
                addCacheHeaders(headers, request.getCacheEntry());
                // 调用mHttpStack.performRequest方法，这里就是newRequestQueue中创建的部分，9及以上为HurlStack，9以下为HttpClientStach,具体就是真正执行网络请求的部分了。
                httpResponse = mHttpStack.performRequest(request, headers);
                StatusLine statusLine = httpResponse.getStatusLine();
                int statusCode = statusLine.getStatusCode();

                responseHeaders = convertHeaders(httpResponse.getAllHeaders());
                // Handle cache validation.
                if (statusCode == HttpStatus.SC_NOT_MODIFIED) {

                    Entry entry = request.getCacheEntry();
                    if (entry == null) {
                        return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, null,
                                responseHeaders, true,
                                SystemClock.elapsedRealtime() - requestStart);
                    }

                    // A HTTP 304 response does not have all header fields. We
                    // have to use the header fields from the cache entry plus
                    // the new ones from the response.
                    // http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.5
                    entry.responseHeaders.putAll(responseHeaders);
                    return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, entry.data,
                            entry.responseHeaders, true,
                            SystemClock.elapsedRealtime() - requestStart);
                }
                
                // Handle moved resources
                if (statusCode == HttpStatus.SC_MOVED_PERMANENTLY || statusCode == HttpStatus.SC_MOVED_TEMPORARILY) {
                	String newUrl = responseHeaders.get("Location");
                	request.setRedirectUrl(newUrl);
                }

                // Some responses such as 204s do not have content.  We must check.
                if (httpResponse.getEntity() != null) {
                  responseContents = entityToBytes(httpResponse.getEntity());
                } else {
                  // Add 0 byte response as a way of honestly representing a
                  // no-content request.
                  responseContents = new byte[0];
                }

                // if the request is slow, log it.
                long requestLifetime = SystemClock.elapsedRealtime() - requestStart;
                logSlowRequests(requestLifetime, request, responseContents, statusLine);

                if (statusCode < 200 || statusCode > 299) {
                    throw new IOException();
                }
                return new NetworkResponse(statusCode, responseContents, responseHeaders, false,
                        SystemClock.elapsedRealtime() - requestStart);
            } catch (SocketTimeoutException e) {
                attemptRetryOnException("socket", request, new TimeoutError());
            } catch (ConnectTimeoutException e) {
                attemptRetryOnException("connection", request, new TimeoutError());
            } catch (MalformedURLException e) {
                throw new RuntimeException("Bad URL " + request.getUrl(), e);
            } catch (IOException e) {
                int statusCode = 0;
                NetworkResponse networkResponse = null;
                if (httpResponse != null) {
                    statusCode = httpResponse.getStatusLine().getStatusCode();
                } else {
                    throw new NoConnectionError(e);
                }
                if (statusCode == HttpStatus.SC_MOVED_PERMANENTLY || 
                		statusCode == HttpStatus.SC_MOVED_TEMPORARILY) {
                	VolleyLog.e("Request at %s has been redirected to %s", request.getOriginUrl(), request.getUrl());
                } else {
                	VolleyLog.e("Unexpected response code %d for %s", statusCode, request.getUrl());
                }
                if (responseContents != null) {
                    networkResponse = new NetworkResponse(statusCode, responseContents,
                            responseHeaders, false, SystemClock.elapsedRealtime() - requestStart);
                    if (statusCode == HttpStatus.SC_UNAUTHORIZED ||
                            statusCode == HttpStatus.SC_FORBIDDEN) {
                        attemptRetryOnException("auth",
                                request, new AuthFailureError(networkResponse));
                    } else if (statusCode == HttpStatus.SC_MOVED_PERMANENTLY || 
                    			statusCode == HttpStatus.SC_MOVED_TEMPORARILY) {
                        attemptRetryOnException("redirect",
                                request, new AuthFailureError(networkResponse));
                    } else {
                        // TODO: Only throw ServerError for 5xx status codes.
                        throw new ServerError(networkResponse);
                    }
                } else {
                    throw new NetworkError(networkResponse);
                }
            }
        }
    }
```
上面调用了mHttpStack.performRequest的方法，这里就以9及以上的HurlStack类来看下源码：   
```java
@Override
public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
		throws IOException, AuthFailureError {
	String url = request.getUrl();
	HashMap<String, String> map = new HashMap<String, String>();
	map.putAll(request.getHeaders());
	map.putAll(additionalHeaders);
	if (mUrlRewriter != null) {
		String rewritten = mUrlRewriter.rewriteUrl(url);
		if (rewritten == null) {
			throw new IOException("URL blocked by rewriter: " + url);
		}
		url = rewritten;
	}
	URL parsedUrl = new URL(url);
	HttpURLConnection connection = openConnection(parsedUrl, request);
	for (String headerName : map.keySet()) {
		connection.addRequestProperty(headerName, map.get(headerName));
	}
	setConnectionParametersForRequest(connection, request);
	// Initialize HttpResponse with data from the HttpURLConnection.
	ProtocolVersion protocolVersion = new ProtocolVersion("HTTP", 1, 1);
	int responseCode = connection.getResponseCode();
	if (responseCode == -1) {
		// -1 is returned by getResponseCode() if the response code could not be retrieved.
		// Signal to the caller that something was wrong with the connection.
		throw new IOException("Could not retrieve response code from HttpUrlConnection.");
	}
	StatusLine responseStatus = new BasicStatusLine(protocolVersion,
			connection.getResponseCode(), connection.getResponseMessage());
	BasicHttpResponse response = new BasicHttpResponse(responseStatus);
	response.setEntity(entityFromConnection(connection));
	for (Entry<String, List<String>> header : connection.getHeaderFields().entrySet()) {
		if (header.getKey() != null) {
			Header h = new BasicHeader(header.getKey(), header.getValue().get(0));
			response.addHeader(h);
		}
	}
	return response;
}
```

接下来还要看一下`mDelivery.postResponse(request, response);`这里的,mDelivery就是`new ExecutorDelivery(new Handler(Looper.getMainLooper()))`
```java
/**
 * Delivers responses and errors.
 */
public class ExecutorDelivery implements ResponseDelivery {
    /** Used for posting responses, typically to the main thread. */
    private final Executor mResponsePoster;

    /**
     * Creates a new response delivery interface.
     * @param handler {@link Handler} to post responses on
     */
    public ExecutorDelivery(final Handler handler) {
        // Make an Executor that just wraps the handler.
        mResponsePoster = new Executor() {
            @Override
            public void execute(Runnable command) {
                handler.post(command);
            }
        };
    }

    /**
     * Creates a new response delivery interface, mockable version
     * for testing.
     * @param executor For running delivery tasks
     */
    public ExecutorDelivery(Executor executor) {
        mResponsePoster = executor;
    }

    @Override
    public void postResponse(Request<?> request, Response<?> response) {
        postResponse(request, response, null);
    }

    @Override
    public void postResponse(Request<?> request, Response<?> response, Runnable runnable) {
        request.markDelivered();
        request.addMarker("post-response");
        // 内部会调用execute方法
        mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));
    }

    @Override
    public void postError(Request<?> request, VolleyError error) {
        request.addMarker("post-error");
        Response<?> response = Response.error(error);
        mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, null));
    }

    /**
     * A Runnable used for delivering network responses to a listener on the
     * main thread.
     */
    @SuppressWarnings("rawtypes")
    private class ResponseDeliveryRunnable implements Runnable {
        private final Request mRequest;
        private final Response mResponse;
        private final Runnable mRunnable;

        public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {
            mRequest = request;
            mResponse = response;
            mRunnable = runnable;
        }

        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            // 这里就是重点部分了
            // If this request has canceled, finish it and don't deliver.
            if (mRequest.isCanceled()) {
                mRequest.finish("canceled-at-delivery");
                return;
            }
            // 调用mRequest的deliverResponse或者deliverError进行分发
            // Deliver a normal response or error, depending.
            if (mResponse.isSuccess()) {
                mRequest.deliverResponse(mResponse.result);
            } else {
                mRequest.deliverError(mResponse.error);
            }

            // If this is an intermediate response, add a marker, otherwise we're done
            // and the request can be finished.
            if (mResponse.intermediate) {
                mRequest.addMarker("intermediate-response");
            } else {
                // 执行finish方法
                mRequest.finish("done");
            }

            // If we have been provided a post-delivery runnable, run it.
            if (mRunnable != null) {
                mRunnable.run();
            }
       }
    }
}

```
这里再看一下mRequest的deliverResponse方法。     
Request接口中没有实现该方法，具体我们以StringRequest为例看一下： 
```java
/**
 * A canned request for retrieving the response body at a given URL as a String.
 */
public class StringRequest extends Request<String> {
    private final Listener<String> mListener;

    /**
     * Creates a new request with the given method.
     *
     * @param method the request {@link Method} to use
     * @param url URL to fetch the string at
     * @param listener Listener to receive the String response
     * @param errorListener Error listener, or null to ignore errors
     */
    public StringRequest(int method, String url, Listener<String> listener,
            ErrorListener errorListener) {
        super(method, url, errorListener);
        mListener = listener;
    }

    /**
     * Creates a new GET request.
     *
     * @param url URL to fetch the string at
     * @param listener Listener to receive the String response
     * @param errorListener Error listener, or null to ignore errors
     */
    public StringRequest(String url, Listener<String> listener, ErrorListener errorListener) {
        this(Method.GET, url, listener, errorListener);
    }

    @Override
    protected void deliverResponse(String response) {
        // 回调
        mListener.onResponse(response);
    }

    @Override
    protected Response<String> parseNetworkResponse(NetworkResponse response) {
        String parsed;
        try {
            parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
        } catch (UnsupportedEncodingException e) {
            parsed = new String(response.data);
        }
        return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
    }
}

```


接着看一下mRequest.finish方法的实现:    
```java
/**
 * Notifies the request queue that this request has finished (successfully or with error).
 *
 * <p>Also dumps all events from this request's event log; for debugging.</p>
 */
void finish(final String tag) {
    if (mRequestQueue != null) {
        // 内部调用了mRequestQueue的finish方法，也就是把请求从请求队列中移除。
        mRequestQueue.finish(this);
    }
    if (MarkerLog.ENABLED) {
        final long threadId = Thread.currentThread().getId();
        if (Looper.myLooper() != Looper.getMainLooper()) {
            // If we finish marking off of the main thread, we need to
            // actually do it on the main thread to ensure correct ordering.
            Handler mainThread = new Handler(Looper.getMainLooper());
            mainThread.post(new Runnable() {
                @Override
                public void run() {
                    mEventLog.add(tag, threadId);
                    mEventLog.finish(this.toString());
                }
            });
            return;
        }

        mEventLog.add(tag, threadId);
        mEventLog.finish(this.toString());
    } else {
        long requestTime = SystemClock.elapsedRealtime() - mRequestBirthTime;
        if (requestTime >= SLOW_REQUEST_THRESHOLD_MS) {
            VolleyLog.d("%d ms: %s", requestTime, this.toString());
        }
    }
}
```

到这里就全部分析完了。


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
