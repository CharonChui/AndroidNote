Retrofit详解(下)
===


上一篇文件介绍了`Retrofit`的基本使用，接下来我们通过从源码的角度分析一下`Retrofit`的实现。    

首先看一下它的基本使用方法:    

```java
// 1
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build();
// 2
GitHubService gitHubService = retrofit.create(GitHubService.class);

// 3
Call<List<Repo>> call = gitHubService.listRepos("CharonChui");

// 4
call.enqueue(new Callback<List<Repo>>() {
    @Override
    public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
        List<Repo> data = response.body();
        Log.i("@@@", "data size : " + (data == null ? "null" : data.size() + ""));
    }

    @Override
    public void onFailure(Call<List<Repo>> call, Throwable t) {

    }
});
```

我把上面主要分为4个部分，接下来逐一分析:    

1. 创建`Retrofit`并进行配置。
---

```java
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com/")
  .addConverterFactory(GsonConverterFactory.create())
  .build();
```

简单的一句话，却埋藏了很多。     

这是典型的***建造者模式、外观模式***      

就想平时我们写的下载模块，作为一个公共的模块，我们可以对外提供一个`DownloadManager`供外界使用，而对于里面的实现我们完全可以闭门造车。    

具体`baseUrl()`、`addConverterFactory()`方法里面的具体实现就不去看了，比较简单。当然这里也用到了***工厂设计模式***。

2. 创建对应的服务类
---

```java
GitHubService gitHubService = retrofit.create(GitHubService.class);
```

这一部分是如何实现的呢？我们看一下`retrofit.create()`方法的实现:    

```java
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          // 这里的Platform主要是为了检测当前的运行平台，是java还是android，会根据当前的平台来返回默认的CallAdapter
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              // 代理调用
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            // 1，根据动态代理的方法去生成ServiceMethod这里动态代理的方法就是listRepos方法
            ServiceMethod serviceMethod = loadServiceMethod(method);
            // 2，根绝ServiceMethod和参数去生成OkHttpCall，这里args是CharonChui
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            // 3, serviceMethod去进行处理并返回Call对象，拿到这个Call对象才能去执行网络请求。
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
```

看到`Proxy.newProxyInstance()`就明白了，这里使用了***动态代理***。简单的说动态代理是在你要调用某个`Class`的方法前或后，插入你想要执行的代码。那这里要代理的是什么方法？ `Call<List<Repo>> call = gitHubService.listRepos("CharonChui");`，这里就是`listRepos()`方法。   就是说在调用`listRepos()`方法时会被动态代理所拦截，然后执行`Proxy.newProxyInstance()`里面的`InvocationHandler.invoke()`中的部分。   而`invoke()`方法的三个参数分别是啥？ 分别是`Object proxy`: 代理对象，`Method method`：调用的方法，就是`listRepos()`方法，`Object... args`：方法的参数，这里是`CharonChui`。   

有关动态代理介绍可以看[张孝祥老师的java1.5高新技术系列中的动态代理]()

这里就不仔细介绍动态代理了，上面的代码中又分为三部分:    

- `loadServiceMethod()`
- `new OkHttpCall()`
- `serviceMethod.callAdapter.adapt()`

我们这里分别来进行分析。    

