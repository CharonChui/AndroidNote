Android HttpURLConnection详解
---

之前写过HttpURLConnection与HttpClient的区别及选择。后来又分析了Volley的源码。
最近又遇到了问题，想在Volley中针对HttpURLConnection添加连接池的功能，虽然感觉这是没必要的，但是心底确拿不出事实证明。所以研究下HttpURLConnection的源码进行分析。


在使用的时候都是通过URL.openConnection()来获取`HttpURLConnection`对象，然后调用其`connect`方法进行链接，所以先从`URL.penConnection()`入手:    
```java
    /**
     * Returns a new connection to the resource referred to by this URL.
     *
     * @throws IOException if an error occurs while opening the connection.
     */
    public URLConnection openConnection() throws IOException {
        return streamHandler.openConnection(this);
    }
```

接下来就要看一下`streamHandler`究竟是何方神圣？我们搜一下他的赋值，实在`setupStreamHandler`方法中进行的：    
```java
    /**
     * Sets the receiver's stream handler to one which is appropriate for its
     * protocol.
     *
     * <p>Note that this will overwrite any existing stream handler with the new
     * one. Senders must check if the streamHandler is null before calling the
     * method if they do not want this behavior (a speed optimization).
     *
     * @throws MalformedURLException if no reasonable handler is available.
     */
    void setupStreamHandler() {
        // Check for a cached (previously looked up) handler for
        // the requested protocol.
        streamHandler = streamHandlers.get(protocol);
        if (streamHandler != null) {
            return;
        }

        // If there is a stream handler factory, then attempt to
        // use it to create the handler.
        if (streamHandlerFactory != null) {
            streamHandler = streamHandlerFactory.createURLStreamHandler(protocol);
            if (streamHandler != null) {
                streamHandlers.put(protocol, streamHandler);
                return;
            }
        }

        // Check if there is a list of packages which can provide handlers.
        // If so, then walk this list looking for an applicable one.
        String packageList = System.getProperty("java.protocol.handler.pkgs");
        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
        if (packageList != null && contextClassLoader != null) {
            for (String packageName : packageList.split("\\|")) {
                String className = packageName + "." + protocol + ".Handler";
                try {
                    Class<?> c = contextClassLoader.loadClass(className);
                    streamHandler = (URLStreamHandler) c.newInstance();
                    if (streamHandler != null) {
                        streamHandlers.put(protocol, streamHandler);
                    }
                    return;
                } catch (IllegalAccessException ignored) {
                } catch (InstantiationException ignored) {
                } catch (ClassNotFoundException ignored) {
                }
            }
        }

        // Fall back to a built-in stream handler if the user didn't supply one
        if (protocol.equals("file")) {
            streamHandler = new FileHandler();
        } else if (protocol.equals("ftp")) {
            streamHandler = new FtpHandler();
        } else if (protocol.equals("http")) {
            // 判断一下如果是HTTP协议，就会创建HtppHandler。看到这里明白了，原来使用的是okhttp.
            try {
                String name = "com.android.okhttp.HttpHandler";
                streamHandler = (URLStreamHandler) Class.forName(name).newInstance();
            } catch (Exception e) {
                throw new AssertionError(e);
            }
        } else if (protocol.equals("https")) {
            try {
                String name = "com.android.okhttp.HttpsHandler";
                streamHandler = (URLStreamHandler) Class.forName(name).newInstance();
            } catch (Exception e) {
             throw new AssertionError(e);
            }
        } else if (protocol.equals("jar")) {
            streamHandler = new JarHandler();
        }
        if (streamHandler != null) {
            streamHandlers.put(protocol, streamHandler);
        }
    }
```
这里我们就以`HTTP`协议为例说一下所以找到`okhttp HttpHandler.openConnection()`方法:     
```java    /**
     * Closes the socket. It is not possible to reconnect or rebind to this
     * socket thereafter which means a new socket instance has to be created.
     *
     * @throws IOException
     *             if an error occurs while closing the socket.
     */
    public synchronized void close() throws IOException {
        isClosed = true;
        isConnected = false;
        // RI compatibility: the RI returns the any address (but the original local port) after
        // close.
        localAddress = Inet4Address.ANY;
        impl.close();
    }
/*
 *  Licensed to the Apache Software Foundation (ASF) under one or more
 *  contributor license agreements.  See the NOTICE file distributed with
 *  this work for additional information regarding copyright ownership.
 *  The ASF licenses this file to You under the Apache License, Version 2.0
 *  (the "License"); you may not use this file except in compliance with
 *  the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 */
package com.squareup.okhttp;
import java.io.IOException;
import java.net.Proxy;
import java.net.URL;
import java.net.URLConnection;
import java.net.URLStreamHandler;
public final class HttpHandler extends URLStreamHandler {
    @Override protected URLConnection openConnection(URL url) throws IOException {
        // 调用了OKHttpClient()的方法
        return new OkHttpClient().open(url);
    }
    @Override protected URLConnection openConnection(URL url, Proxy proxy) throws IOException {
        if (url == null || proxy == null) {
            throw new IllegalArgumentException("url == null || proxy == null");
        }
        return new OkHttpClient().setProxy(proxy).open(url);
    }
    @Override protected int getDefaultPort() {
        return 80;
    }
}
```

