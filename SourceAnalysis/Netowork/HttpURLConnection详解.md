Android HttpURLConnection源码分析
---

之前写过HttpURLConnection与HttpClient的区别及选择。后来又分析了Volley的源码。
最近又遇到了问题，想在Volley中针对HttpURLConnection添加连接池的功能，开始有点懵了，不知道HttpURLConnection要怎么加连接池，
虽然感觉这是没必要的，但是心底确拿不出依据。所以研究下HttpURLConnection的源码进行分析。


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
```java    
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
package com.squareup.okhttp;

import com.squareup.okhttp.internal.Util;
import com.squareup.okhttp.internal.http.HttpAuthenticator;
import com.squareup.okhttp.internal.http.HttpURLConnectionImpl;
import com.squareup.okhttp.internal.http.HttpsURLConnectionImpl;
import com.squareup.okhttp.internal.http.ResponseCacheAdapter;
import com.squareup.okhttp.internal.okio.ByteString;
import com.squareup.okhttp.internal.tls.OkHostnameVerifier;
import java.io.IOException;
import java.net.CookieHandler;
import java.net.HttpURLConnection;
import java.net.Proxy;
import java.net.ProxySelector;
import java.net.ResponseCache;
import java.net.URL;
import java.net.URLConnection;
import java.net.URLStreamHandler;
import java.net.URLStreamHandlerFactory;
import java.security.GeneralSecurityException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;
import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;

/** Configures and creates HTTP connections. */
public final class OkHttpClient implements URLStreamHandlerFactory, Cloneable {

  private final RouteDatabase routeDatabase;
  private Proxy proxy;
  private List<Protocol> protocols;
  private ProxySelector proxySelector;
  private CookieHandler cookieHandler;
  private OkResponseCache responseCache;
  private SSLSocketFactory sslSocketFactory;
  private HostnameVerifier hostnameVerifier;
  private OkAuthenticator authenticator;
  private ConnectionPool connectionPool;
  private boolean followProtocolRedirects = true;
  private int connectTimeout;
  private int readTimeout;

  public OkHttpClient() {
    routeDatabase = new RouteDatabase();
  }

  /**
   * Sets the default connect timeout for new connections. A value of 0 means no timeout.
   *
   * @see URLConnection#setConnectTimeout(int)
   */
  public void setConnectTimeout(long timeout, TimeUnit unit) {
    if (timeout < 0) {
      throw new IllegalArgumentException("timeout < 0");
    }
    if (unit == null) {
      throw new IllegalArgumentException("unit == null");
    }
    long millis = unit.toMillis(timeout);
    if (millis > Integer.MAX_VALUE) {
      throw new IllegalArgumentException("Timeout too large.");
    }
    connectTimeout = (int) millis;
  }

  /** Default connect timeout (in milliseconds). */
  public int getConnectTimeout() {
    return connectTimeout;
  }

  /**
   * Sets the default read timeout for new connections. A value of 0 means no timeout.
   *
   * @see URLConnection#setReadTimeout(int)
   */
  public void setReadTimeout(long timeout, TimeUnit unit) {
    if (timeout < 0) {
      throw new IllegalArgumentException("timeout < 0");
    }
    if (unit == null) {
      throw new IllegalArgumentException("unit == null");
    }
    long millis = unit.toMillis(timeout);
    if (millis > Integer.MAX_VALUE) {
      throw new IllegalArgumentException("Timeout too large.");
    }
    readTimeout = (int) millis;
  }

  /** Default read timeout (in milliseconds). */
  public int getReadTimeout() {
    return readTimeout;
  }

  /**
   * Sets the HTTP proxy that will be used by connections created by this
   * client. This takes precedence over {@link #setProxySelector}, which is
   * only honored when this proxy is null (which it is by default). To disable
   * proxy use completely, call {@code setProxy(Proxy.NO_PROXY)}.
   */
  public OkHttpClient setProxy(Proxy proxy) {
    this.proxy = proxy;
    return this;
  }

  public Proxy getProxy() {
    return proxy;
  }

  /**
   * Sets the proxy selection policy to be used if no {@link #setProxy proxy}
   * is specified explicitly. The proxy selector may return multiple proxies;
   * in that case they will be tried in sequence until a successful connection
   * is established.
   *
   * <p>If unset, the {@link ProxySelector#getDefault() system-wide default}
   * proxy selector will be used.
   */
  public OkHttpClient setProxySelector(ProxySelector proxySelector) {
    this.proxySelector = proxySelector;
    return this;
  }

  public ProxySelector getProxySelector() {
    return proxySelector;
  }

  /**
   * Sets the cookie handler to be used to read outgoing cookies and write
   * incoming cookies.
   *
   * <p>If unset, the {@link CookieHandler#getDefault() system-wide default}
   * cookie handler will be used.
   */
  public OkHttpClient setCookieHandler(CookieHandler cookieHandler) {
    this.cookieHandler = cookieHandler;
    return this;
  }

  public CookieHandler getCookieHandler() {
    return cookieHandler;
  }

  /**
   * Sets the response cache to be used to read and write cached responses.
   */
  public OkHttpClient setResponseCache(ResponseCache responseCache) {
    return setOkResponseCache(toOkResponseCache(responseCache));
  }

  public ResponseCache getResponseCache() {
    return responseCache instanceof ResponseCacheAdapter
        ? ((ResponseCacheAdapter) responseCache).getDelegate()
        : null;
  }

  public OkHttpClient setOkResponseCache(OkResponseCache responseCache) {
    this.responseCache = responseCache;
    return this;
  }

  public OkResponseCache getOkResponseCache() {
    return responseCache;
  }

  /**
   * Sets the socket factory used to secure HTTPS connections.
   *
   * <p>If unset, a lazily created SSL socket factory will be used.
   */
  public OkHttpClient setSslSocketFactory(SSLSocketFactory sslSocketFactory) {
    this.sslSocketFactory = sslSocketFactory;
    return this;
  }

  public SSLSocketFactory getSslSocketFactory() {
    return sslSocketFactory;
  }

  /**
   * Sets the verifier used to confirm that response certificates apply to
   * requested hostnames for HTTPS connections.
   *
   * <p>If unset, the
   * {@link javax.net.ssl.HttpsURLConnection#getDefaultHostnameVerifier()
   * system-wide default} hostname verifier will be used.
   */
  public OkHttpClient setHostnameVerifier(HostnameVerifier hostnameVerifier) {
    this.hostnameVerifier = hostnameVerifier;
    return this;
  }

  public HostnameVerifier getHostnameVerifier() {
    return hostnameVerifier;
  }

  /**
   * Sets the authenticator used to respond to challenges from the remote web
   * server or proxy server.
   *
   * <p>If unset, the {@link java.net.Authenticator#setDefault system-wide default}
   * authenticator will be used.
   */
  public OkHttpClient setAuthenticator(OkAuthenticator authenticator) {
    this.authenticator = authenticator;
    return this;
  }

  public OkAuthenticator getAuthenticator() {
    return authenticator;
  }

  /**
   * Sets the connection pool used to recycle HTTP and HTTPS connections.
   *
   * <p>If unset, the {@link ConnectionPool#getDefault() system-wide
   * default} connection pool will be used.
   */
  public OkHttpClient setConnectionPool(ConnectionPool connectionPool) {
    this.connectionPool = connectionPool;
    return this;
  }

  public ConnectionPool getConnectionPool() {
    return connectionPool;
  }

  /**
   * Configure this client to follow redirects from HTTPS to HTTP and from HTTP
   * to HTTPS.
   *
   * <p>If unset, protocol redirects will be followed. This is different than
   * the built-in {@code HttpURLConnection}'s default.
   */
  public OkHttpClient setFollowProtocolRedirects(boolean followProtocolRedirects) {
    this.followProtocolRedirects = followProtocolRedirects;
    return this;
  }

  public boolean getFollowProtocolRedirects() {
    return followProtocolRedirects;
  }

  public RouteDatabase getRoutesDatabase() {
    return routeDatabase;
  }

  /**
   * @deprecated OkHttp 1.5 enforces an enumeration of {@link Protocol protocols}
   * that can be selected. Please switch to {@link #setProtocols(java.util.List)}.
   */
  @Deprecated
  public OkHttpClient setTransports(List<String> transports) {
    List<Protocol> protocols = new ArrayList<Protocol>(transports.size());
    for (int i = 0, size = transports.size(); i < size; i++) {
      try {
        Protocol protocol = Util.getProtocol(ByteString.encodeUtf8(transports.get(i)));
        protocols.add(protocol);
      } catch (IOException e) {
        throw new IllegalArgumentException(e);
      }
    }
    return setProtocols(protocols);
  }

  /**
   * Configure the protocols used by this client to communicate with remote
   * servers. By default this client will prefer the most efficient transport
   * available, falling back to more ubiquitous protocols. Applications should
   * only call this method to avoid specific compatibility problems, such as web
   * servers that behave incorrectly when SPDY is enabled.
   *
   * <p>The following protocols are currently supported:
   * <ul>
   *   <li><a href="http://www.w3.org/Protocols/rfc2616/rfc2616.html">http/1.1</a>
   *   <li><a href="http://www.chromium.org/spdy/spdy-protocol/spdy-protocol-draft3-1">spdy/3.1</a>
   *   <li><a href="http://tools.ietf.org/html/draft-ietf-httpbis-http2-09">HTTP-draft-09/2.0</a>
   * </ul>
   *
   * <p><strong>This is an evolving set.</strong> Future releases may drop
   * support for transitional protocols (like spdy/3.1), in favor of their
   * successors (spdy/4 or http/2.0). The http/1.1 transport will never be
   * dropped.
   *
   * <p>If multiple protocols are specified, <a
   * href="https://technotes.googlecode.com/git/nextprotoneg.html">NPN</a> will
   * be used to negotiate a transport. Future releases may use another mechanism
   * (such as <a href="http://tools.ietf.org/html/draft-friedl-tls-applayerprotoneg-02">ALPN</a>)
   * to negotiate a transport.
   *
   * @param protocols the protocols to use, in order of preference. The list
   *     must contain "http/1.1". It must not contain null.
   */
  public OkHttpClient setProtocols(List<Protocol> protocols) {
    protocols = Util.immutableList(protocols);
    if (!protocols.contains(Protocol.HTTP_11)) {
      throw new IllegalArgumentException("protocols doesn't contain http/1.1: " + protocols);
    }
    if (protocols.contains(null)) {
      throw new IllegalArgumentException("protocols must not contain null");
    }
    this.protocols = Util.immutableList(protocols);
    return this;
  }

  /**
   * @deprecated OkHttp 1.5 enforces an enumeration of {@link Protocol
   *     protocols} that can be selected. Please switch to {@link
   *     #getProtocols()}.
   */
  @Deprecated
  public List<String> getTransports() {
    List<String> transports = new ArrayList<String>(protocols.size());
    for (int i = 0, size = protocols.size(); i < size; i++) {
      transports.add(protocols.get(i).name.utf8());
    }
    return transports;
  }

  public List<Protocol> getProtocols() {
    return protocols;
  }

  public HttpURLConnection open(URL url) {
    return open(url, proxy);
  }

  HttpURLConnection open(URL url, Proxy proxy) {
    String protocol = url.getProtocol();
	// 将该对象clone后设置一些其他的属性返回，里面会设置一个默认的连接池。
    OkHttpClient copy = copyWithDefaults();
    copy.proxy = proxy;
	// 返回了HttpURLConnectionImpl，并且把clone后的OKHttpClient对象传递进去。
    if (protocol.equals("http")) return new HttpURLConnectionImpl(url, copy);
    if (protocol.equals("https")) return new HttpsURLConnectionImpl(url, copy);
    throw new IllegalArgumentException("Unexpected protocol: " + protocol);
  }

  /**
   * Returns a shallow copy of this OkHttpClient that uses the system-wide
   * default for each field that hasn't been explicitly configured.
   */
  OkHttpClient copyWithDefaults() {
    OkHttpClient result = clone();
    if (result.proxySelector == null) {
      result.proxySelector = ProxySelector.getDefault();
    }
    if (result.cookieHandler == null) {
      result.cookieHandler = CookieHandler.getDefault();
    }
    if (result.responseCache == null) {
      result.responseCache = toOkResponseCache(ResponseCache.getDefault());
    }
    if (result.sslSocketFactory == null) {
      result.sslSocketFactory = getDefaultSSLSocketFactory();
    }
    if (result.hostnameVerifier == null) {
      result.hostnameVerifier = OkHostnameVerifier.INSTANCE;
    }
    if (result.authenticator == null) {
      result.authenticator = HttpAuthenticator.SYSTEM_DEFAULT;
    }
    if (result.connectionPool == null) {
	  // 会给OkHttpClient设置一个默认的连接池
      result.connectionPool = ConnectionPool.getDefault();
    }
    if (result.protocols == null) {
      result.protocols = Util.HTTP2_SPDY3_AND_HTTP;
    }
    return result;
  }

  /**
   * Java and Android programs default to using a single global SSL context,
   * accessible to HTTP clients as {@link SSLSocketFactory#getDefault()}. If we
   * used the shared SSL context, when OkHttp enables NPN for its SPDY-related
   * stuff, it would also enable NPN for other usages, which might crash them
   * because NPN is enabled when it isn't expected to be.
   * <p>
   * This code avoids that by defaulting to an OkHttp created SSL context. The
   * significant drawback of this approach is that apps that customize the
   * global SSL context will lose these customizations.
   */
  private synchronized SSLSocketFactory getDefaultSSLSocketFactory() {
    if (sslSocketFactory == null) {
      try {
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null, null, null);
        sslSocketFactory = sslContext.getSocketFactory();
      } catch (GeneralSecurityException e) {
        throw new AssertionError(); // The system has no TLS. Just give up.
      }
    }
    return sslSocketFactory;
  }

  /** Returns a shallow copy of this OkHttpClient. */
  @Override public OkHttpClient clone() {
    try {
      return (OkHttpClient) super.clone();
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }

  private OkResponseCache toOkResponseCache(ResponseCache responseCache) {
    return responseCache == null || responseCache instanceof OkResponseCache
        ? (OkResponseCache) responseCache
        : new ResponseCacheAdapter(responseCache);
  }

  /**
   * Creates a URLStreamHandler as a {@link URL#setURLStreamHandlerFactory}.
   *
   * <p>This code configures OkHttp to handle all HTTP and HTTPS connections
   * created with {@link URL#openConnection()}: <pre>   {@code
   *
   *   OkHttpClient okHttpClient = new OkHttpClient();
   *   URL.setURLStreamHandlerFactory(okHttpClient);
   * }</pre>
   */
  public URLStreamHandler createURLStreamHandler(final String protocol) {
    if (!protocol.equals("http") && !protocol.equals("https")) return null;

    return new URLStreamHandler() {
      @Override protected URLConnection openConnection(URL url) {
        return open(url);
      }

      @Override protected URLConnection openConnection(URL url, Proxy proxy) {
        return open(url, proxy);
      }

      @Override protected int getDefaultPort() {
        if (protocol.equals("http")) return 80;
        if (protocol.equals("https")) return 443;
        throw new AssertionError();
      }
    };
  }
}
```
接着看一下`HttpURLConnectionImpl`类,它是`HttpURLConnection`的子类：     
```java
public class HttpURLConnectionImpl extends HttpURLConnection {
    .....
}
```
到这里`new URL(url).openConnection()`方法已经分析完了，其实就是返回了一个HtppURLConnectionImpl对象。    

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
	// 将之前通过构造函数传递进来的OkHttpClient对象clone一份后再传递给HttpEngine
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
	  // 记录下当前的请求是来自网络请求还是来时缓存中的数据。
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
		// 调用connect方法，内部会重新创建一个connection，连接到服务器、重定向或者通过代理。
		connect();
	  }
	  // 通过Connection创建一个HttpTransport类。这个和后面的connection类一起看， Transport接口提供了一个用户写Request头和数据的输出流。
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
		// 回收connection，这里就看到了连接池，这个client就是构造函数中传递进来的OKHttpClient
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
	  // RoteSeclector类介绍. Selects routes to connect to an origin server. Each connection requires a
	  // choice of proxy server, IP address, and TLS mode. Connections may also be
	  // recycled.注意他把OkHttpClient中的connection pool传递进来了。
	  routeSelector = new RouteSelector(address, request.uri(), client.getProxySelector(),
		  client.getConnectionPool(), Dns.DEFAULT, client.getRoutesDatabase());
	}

	// roteSeclecrot.next()方法的注释Returns the next route address to attempt.这一步非常重要。
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
接下来我们要先看一下routeSelector.next()方法如何返回connection对象，然后在看Connection.connect()方法：    
```java
/**
* Returns the next route address to attempt.
*
* @throws NoSuchElementException if there are no more routes to attempt.
*/
public Connection next(String method) throws IOException {
    // 使用连接池获取Connection的地方。pool就是OkHttpClient中的连接池。
	// Always prefer pooled connections over new connections.
	for (Connection pooled; (pooled = pool.get(address)) != null; ) {
	  // 匹配get方法，或者判断是否可读，http1.x是通过判断socket是否关闭来判断是否可读的。
	  if (method.equals("GET") || pooled.isReadable()) return pooled;
	  // 不满足重用，就关闭。
	  pooled.close();
	}

	// Compute the next route to attempt.
	if (!hasNextTlsMode()) {
	  if (!hasNextInetSocketAddress()) {
		if (!hasNextProxy()) {
		  if (!hasNextPostponed()) {
			throw new NoSuchElementException();
		  }
		  return new Connection(pool, nextPostponed());
		}
		lastProxy = nextProxy();
		resetNextInetSocketAddress(lastProxy);
	  }
	  lastInetSocketAddress = nextInetSocketAddress();
	  resetNextTlsMode();
	}

	boolean modernTls = nextTlsMode() == TLS_MODE_MODERN;
	Route route = new Route(address, lastProxy, lastInetSocketAddress, modernTls);
	if (routeDatabase.shouldPostpone(route)) {
	  postponedRoutes.add(route);
	  // We will only recurse in order to skip previously failed routes. They will be
	  // tried last.
	  return next(method);
	}
    // 没有的话也会去创建，并把OkHttpClient中的连接池传递进去。
	return new Connection(pool, route);
}
```