- `loadServiceMethod()`
	实现如下:   
	
	```java
	ServiceMethod loadServiceMethod(Method method) {
	    ServiceMethod result;
	    synchronized (serviceMethodCache) {
	      // ServiceMethod包含了请求的所有相关数据，以及获取请求的request和把请求结果转换成java对象，所以相比较而言较重，用缓存来提高效率。
	      result = serviceMethodCache.get(method);
	      if (result == null) {
	        // build
	        result = new ServiceMethod.Builder(this, method).build();
	        serviceMethodCache.put(method, result);
	      }
	    }
	    return result;
	  }
	```
	
	会通过缓存的方式来获取一个`ServiceMethod`类。通过缓存来保证同一个`API`的同一个方法只会创建一次。 
	有关`ServiceMethod`类的文档介绍是:     
	```java
	Adapts an invocation of an interface method into an HTTP call.
	```
	大体翻译一下就是将一个请求接口的方法转换到`Http Call`中调用。    
	
	而上面第一次使用的时候会通过`new ServiceMethod.Builder(this, method).build()`创建，那我们看一下它的实现:    
	
	```java
	public ServiceMethod build() {
	       // 创建CallAdapter用来代理Call
	      callAdapter = createCallAdapter();
	      responseType = callAdapter.responseType();
	      if (responseType == Response.class || responseType == okhttp3.Response.class) {
	        throw methodError("'"
	            + Utils.getRawType(responseType).getName()
	            + "' is not a valid response body type. Did you mean ResponseBody?");
	      }
	      // responseConverter用来解析结果将json等返回结果解析成java对象
	      responseConverter = createResponseConverter();
	
	      for (Annotation annotation : methodAnnotations) {
	        parseMethodAnnotation(annotation);
	      }
	
	      if (httpMethod == null) {
	        throw methodError("HTTP method annotation is required (e.g., @GET, @POST, etc.).");
	      }
	
	      if (!hasBody) {
	        if (isMultipart) {
	          throw methodError(
	              "Multipart can only be specified on HTTP methods with request body (e.g., @POST).");
	        }
	        if (isFormEncoded) {
	          throw methodError("FormUrlEncoded can only be specified on HTTP methods with "
	              + "request body (e.g., @POST).");
	        }
	      }
	
	      int parameterCount = parameterAnnotationsArray.length;
	      parameterHandlers = new ParameterHandler<?>[parameterCount];
	      for (int p = 0; p < parameterCount; p++) {
	        Type parameterType = parameterTypes[p];
	        if (Utils.hasUnresolvableType(parameterType)) {
	          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
	              parameterType);
	        }
			//  解析对应method的注解，这里是listRepos方法的注解。
	        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
	        if (parameterAnnotations == null) {
	          throw parameterError(p, "No Retrofit annotation found.");
	        }
	        // 通过注解和参数类型，解析并赋值到parameterHandlers中
	        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
	      }
	
	      if (relativeUrl == null && !gotUrl) {
	        throw methodError("Missing either @%s URL or @Url parameter.", httpMethod);
	      }
	      if (!isFormEncoded && !isMultipart && !hasBody && gotBody) {
	        throw methodError("Non-body HTTP method cannot contain @Body.");
	      }
	      if (isFormEncoded && !gotField) {
	        throw methodError("Form-encoded method must contain at least one @Field.");
	      }
	      if (isMultipart && !gotPart) {
	        throw methodError("Multipart method must contain at least one @Part.");
	      }
	      // 创建`ServiceMethod()`对象
	      return new ServiceMethod<>(this);
	    }
	```
	总起来说，就是创建`CallAdapter`、`responseConverter`、解析注解、设置参数，然后创建`ServiceMethod`对象。   
	
	而`ServiceMethod`的构造函数如下:    
	```java
	ServiceMethod(Builder<T> builder) {
	    this.callFactory = builder.retrofit.callFactory();
	    this.callAdapter = builder.callAdapter;
	    this.baseUrl = builder.retrofit.baseUrl();
	    this.responseConverter = builder.responseConverter;
	    this.httpMethod = builder.httpMethod;
	    this.relativeUrl = builder.relativeUrl;
	    this.headers = builder.headers;
	    this.contentType = builder.contentType;
	    this.hasBody = builder.hasBody;
	    this.isFormEncoded = builder.isFormEncoded;
	    this.isMultipart = builder.isMultipart;
	    this.parameterHandlers = builder.parameterHandlers;
	  }
	```
	
	看到吗？ 这一部分我们应该都稍微有点印象，因为在上一篇文章介绍使用方法的时候，基本会用到这里。 

至于这里的`CallAdapter`、`ResponseConverter`、`Headers`、`ParamterHandlers`等这里就不分析了，最后我们再简单介绍下。