接下来就悲剧了，因为我找不到OkHttpClient()类中有`open`方法。               
仔细查看了文档后发现在`OKHttp`1.6.0的时候该方法就已经已经过时了。    
```java
@Deprecated
public HttpURLConnection open(URL url)
Deprecated. moved to OkUrlFactory.open.
```

那我们怎么往下分析呢？很显然`Android sdk`中使用的`OkHttp`不是最新版。所以我们可以使用`1.5.0`版本的`OKHttp`接着分析。
在项目`build.gradle`中进行配置 compile 'com.squareup.okhttp:okhttp:1.5.0' 然后开始愉快的查看源码。    

```java
  public HttpURLConnection open(URL url) {
    return open(url, proxy);
  }

  HttpURLConnection open(URL url, Proxy proxy) {
    String protocol = url.getProtocol();
    OkHttpClient copy = copyWithDefaults();
    copy.proxy = proxy;

    // 返回了HttpURLConnectionImpl
    if (protocol.equals("http")) return new HttpURLConnectionImpl(url, copy);
    if (protocol.equals("https")) return new HttpsURLConnectionImpl(url, copy);
    throw new IllegalArgumentException("Unexpected protocol: " + protocol);
  }

```
接着看一下`HttpURLConnectionImpl`类,它是`HttpURLConnection`的子类：     
```java
public class HttpURLConnectionImpl extends HttpURLConnection {
}
```
到这里`new URL(url).openConnection()`方法已经分析完了。    

我们在使用`HttpURLConnection`都是这样使用：    
```java
String url = "http://www.baidu.com"
URL url = new URL(url);
HttpURLConnection connection = (HttpURLConnection)url.openConnection();
// 设置一些请求头等参数
...
connection.connect();
// 然后调用一些其他的获取结果或者状态的方法。
...
connection.getResponseCode();
connection.getOuoputStream();
connection.getInputStream();
....
```

上面分析了`new URL().openConnection()`那我们这里就接着分析第二步了，就是调用`connect()`方法的处理:      
这里看一下`HttpURLConnectionImpl.connect()`方法：   
```java
  @Override public final void connect() throws IOException {
    initHttpEngine();
    boolean success;
    do {
      success = execute(false);
    } while (!success);
  }
```

接着看一下`initHttpEngine()`方法的实现: 
```java
private void initHttpEngine() throws IOException {
    if (httpEngineFailure != null) {
      throw httpEngineFailure;
    } else if (httpEngine != null) {
      return;
    }

    connected = true;
    try {
      if (doOutput) {
        if (method.equals("GET")) {
          // they are requesting a stream to write to. This implies a POST method
          method = "POST";
        } else if (!HttpMethod.hasRequestBody(method)) {
          // If the request method is neither POST nor PUT nor PATCH, then you're not writing
          throw new ProtocolException(method + " does not support writing");
        }
      }

      // 将newHttpEngine方法的返回值赋值给HttpEngine的成员变量。
      httpEngine = newHttpEngine(method, null, null);
    } catch (IOException e) {
      httpEngineFailure = e;
      throw e;
    }
  }

  private HttpEngine newHttpEngine(String method, Connection connection,
      RetryableSink requestBody) {
    Request.Builder builder = new Request.Builder()
        .url(getURL())
        .method(method, null /* No body; that's passed separately. */);
    Headers headers = requestHeaders.build();
    for (int i = 0; i < headers.size(); i++) {
      builder.addHeader(headers.name(i), headers.value(i));
    }

    boolean bufferRequestBody;
    if (fixedContentLength != -1) {
      bufferRequestBody = false;
      builder.header("Content-Length", Long.toString(fixedContentLength));
    } else if (chunkLength > 0) {
      bufferRequestBody = false;
      builder.header("Transfer-Encoding", "chunked");
    } else {
      bufferRequestBody = true;
    }

    Request request = builder.build();

    // If we're currently not using caches, make sure the engine's client doesn't have one.
    OkHttpClient engineClient = client;
    if (engineClient.getOkResponseCache() != null && !getUseCaches()) {
      engineClient = client.clone().setOkResponseCache(null);
    }

    return new HttpEngine(engineClient, request, bufferRequestBody, connection, null, requestBody);
  }

```