再看一下`Connection`类的实现以及其`connect()`方法:   
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
  // 传递进来的连接池。
  public Connection(ConnectionPool pool, Route route) {
    this.pool = pool;
    this.route = route;
  }

  public void connect(int connectTimeout, int readTimeout, TunnelRequest tunnelRequest)
      throws IOException {
    if (connected) throw new IllegalStateException("already connected");

    socket = (route.proxy.type() != Proxy.Type.HTTP) ? new Socket(route.proxy) : new Socket();
    // 连socket了，内部调用了socket.connect()方法。Connects this socket to the given remote host address and port specified
    // by the SocketAddress {@code remoteAddr} with the specified timeout. The
    // connecting method will block until the connection is established or an
    // error occurred.
    Platform.get().connectSocket(socket, route.inetSocketAddress, connectTimeout);
    socket.setSoTimeout(readTimeout);
    in = socket.getInputStream();
    out = socket.getOutputStream();

    if (route.address.sslSocketFactory != null) {
	  // 完成TLS握手和验证
      upgradeToTls(tunnelRequest);
    } else {
      initSourceAndSink();
      // 创建HttpConnection.A socket connection that can be used to send HTTP/1.1 messages. 
      // 这个HttpConnection有什么用呢？就是下面的newTransport方法中会用。而且还要把连接池传递进去？
      httpConnection = new HttpConnection(pool, this, source, sink);
    }
	// 这样就已经连接上了
    connected = true;
  }

  // 该方法决定了使用的协议是SPDY还是HTTP
  /** Returns the transport appropriate for this connection. */
  public Object newTransport(HttpEngine httpEngine) throws IOException {
    return (spdyConnection != null)
        ? new SpdyTransport(httpEngine, spdyConnection)
        : new HttpTransport(httpEngine, httpConnection);
  }
}
```

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
  // 检查缓存是否可用，如果可用就用当前缓存的response。并且释放该连接。
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

```java
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