- 创建`OkHttpCall`

	接下来会创建`OkHttpCall`，而`OkHttpCall`是`Call`的子类，那`Call`是什么鬼？ 它是具体的网络请求类。
	
	文档中对`Call`类的介绍如下:    
	```java
	/**
	 * An invocation of a Retrofit method that sends a request to a webserver and returns a response.
	 * Each call yields its own HTTP request and response pair. Use {@link #clone} to make multiple
	 * calls with the same parameters to the same webserver; this may be used to implement polling or
	 * to retry a failed call.
	 *
	 * <p>Calls may be executed synchronously with {@link #execute}, or asynchronously with {@link
	 * #enqueue}. In either case the call can be canceled at any time with {@link #cancel}. A call that
	 * is busy writing its request or reading its response may receive a {@link IOException}; this is
	 * working as designed.
	 *
	 * @param <T> Successful response body type.
	 */
	public interface Call<T> extends Cloneable {
		// 这里我特地把这句话放上，Call集成了Cloneable接口。
		// 每一个 call 对象实例只能被用一次，所以说 request 和 response 都是一一对应的。你其实可以通过 Clone 方法来创建一个一模一样的实例，这个开销是很小的。比如说：你可以在每次决定发请求前 clone 一个之前的实例。
		....
	}
	```
	
	`Retrofit`底层默认使用`OkHttp`，所以当然要创建`OkHttpCall`了。    
	
	- `serviceMethod.callAdapter.adapt(okHttpCall)`
	
	这个`CallApdater`是什么鬼？   
	
	```java
	/**
	 * Adapts a {@link Call} into the type of {@code T}. Instances are created by {@linkplain Factory a
	 * factory} which is {@linkplain Retrofit.Builder#addCallAdapterFactory(Factory) installed} into
	 * the {@link Retrofit} instance.
	 */
	```
	
	可以很简单的看出来这是一个结果类型转换的类。就是`Call`的适配器，作用就是创建/转换`Call`对象，把`Call`转换成预期的格式。`CallAdatper`创建是通过`CallAdapter.factory`工厂类进行的。`DefaultCallAdapter`为`Retrofit2`自带默认`Call`转换器，用来生成`OKHTTP`的`call`请求调用。

	
	而它里面的`adapt()`方法的作用呢？ 
	
	```java
	/**
	   * Returns an instance of {@code T} which delegates to {@code call}.
	   * <p>
	   * For example, given an instance for a hypothetical utility, {@code Async}, this instance would
	   * return a new {@code Async<R>} which invoked {@code call} when run.
	   * <pre><code>
	   * &#64;Override
	   * public &lt;R&gt; Async&lt;R&gt; adapt(final Call&lt;R&gt; call) {
	   *   return Async.create(new Callable&lt;Response&lt;R&gt;&gt;() {
	   *     &#64;Override
	   *     public Response&lt;R&gt; call() throws Exception {
	   *       return call.execute();
	   *     }
	   *   });
	   * }
	   * </code></pre>
	   */
	  <R> T adapt(Call<R> call);
	```
	
	所以分析到这里我们基本明白了`retrofit.create()`方法的作用，就是将请求接口的服务类转换成`Call`，然后将`Call`的结果转换成实体类。
	
3. 调用方法，得到`Call`对象
---

```java
Call<List<Repo>> call = gitHubService.listRepos("CharonChui");
```

这个就不分析了，就是在`ServiceMethod`中返回的`Call`。
 
4. 调用`Call.enqueue()`
---

在上面分析了，这个`Call`其实是`OkHttpCall`，那我们来看一下`OkHttpCall.enqueue()`方法的实现:     

```java
@Override public void enqueue(final Callback<T> callback) {
    if (callback == null) throw new NullPointerException("callback == null");

    okhttp3.Call call;
    Throwable failure;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      call = rawCall;
      failure = creationFailure;
      if (call == null && failure == null) {
        try {
          // 调用createRawCall()方法创建Call
          call = rawCall = createRawCall();
        } catch (Throwable t) {
          failure = creationFailure = t;
        }
      }
    }

    if (failure != null) {
      callback.onFailure(this, failure);
      return;
    }

    if (canceled) {
      call.cancel();
    }
    // 把请求任务加入到okhttp的请求队列中，执行网络请求，注意这里的Call是okhttp3.Call
    call.enqueue(new okhttp3.Callback() {
      @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
          throws IOException {
        Response<T> response;
        try {
          // 解析结果，该方法内部会将OkHttp中Request的执行结果转换成对应的Java对象。
          response = parseResponse(rawResponse);
        } catch (Throwable e) {
          callFailure(e);
          return;
        }
        callSuccess(response);
      }

      @Override public void onFailure(okhttp3.Call call, IOException e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callFailure(Throwable e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callSuccess(Response<T> response) {
        try {
          callback.onResponse(OkHttpCall.this, response);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }
    });
  }
```