到这里我们知道他会创建一个`HttpEngine`类，我们先不管它，接着看一下`execute()`方法的内部实现： 
```java
  /**
   * Sends a request and optionally reads a response. Returns true if the
   * request was successfully executed, and false if the request can be
   * retried. Throws an exception if the request failed permanently.
   */
  private boolean execute(boolean readResponse) throws IOException {
    try {
      // 调用了HttpEngine的sendRequest方法。
      httpEngine.sendRequest();
      route = httpEngine.getRoute();
      handshake = httpEngine.getConnection() != null
          ? httpEngine.getConnection().getHandshake()
          : null;
      // 读取结果，我们先不分析这里，等把sendRequest部分全部分析完成后再回来分析readResponse()部分。
      if (readResponse) {
        httpEngine.readResponse();
      }

      return true;
    } catch (IOException e) {
      HttpEngine retryEngine = httpEngine.recover(e);
      if (retryEngine != null) {
        httpEngine = retryEngine;
        return false;
      }

      // Give up; recovery is not possible.
      httpEngineFailure = e;
      throw e;
    }
  }
```
到这里，可以大胆的猜测一下了`HttpEngine`应该就是实际在`Socket`链接上进行数据收发的类。 当然这只是猜测，接着看一下它的实现:      
```java
/**
 * Handles a single HTTP request/response pair. Each HTTP engine follows this
 * lifecycle:
 * <ol>
 * <li>It is created.
 * <li>The HTTP request message is sent with sendRequest(). Once the request
 * is sent it is an error to modify the request headers. After
 * sendRequest() has been called the request body can be written to if
 * it exists.
 * <li>The HTTP response message is read with readResponse(). After the
 * response has been read the response headers and body can be read.
 * All responses have a response body input stream, though in some
 * instances this stream is empty.
 * </ol>
 *
 * <p>The request and response may be served by the HTTP response cache, by the
 * network, or by both in the event of a conditional GET.
 */
```
文档验证了我们的想法。因为它内部实现代码比较多，所以就不全部贴了，按照重要的一步步分析，既然上面调用了`sendRequest()`方法，这里就从他入手：　　　   
```java
  /**
   * Figures out what the response source will be, and opens a socket to that
   * source if necessary. Prepares the request headers and gets ready to start
   * writing the request body if it exists.
   */
  public final void sendRequest() throws IOException {
    if (responseSource != null) return; // Already sent.
    if (transport != null) throw new IllegalStateException();

    // 设置一些请求头，正式这个方法内部默认设置了`Keep-Alive`的值，也就是在Android Level 9之前因为Bug，我们需要关闭它的具体处理位置。   
    prepareRawRequestHeaders();
    // 处理cache
    OkResponseCache responseCache = client.getOkResponseCache();

    Response cacheResponse = responseCache != null
        ? responseCache.get(request)
        : null;
    long now = System.currentTimeMillis();
    CacheStrategy cacheStrategy = new CacheStrategy.Factory(now, request, cacheResponse).get();
    responseSource = cacheStrategy.source;
    request = cacheStrategy.request;

    if (responseCache != null) {
      responseCache.trackResponse(responseSource);
    }

    if (responseSource != ResponseSource.NETWORK) {
      validatingResponse = cacheStrategy.response;
    }

    if (cacheResponse != null && !responseSource.usesCache()) {
      closeQuietly(cacheResponse.body()); // We don't need this cached response. Close it.
    }

    if (responseSource.requiresConnection()) {
      // Open a connection unless we inherited one from a redirect.
      if (connection == null) {
        // 调用connect方法，内部会重新创建一个connection
        connect();
      }
      // 这个和后面的connection类一起看， Transport接口提供了一个用户写Request头和数据的输出流。
      transport = (Transport) connection.newTransport(this);

      // Create a request body if we don't have one already. We'll already have
      // one if we're retrying a failed POST.
      if (hasRequestBody() && requestBodyOut == null) {
        // 通过transport创建一个请求体的输出流，requestBodyOut是Sink接口的实现类，其实就是将请求头和请求体发送给服务器。这部分跟下去内容比较多，就不往下跟了。
　　　　// 到这里就已经完成了与服务器的连接功能，并且把请求内容发送给服务器。请求部分就执行完了，可以回去了，还知道开头是哪吗？哈哈。接下来的就是从服务器接口读取返回数据了。  
        requestBodyOut = transport.createRequestBody(request);
      }

    } else {
      // We're using a cached response. Recycle a connection we may have inherited from a redirect.
      if (connection != null) {
        // 回收connection，这里就看到了连接池
        client.getConnectionPool().recycle(connection);
        connection = null;
      }

      // No need for the network! Promote the cached response immediately.
      this.response = validatingResponse;
      if (validatingResponse.body() != null) {
        initContentStream(validatingResponse.body().source());
      }
    }
  }

```