// HttpTransport.canReuseConnection()用于判断该Connection是否可复用
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
package com.squareup.okhttp;

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


到这里就都看完了，最后我们再看一下上面用到的连接池，也就是`ConnectionPool`类:  因为在上面的分析中，我们发现此类贯穿了很多类。
它为`OkHttpClient`中的对象，后贯穿到`HttpEngine`、`Connection`、`HttpConnection`等。所以要分析下。
```java
/**
 * Manages reuse of HTTP and SPDY connections for reduced network latency. HTTP
 * requests that share the same {@link com.squareup.okhttp.Address} may share a
 * {@link com.squareup.okhttp.Connection}. This class implements the policy of
 * which connections to keep open for future use.
 *
 * <p>The {@link #getDefault() system-wide default} uses system properties for
 * tuning parameters:
 * <ul>
 *     <li>{@code http.keepAlive} true if HTTP and SPDY connections should be
 *         pooled at all. Default is true.
 *     <li>{@code http.maxConnections} maximum number of idle connections to
 *         each to keep in the pool. Default is 5.
 *     <li>{@code http.keepAliveDuration} Time in milliseconds to keep the
 *         connection alive in the pool before closing it. Default is 5 minutes.
 *         This property isn't used by {@code HttpURLConnection}.
 * </ul>
 *
 * <p>The default instance <i>doesn't</i> adjust its configuration as system
 * properties are changed. This assumes that the applications that set these
 * parameters do so before making HTTP connections, and that this class is
 * initialized lazily.
 */
public class ConnectionPool {
  private static final int MAX_CONNECTIONS_TO_CLEANUP = 2;
  private static final long DEFAULT_KEEP_ALIVE_DURATION_MS = 5 * 60 * 1000; // 5 min

  // 这个就是getDefault方法所返回的默认连接池。
  private static final ConnectionPool systemDefault;

  static {
    String keepAlive = System.getProperty("http.keepAlive");
	// 存活时间
    String keepAliveDuration = System.getProperty("http.keepAliveDuration");
	// 最大空闲连接数
    String maxIdleConnections = System.getProperty("http.maxConnections");
    long keepAliveDurationMs = keepAliveDuration != null ? Long.parseLong(keepAliveDuration)
        : DEFAULT_KEEP_ALIVE_DURATION_MS;
    if (keepAlive != null && !Boolean.parseBoolean(keepAlive)) {
      systemDefault = new ConnectionPool(0, keepAliveDurationMs);
    } else if (maxIdleConnections != null) {
      systemDefault = new ConnectionPool(Integer.parseInt(maxIdleConnections), keepAliveDurationMs);
    } else {
      systemDefault = new ConnectionPool(5, keepAliveDurationMs);
    }
  }

  /** The maximum number of idle connections for each address. */
  private final int maxIdleConnections;
  private final long keepAliveDurationNs;

  private final LinkedList<Connection> connections = new LinkedList<Connection>();

  // 连接的有效性检测是所有连接池都面临的一个通用问题，大部分HTTP服务器为了控制资源开销，并不会永久的维护一个长连接，而是一段时间就会关闭该连接。
  // 放回连接池的连接，如果在服务器端已经关闭，客户端是无法检测到这个状态变化而及时的关闭Socket的。这就造成了线程从连接池中获取的连接不一定是有效的。
  // 这个问题的一个解决方法就是在每次请求之前检查该连接是否已经存在了过长时间，可能已过期。但是这个方法会使得每次请求都增加额外的开销。
  // 所以通过下面的任务来执行清理过期的连接。
  /** We use a single background thread to cleanup expired connections. */
  private final ExecutorService executorService = new ThreadPoolExecutor(0, 1,
      60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(),
      Util.threadFactory("OkHttp ConnectionPool", true));
  private final Runnable connectionsCleanupRunnable = new Runnable() {
    @Override public void run() {
      List<Connection> expiredConnections = new ArrayList<Connection>(MAX_CONNECTIONS_TO_CLEANUP);
      int idleConnectionCount = 0;
      synchronized (ConnectionPool.this) {
        for (ListIterator<Connection> i = connections.listIterator(connections.size());
            i.hasPrevious(); ) {
          Connection connection = i.previous();
          if (!connection.isAlive() || connection.isExpired(keepAliveDurationNs)) {
            i.remove();
            expiredConnections.add(connection);
            if (expiredConnections.size() == MAX_CONNECTIONS_TO_CLEANUP) break;
          } else if (connection.isIdle()) {
            idleConnectionCount++;
          }
        }

        for (ListIterator<Connection> i = connections.listIterator(connections.size());
            i.hasPrevious() && idleConnectionCount > maxIdleConnections; ) {
          Connection connection = i.previous();
          if (connection.isIdle()) {
            expiredConnections.add(connection);
            i.remove();
            --idleConnectionCount;
          }
        }
      }
      for (Connection expiredConnection : expiredConnections) {
        Util.closeQuietly(expiredConnection);
      }
    }
  };

  public ConnectionPool(int maxIdleConnections, long keepAliveDurationMs) {
    this.maxIdleConnections = maxIdleConnections;
    this.keepAliveDurationNs = keepAliveDurationMs * 1000 * 1000;
  }

  /**
   * Returns a snapshot of the connections in this pool, ordered from newest to
   * oldest. Waits for the cleanup callable to run if it is currently scheduled.
   */
  List<Connection> getConnections() {
    waitForCleanupCallableToRun();
    synchronized (this) {
      return new ArrayList<Connection>(connections);
    }
  }

  /**
   * Blocks until the executor service has processed all currently enqueued
   * jobs.
   */
  private void waitForCleanupCallableToRun() {
    try {
      executorService.submit(new Runnable() {
        @Override public void run() {
        }
      }).get();
    } catch (Exception e) {
      throw new AssertionError();
    }
  }

  public static ConnectionPool getDefault() {
    return systemDefault;
  }

  /** Returns total number of connections in the pool. */
  public synchronized int getConnectionCount() {
    return connections.size();
  }

  /** Returns total number of spdy connections in the pool. */
  public synchronized int getSpdyConnectionCount() {
    int total = 0;
    for (Connection connection : connections) {
      if (connection.isSpdy()) total++;
    }
    return total;
  }

  /** Returns total number of http connections in the pool. */
  public synchronized int getHttpConnectionCount() {
    int total = 0;
    for (Connection connection : connections) {
      if (!connection.isSpdy()) total++;
    }
    return total;
  }

  /** Returns a recycled connection to {@code address}, or null if no such connection exists. */
  public synchronized Connection get(Address address) {
    Connection foundConnection = null;
    for (ListIterator<Connection> i = connections.listIterator(connections.size());
        i.hasPrevious(); ) {
      Connection connection = i.previous();
      if (!connection.getRoute().getAddress().equals(address)
          || !connection.isAlive()
          || System.nanoTime() - connection.getIdleStartTimeNs() >= keepAliveDurationNs) {
        continue;
      }
      i.remove();
      if (!connection.isSpdy()) {
	    // 不是spdy连接
        try {
		  // Platforml类对当前Android平台做了适配。
          Platform.get().tagSocket(connection.getSocket());
        } catch (SocketException e) {
          Util.closeQuietly(connection);
          // When unable to tag, skip recycling and close
          Platform.get().logW("Unable to tagSocket(): " + e);
          continue;
        }
      }
	  // 找到可复用的Connection
      foundConnection = connection;
      break;
    }

	// 针对spdy连接，添加到连接池中
    if (foundConnection != null && foundConnection.isSpdy()) {
      connections.addFirst(foundConnection); // Add it back after iteration.
    }

    executorService.execute(connectionsCleanupRunnable);
    return foundConnection;
  }

  /**
   * Gives {@code connection} to the pool. The pool may store the connection,
   * or close it, as its policy describes.
   *
   * <p>It is an error to use {@code connection} after calling this method.
   */
  public void recycle(Connection connection) {
    if (connection.isSpdy()) {
      return;
    }

    if (!connection.isAlive()) {
      Util.closeQuietly(connection);
      return;
    }

    try {
      Platform.get().untagSocket(connection.getSocket());
    } catch (SocketException e) {
      // When unable to remove tagging, skip recycling and close.
      Platform.get().logW("Unable to untagSocket(): " + e);
      Util.closeQuietly(connection);
      return;
    }

    synchronized (this) {
      connections.addFirst(connection);
      connection.incrementRecycleCount();
      connection.resetIdleStartTime();
    }

    executorService.execute(connectionsCleanupRunnable);
  }

  /**
   * Shares the SPDY connection with the pool. Callers to this method may
   * continue to use {@code connection}.
   */
  public void share(Connection connection) {
    if (!connection.isSpdy()) throw new IllegalArgumentException();
    executorService.execute(connectionsCleanupRunnable);
    if (connection.isAlive()) {
      synchronized (this) {
        connections.addFirst(connection);
      }
    }
  }

  /** Close and remove all connections in the pool. */
  public void evictAll() {
    List<Connection> connections;
    synchronized (this) {
      connections = new ArrayList<Connection>(this.connections);
      this.connections.clear();
    }

    for (int i = 0, size = connections.size(); i < size; i++) {
      Util.closeQuietly(connections.get(i));
    }
  }
}

```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