上面的部分也主要分为三部分:    

- 创建Call
- 执行网络请求
- 解析结果 

#### 第一部分：创建`Call`

我们分别进行分析，首先是`createRawCall()`方法的实现:    
```java
private okhttp3.Call createRawCall() throws IOException {
  // ServiceMethod.toRequest()方法的作用是将ServiceMethod中的网络请求相关的数据转换成一个OkHttp的网络请求所需要的Request对象。因为之前分析过所有Retrofit解析的网络请求相关的数据都是在ServiceMethod中
  Request request = serviceMethod.toRequest(args);
  // 调用Factory.newCall方法
  okhttp3.Call call = serviceMethod.callFactory.newCall(request);
  if (call == null) {
    throw new NullPointerException("Call.Factory returned null.");
  }
  return call;
}
```

然后看一下`Factory.newCall()`方法:  
```java
interface Factory {
    Call newCall(Request request);
  }
```

它的实现类是`OkHttpClient`类中的`newCall()`方法，并且创建`RealCall`对象:    
```java
@Override public Call newCall(Request request) {
    return new RealCall(this, request);
  }
```

#### 第二部分：执行网络请求

这一部分是在`call.enqueue()`方法中执行的，上面我们分析了创建的`Call`最终是`RealCall`类，所以这里直接到看`RealCall.enqueue()`方法:    

```java
@Override public void enqueue(Callback responseCallback) {
    enqueue(responseCallback, false);
  }

  void enqueue(Callback responseCallback, boolean forWebSocket) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }

    // 调用OkHttpClient中的Dispatcher中的enqueue方法
    client.dispatcher().enqueue(new AsyncCall(responseCallback, forWebSocket));
  }
```

我们接着看`client.dispatcher().enqueue()`方法:

```java
synchronized void enqueue(AsyncCall call) {
    // maxRequests的个数是64;
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      // 线程池执行
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }
```   

而这个参数`AsyncCall`是什么鬼？ 它是`RealCall`的内部类，它里面的`execute()`方法是如何实现的？ 只要找到该方法的实现就算是完成了。     

首先看一下`AsyncCall`的声明和构造:    
```java
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;
    private final boolean forWebSocket;

    private AsyncCall(Callback responseCallback, boolean forWebSocket) {
      super("OkHttp %s", redactedUrl().toString());
      this.responseCallback = responseCallback;
      this.forWebSocket = forWebSocket;
    }
    ...
  }
```

`NamedRunnable`是`Runnable`的实现类。我们看一下它的`execute()`方法:   
```java
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        // 获取请求结果
        Response response = getResponseWithInterceptorChain(forWebSocket);
        if (canceled) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          // 将响应结果设置给之前构造函数传递回来的回调
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```

接着看一下`RealCall.getResponseWithInterceptorChain()`:    

```java
private Response getResponseWithInterceptorChain(boolean forWebSocket) throws IOException {
    Interceptor.Chain chain = new ApplicationInterceptorChain(0, originalRequest, forWebSocket);
    return chain.proceed(originalRequest);
  }

  class ApplicationInterceptorChain implements Interceptor.Chain {
    private final int index;
    private final Request request;
    private final boolean forWebSocket;

    ApplicationInterceptorChain(int index, Request request, boolean forWebSocket) {
      this.index = index;
      this.request = request;
      this.forWebSocket = forWebSocket;
    }

    @Override public Connection connection() {
      return null;
    }

    @Override public Request request() {
      return request;
    }

    @Override public Response proceed(Request request) throws IOException {
      // If there's another interceptor in the chain, call that.
      if (index < client.interceptors().size()) {
        Interceptor.Chain chain = new ApplicationInterceptorChain(index + 1, request, forWebSocket);
        Interceptor interceptor = client.interceptors().get(index);
        Response interceptedResponse = interceptor.intercept(chain);

        if (interceptedResponse == null) {
          throw new NullPointerException("application interceptor " + interceptor
              + " returned null");
        }

        return interceptedResponse;
      }

      // No more interceptors. Do HTTP.
      return getResponse(request, forWebSocket);
    }
  }
```