接着看一下`connect()`方法：     
```java
  /** Connect to the origin server either directly or via a proxy. */
  private void connect() throws IOException {
    if (connection != null) throw new IllegalStateException();

    if (routeSelector == null) {
      String uriHost = request.url().getHost();
      if (uriHost == null || uriHost.length() == 0) {
        throw new UnknownHostException(request.url().toString());
      }
      SSLSocketFactory sslSocketFactory = null;
      HostnameVerifier hostnameVerifier = null;
      if (request.isHttps()) {
        sslSocketFactory = client.getSslSocketFactory();
        hostnameVerifier = client.getHostnameVerifier();
      }
      Address address = new Address(uriHost, getEffectivePort(request.url()), sslSocketFactory,
          hostnameVerifier, client.getAuthenticator(), client.getProxy(), client.getProtocols());
      routeSelector = new RouteSelector(address, request.uri(), client.getProxySelector(),
          client.getConnectionPool(), Dns.DEFAULT, client.getRoutesDatabase());
    }

    connection = routeSelector.next(request.method());

    if (!connection.isConnected()) {
      // connection 进行连接了啊。他里面会用Socket开始连了。。后面我们再细看。简单的说Connection管理了Socket，后面我们要重点看这个类。
      connection.connect(client.getConnectTimeout(), client.getReadTimeout(), getTunnelConfig());
      if (connection.isSpdy()) client.getConnectionPool().share(connection);
      client.getRoutesDatabase().connected(connection.getRoute());
    } else if (!connection.isSpdy()) {
      connection.updateReadTimeout(client.getReadTimeout());
    }

    route = connection.getRoute();
  }
```

再看一下`Connection`类的实现:   
```java

public final class Connection implements Closeable {
  private final ConnectionPool pool;
  private final Route route;

  private Socket socket;
  private InputStream in;
  private OutputStream out;
  private BufferedSource source;
  private BufferedSink sink;
  private boolean connected = false;
  private HttpConnection httpConnection;
  private SpdyConnection spdyConnection;
  private int httpMinorVersion = 1; // Assume HTTP/1.1
  private long idleStartTimeNs;
  private Handshake handshake;
  private int recycleCount;

  public Connection(ConnectionPool pool, Route route) {
    this.pool = pool;
    this.route = route;
  }

  public void connect(int connectTimeout, int readTimeout, TunnelRequest tunnelRequest)
      throws IOException {
    if (connected) throw new IllegalStateException("already connected");

    socket = (route.proxy.type() != Proxy.Type.HTTP) ? new Socket(route.proxy) : new Socket();
    // 连socket了
    Platform.get().connectSocket(socket, route.inetSocketAddress, connectTimeout);
    socket.setSoTimeout(readTimeout);
    in = socket.getInputStream();
    out = socket.getOutputStream();

    if (route.address.sslSocketFactory != null) {
      upgradeToTls(tunnelRequest);
    } else {
      initSourceAndSink();
      // 创建HttpConnection.A socket connection that can be used to send HTTP/1.1 messages. 
      // 这个HttpConnection有什么用呢？就是下面的newTransport方法中会用。
      httpConnection = new HttpConnection(pool, this, source, sink);
    }
    connected = true;
  }

     /** Returns the transport appropriate for this connection. */
  public Object newTransport(HttpEngine httpEngine) throws IOException {
    return (spdyConnection != null)
        ? new SpdyTransport(httpEngine, spdyConnection)
        : new HttpTransport(httpEngine, httpConnection);
  }

}
````

到这里就已经把发送请求到服务器的部分全部分析完了，就想上面所说的我们应该回去了，回去分析发送完请求后的部分。
我们这个分析是在`HttpURLConnectionImpl.execute()`方法中的`HttpEngine.sendRequest()`方法开始一直分析下来的，
所以我们还是要回到`HttpURLConnectionImpl.execute()`方法中. 
```java
  private boolean execute(boolean readResponse) throws IOException {
    try {
      // 上面已经把sendRequest部分全部分析完了，该方法会与服务器通过Socket建立连接并把请求部分发送给服务器。
      httpEngine.sendRequest();
      route = httpEngine.getRoute();
      handshake = httpEngine.getConnection() != null
          ? httpEngine.getConnection().getHandshake()
          : null;
      if (readResponse) {
        // 发送完请求之后该干什么呢？ 当然是读取返回数据了。。。 没错就是它。但是不要忘了在connect()方法传递过来的时候这个值是false。
        // 所以这一步在这里是不执行的，但是我们也分析下，方便以后理解。那这个值什么时候是true呢？就是在getResponse()方法中。
        
        httpEngine.readResponse();
      }

      return true;
    } catch (IOException e) {
      HttpEngine retryEngine = httpEngine.recover(e);
      if (retryEngine != null) {
        httpEngine = retryEngine;
        return false;
      }

      // Give up; recovery is not possible.
      httpEngineFailure = e;
      throw e;
    }
  }
```
接着看`HttpEngine.readResponse()`方法吧。注释说的非常明白。   
```java
 /**
   * Flushes the remaining request header and body, parses the HTTP response
   * headers and starts reading the HTTP response body if it exists.
   */
  public final void readResponse() throws IOException {
    if (response != null) return;
    if (responseSource == null) throw new IllegalStateException("call sendRequest() first!");
    if (!responseSource.requiresConnection()) return;

    // Flush the request body if there's data outstanding.
    if (bufferedRequestBody != null && bufferedRequestBody.buffer().size() > 0) {
      bufferedRequestBody.flush();
    }

    if (sentRequestMillis == -1) {
      if (OkHeaders.contentLength(request) == -1 && requestBodyOut instanceof RetryableSink) {
        // We might not learn the Content-Length until the request body has been buffered.
        long contentLength = ((RetryableSink) requestBodyOut).contentLength();
        request = request.newBuilder()
            .header("Content-Length", Long.toString(contentLength))
            .build();
      }
      transport.writeRequestHeaders(request);
    }

    if (requestBodyOut != null) {
      if (bufferedRequestBody != null) {
        // This also closes the wrapped requestBodyOut.
        bufferedRequestBody.close();
      } else {
        requestBodyOut.close();
      }
      if (requestBodyOut instanceof RetryableSink) {
        transport.writeRequestBody((RetryableSink) requestBodyOut);
      }
    }

    transport.flushRequest();

    response = transport.readResponseHeaders()
        .request(request)
        .handshake(connection.getHandshake())
        .header(OkHeaders.SENT_MILLIS, Long.toString(sentRequestMillis))
        .header(OkHeaders.RECEIVED_MILLIS, Long.toString(System.currentTimeMillis()))
        .setResponseSource(responseSource)
        .build();
    connection.setHttpMinorVersion(response.httpMinorVersion());
    receiveHeaders(response.headers());

    if (responseSource == ResponseSource.CONDITIONAL_CACHE) {
      if (validatingResponse.validate(response)) {
        transport.emptyTransferStream();
        releaseConnection();
        response = combine(validatingResponse, response);

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        OkResponseCache responseCache = client.getOkResponseCache();
        responseCache.trackConditionalCacheHit();
        responseCache.update(validatingResponse, cacheableResponse());

        if (validatingResponse.body() != null) {
          initContentStream(validatingResponse.body().source());
        }
        return;
      } else {
        closeQuietly(validatingResponse.body());
      }
    }

    if (!hasResponseBody()) {
      // Don't call initContentStream() when the response doesn't have any content.
      responseTransferSource = transport.getTransferStream(cacheRequest);
      responseBody = responseTransferSource;
      return;
    }

    // 设置cacheRequest的值
    maybeCache();
    // 设置返回数据了，transport这里也就是HttpTransport。他的getTransferStream返回的是一个Source接口的实现类。也就是返回的数据。
    initContentStream(transport.getTransferStream(cacheRequest));
  }