又会调用`getResponse()`方法：    
```java
/**
   * Performs the request and returns the response. May return null if this call was canceled.
   */
  Response getResponse(Request request, boolean forWebSocket) throws IOException {
    // Copy body metadata to the appropriate request headers.
    RequestBody body = request.body();
    if (body != null) {
      Request.Builder requestBuilder = request.newBuilder();

      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }

      request = requestBuilder.build();
    }

    // Create the initial HTTP engine. Retries and redirects need new engine for each attempt.
    engine = new HttpEngine(client, request, false, false, forWebSocket, null, null, null);

    int followUpCount = 0;
    while (true) {
      if (canceled) {
        engine.releaseStreamAllocation();
        throw new IOException("Canceled");
      }

      boolean releaseConnection = true;
      try {
        engine.sendRequest();
        engine.readResponse();
        releaseConnection = false;
      } catch (RequestException e) {
        // The attempt to interpret the request failed. Give up.
        throw e.getCause();
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        HttpEngine retryEngine = engine.recover(e.getLastConnectException(), true, null);
        if (retryEngine != null) {
          releaseConnection = false;
          engine = retryEngine;
          continue;
        }
        // Give up; recovery is not possible.
        throw e.getLastConnectException();
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        HttpEngine retryEngine = engine.recover(e, false, null);
        if (retryEngine != null) {
          releaseConnection = false;
          engine = retryEngine;
          continue;
        }

        // Give up; recovery is not possible.
        throw e;
      } finally {
        // We're throwing an unchecked exception. Release any resources.
        if (releaseConnection) {
          StreamAllocation streamAllocation = engine.close();
          streamAllocation.release();
        }
      }

      Response response = engine.getResponse();
      Request followUp = engine.followUpRequest();

      if (followUp == null) {
        if (!forWebSocket) {
          engine.releaseStreamAllocation();
        }
        return response;
      }

      StreamAllocation streamAllocation = engine.close();

      if (++followUpCount > MAX_FOLLOW_UPS) {
        streamAllocation.release();
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }

      if (!engine.sameConnection(followUp.url())) {
        streamAllocation.release();
        streamAllocation = null;
      } else if (streamAllocation.stream() != null) {
        throw new IllegalStateException("Closing the body of " + response
            + " didn't close its backing stream. Bad interceptor?");
      }

      request = followUp;
      engine = new HttpEngine(client, request, false, false, forWebSocket, streamAllocation, null,
          response);
    }
  }
```

完了没有？ 完了....


上面只是简单的分析了下大体的调用流程和主要的类，但是好像并没有什么乱用，因为没有具体的去分析里面各部分的实现，如果都分析下来内容太多了。这里就不仔细看了，大体总结一下。    

通过上面的分析，最终的网络请求是在`OkHttp`的`Call`中去执行，也就是说`Retrofit`其实是将一个`Java`接口通过注解等方式来解析参数等然后转换成一个请求交给`OkHttp`去执行，然后将执行结果进行解析转换暴露给上层调用者。 
而这一切是如何实现的呢？ 
`Retrofit`非常巧妙的用注解来描述一个`HTTP`请求，将一个`HTTP`请求抽象成一个`Java`接口，然后用了`动态代理`的方式，动态的将这个接口的注解转换成一个`HTTP`请求，然后再将这个`Http`请求交给`OkHttp`执行。

动态代理用的太妙，而它的过程中也使用了大量的工厂模式，这里就不分析了。


参考:   
- [simple-http-retrofit-2](https://realm.io/news/droidcon-jake-wharton-simple-http-retrofit-2)
- [Retrofit API](http://square.github.io/retrofit/2.x/retrofit/)


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