```
先看一下maybeCache()方法：    
```java
  private void maybeCache() throws IOException {
    OkResponseCache responseCache = client.getOkResponseCache();
    if (responseCache == null) return;

    // Should we cache this response for this request?
    if (!CacheStrategy.isCacheable(response, request)) {
      responseCache.maybeRemove(request);
      return;
    }

    // Offer this request to the cache.
    cacheRequest = responseCache.put(cacheableResponse());
  }
```
再看一下`initContentStream()`方法：    
```java
  /**
   * Initialize the response content stream from the response transfer source.
   * These two sources are the same unless we're doing transparent gzip, in
   * which case the content source is decompressed.
   *
   * <p>Whenever we do transparent gzip we also strip the corresponding headers.
   * We strip the Content-Encoding header to prevent the application from
   * attempting to double decompress. We strip the Content-Length header because
   * it is the length of the compressed content, but the application is only
   * interested in the length of the uncompressed content.
   *
   * <p>This method should only be used for non-empty response bodies. Response
   * codes like "304 Not Modified" can include "Content-Encoding: gzip" without
   * a response body and we will crash if we attempt to decompress the zero-byte
   * source.
   */
  private void initContentStream(Source transferSource) throws IOException {
    responseTransferSource = transferSource;
    if (transparentGzip && "gzip".equalsIgnoreCase(response.header("Content-Encoding"))) { 
      // 没有结果时response为null，有结果了就会给他赋值。
      response = response.newBuilder()
          .removeHeader("Content-Encoding")
          .removeHeader("Content-Length")
          .build();
      responseBody = new GzipSource(transferSource);
    } else {
      responseBody = transferSource;
    }
  }
```
到这里又执行完了，responseBody已经被赋值了，他是一个Source接口的实现类。也就是说到这里，这次网络请求就完成了，也收到了服务器返回的数据。


也就是说到这里我们已经分析了:  
```
HttpURLConnection connection = (HttpURLConnection)new URL(url).openConnection();
connection.connect();
```
接下来就是分析`connection.getResponseCode()`以及`connection.getOutputStream()`这两个方法了。       
先看一下`getResponseCode()`方法：　　　　
```java
  @Override public final int getResponseCode() throws IOException {
    // 看到了吗？这里就是刚才我们说的execute()方法的参数什么时候会为true的地方。
    return getResponse().getResponse().code();
  }

```
那我们接着看一下`getResponse()`方法，其实就是直接读取响应头的响应值：　　　　　　
```java
  /**
   * Aggressively tries to get the final HTTP response, potentially making
   * many HTTP requests in the process in order to cope with redirects and
   * authentication.
   */
  private HttpEngine getResponse() throws IOException {
    initHttpEngine();

    // 如果已经有返回数据了就直接返回
    if (httpEngine.hasResponse()) {
      return httpEngine;
    }

    while (true) {
      // 参数为true了。
      if (!execute(true)) {
        continue;
      }

      Retry retry = processResponseHeaders();
      if (retry == Retry.NONE) {
        httpEngine.releaseConnection();
        return httpEngine;
      }

      // The first request was insufficient. Prepare for another...
      String retryMethod = method;
      Sink requestBody = httpEngine.getRequestBody();

      // Although RFC 2616 10.3.2 specifies that a HTTP_MOVED_PERM
      // redirect should keep the same method, Chrome, Firefox and the
      // RI all issue GETs when following any redirect.
      int responseCode = httpEngine.getResponse().code();
      if (responseCode == HTTP_MULT_CHOICE
          || responseCode == HTTP_MOVED_PERM
          || responseCode == HTTP_MOVED_TEMP
          || responseCode == HTTP_SEE_OTHER) {
        retryMethod = "GET";
        requestHeaders.removeAll("Content-Length");
        requestBody = null;
      }

      if (requestBody != null && !(requestBody instanceof RetryableSink)) {
        throw new HttpRetryException("Cannot retry streamed HTTP body", responseCode);
      }

      if (retry == Retry.DIFFERENT_CONNECTION) {
        httpEngine.releaseConnection();
      }

      Connection connection = httpEngine.close();
      httpEngine = newHttpEngine(retryMethod, connection, (RetryableSink) requestBody);
    }
  }
```
接着看一下`getOutputStream()`的源码：     
```java
  @Override public final OutputStream getOutputStream() throws IOException {
    // 看到了吗？他内部会先去调用connect()方法
    connect();

　  // 这里可能有人会有说，getOutputStream和request body有什么关系，应该是response body才对啊。
    // 不要弄混了啊，getOutputStream是要把post请求的数据输入给请求。
    BufferedSink sink = httpEngine.getBufferedRequestBody();
    if (sink == null) {
      throw new ProtocolException("method does not support a request body: " + method);
    } else if (httpEngine.hasResponse()) {
      throw new ProtocolException("cannot write request body after response has been read");
    }

    return sink.outputStream();
  }
```
顺便再看一下`getInputStream()`方法：     　
```java
  @Override public final InputStream getInputStream() throws IOException {
    if (!doInput) {
      throw new ProtocolException("This protocol does not support input");
    }
    // 它也会直接调用getResponse()方法，这个比较好理解。
    HttpEngine response = getResponse();

    // if the requested file does not exist, throw an exception formerly the
    // Error page from the server was returned if the requested file was
    // text/html this has changed to return FileNotFoundException for all
    // file types
    if (getResponseCode() >= HTTP_BAD_REQUEST) {
      throw new FileNotFoundException(url.toString());
    }

    InputStream result = response.getResponseBodyBytes();
    if (result == null) {
      throw new ProtocolException("No response body exists; responseCode=" + getResponseCode());
    }
    return result;
  }

```
再顺便看一下`HttpURLConnection.disconnect()`方法，因为这个方法可能很多人不清楚该不该调用他,不过注释说的很明白了：     
```java
    /**
     * Releases this connection so that its resources may be either reused or
     * closed.
     *
     * <p>Unlike other Java implementations, this will not necessarily close
     * socket connections that can be reused. You can disable all connection
     * reuse by setting the {@code http.keepAlive} system property to {@code
     * false} before issuing any HTTP requests.
     */
  @Override public final void disconnect() {
    // Calling disconnect() before a connection exists should have no effect.
    if (httpEngine != null) {
      // 调用HttpEngine.close()方法
      httpEngine.close();
    }
  }
```
看一下HttpEngine.close()方法:     
```java
  /**
   * Release any resources held by this engine. If a connection is still held by
   * this engine, it is returned.
   */
  public final Connection close() {
    if (bufferedRequestBody != null) {
      // This also closes the wrapped requestBodyOut.
      closeQuietly(bufferedRequestBody);
    } else if (requestBodyOut != null) {
      closeQuietly(requestBodyOut);
    }

    // If this engine never achieved a response body, its connection cannot be reused.
    if (responseBody == null) {
      closeQuietly(connection);
      connection = null;
      return null;
    }

    // Close the response body. This will recycle the connection if it is eligible.
    closeQuietly(responseBody);

    // Clear the buffer held by the response body input stream adapter.
    closeQuietly(responseBodyBytes);

    // Close the connection if it cannot be reused.
    if (transport != null && !transport.canReuseConnection()) {
      closeQuietly(connection);
      connection = null;
      return null;
    }

    Connection result = connection;
    connection = null;
    return result;
  }
```
继续看一下closeQuietly(connection)方法：     
```java
  /**
   * Closes {@code closeable}, ignoring any checked exceptions. Does nothing
   * if {@code closeable} is null.
   */
  public static void closeQuietly(Closeable closeable) {
    if (closeable != null) {
      try {
        // 这里也就是Connection的close()方法
        closeable.close();
      } catch (RuntimeException rethrown) {
        throw rethrown;
      } catch (Exception ignored) {
      }
    }
  }
```

再接着看一下Connection.close()方法：    
```java
  @Override public void close() throws IOException {
    // 直接调用了socket.close()方法。这些socket也关了。
    socket.close();
  }
```
再看Socket.close()方法：    
```java
    /**
     * Closes the socket. It is not possible to reconnect or rebind to this
     * socket thereafter which means a new socket instance has to be created.
     *
     * @throws IOException
     *             if an error occurs while closing the socket.
     */
    public synchronized void close() throws IOException {
        isClosed = true;
        isConnected = false;
        // RI compatibility: the RI returns the any address (but the original local port) after
        // close.
        localAddress = Inet4Address.ANY;
        impl.close();
    }
```





