Glide简介(下)
===

![image](https://github.com/bumptech/glide/blob/master/static/glide_logo.png)


官网:[Glide](https://github.com/bumptech/glide)[![Maven Central](https://img.shields.io/badge/version-4.1.1-brightgreen.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide) 


### `Glide`的回调:`Targets`   

假设我们不想将加载的图片显示到`ImageView`上，而是只想得到对应的`Bitmap`。`Glide`提供了一种可以通过`Targets`获取`Bitmap`的方法。
`Target`和`callback`没什么不同，都是在通过`Glide`的异步线程下载和处理后返回结果。  

`Glide`提供了多个不同目的的`targets`，我们先从`BaseTarget`开始。  

##### `BaseTarget`

```java
private BaseTarget target = new BaseTarget<BitmapDrawable>() {  
  @Override
  public void onResourceReady(BitmapDrawable bitmap, Transition<? super BitmapDrawable> transition) {
    // do something with the bitmap
    // for demonstration purposes, let's set it to an imageview
    imageView1.setImageDrawable(bitmap);
  }

  @Override
  public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(SIZE_ORIGINAL, SIZE_ORIGINAL);
  }

  @Override
  public void removeCallback(SizeReadyCallback cb) {}
};

private void loadImageSimpleTarget() {  
  GlideApp
    .with(context) // could be an issue!
    .load(eatFoodyImages[0])
    .into(target);
}
```
需要重写`getSize`方法，并且调用`cb.onSizeReady(SIZE_ORIGINAL, SIZE_ORIGINAL);`来保证`Glide`使用最高质量的值。当然，也可以传递一个具体的值，例如
根据`View`的大小返回。


##### `Target`注意事项    

除了刚介绍的`Target`的回调系统的一个简单版本，你要学会额外两件事。

第一个是`BaseTarget`对象的定义。`Java/Android`可以允许在`.into()`内匿名定义，但这会显著增加在`Glide`处理完图片请求前`Android`垃圾回收清理匿名`target`对象的可能性。最终，会导致图片被加载了，但是回调永远不会被调用。所以，请确保将你的回调定义为一个字段对象，防止被万恶的`Android`垃圾回收给清理掉。

第二个关键部分是`Glide`的`.with(context)`。这个问题实际上是`Glide`一个特性问题：当你传递了一个`context`，例如当前`app`的`activity`，当`activity`停止后，`Glide`会自动停止当前的请求。这种整合到`app`生命周期内是非常有用的，但也是很难处理的。如果你的`target`是独立于`app`的生命周期。这里的解决方案是使用`application的context`当`app`自己停止运行的时候，`Glide`会只取消掉图片的请求。请记住，再次提醒，如果你的请求需要在`activity`的生命周期以外，使用下面的代码：
```java
private void loadImageSimpleTargetApplicationContext() {  
    GlideApp
        .with(context.getApplicationContext() ) // safer!
        .load(eatFoodyImages[1] 
        .asBitmap()
        .into(target2);
}
```

##### 设置具体尺寸的`Target`    

通过`Target`的另一个问题就是它没有具体的尺寸。如果在`.into()`方法的参数中传递一个`ImageView`,`Glide`会根据`ImageView`的大小来控制图片的尺寸。例如在加载一个`1000x1000`的
图片时，但是要显示到的`ImageView`的大小是`250x250`，`Glide`会把该图片裁剪到小的尺寸来节省处理时间和内存。显示，在使用`Targets`时，这种方式并没有效果，因为根本不知道所需要的图片的大小。但是，如果你知道图片的最终所需要的尺寸时，你可以在`callback`中指定图片的尺寸来节省内存。

```java
private BaseTarget target2 = new BaseTarget<BitmapDrawable>() {  
  @Override
  public void onResourceReady(BitmapDrawable bitmap, Transition<? super BitmapDrawable> transition) {
    // do something with the bitmap
    // for demonstration purposes, let's set it to an imageview
    imageView2.setImageDrawable(bitmap);
  }

  @Override
  public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(250, 250);
  }

  @Override
  public void removeCallback(SizeReadyCallback cb) {}
};

private void loadImageSimpleTargetApplicationContext() {  
  GlideApp
    .with(context.getApplicationContext()) // safer!
    .load(eatFoodyImages[1])
    .into(target2);
}
```

正如上面通过`cb.onSizeReady(250, 250);`来指定尺寸。    


##### `ViewTarget`  

我们不直接使用`ImageView`的原因会有很多。我们可以通过上面的方式直接获取`Bitmap`。假如现在有一个自定义`View`，`Glide`并不支持加载图片到自定义`View`中，因为它并不知道要
将图片设置到该`View`中的哪个位置。然而，`Glide`提供了`ViewTargets`来让这种问题变的非常简单。
下面是我们的自定义`view`:    

```java
public class FutureStudioView extends FrameLayout {  
    ImageView iv;
    TextView tv;

    public void initialize(Context context) {
        inflate( context, R.layout.custom_view_futurestudio, this );

        iv = (ImageView) findViewById( R.id.custom_view_image );
        tv = (TextView) findViewById( R.id.custom_view_text );
    }

    public FutureStudioView(Context context, AttributeSet attrs) {
        super( context, attrs );
        initialize( context );
    }

    public FutureStudioView(Context context, AttributeSet attrs, int defStyleAttr) {
        super( context, attrs, defStyleAttr );
        initialize( context );
    }

    public void setImage(Drawable drawable) {
        iv = (ImageView) findViewById( R.id.custom_view_image );

        iv.setImageDrawable( drawable );
    }
}
```
因为该`View`并没有继承`ImageView`所以不能直接使用`.into()`方法。我们可以创建一个`ViewTarget`来替代`.into()`方法进行使用:   
```java
FutureStudioView customView = (FutureStudioView) findViewById( R.id.custom_view );

viewTarget = new ViewTarget<FutureStudioView, BitmapDrawable>(customView) {  
  @Override
  public void onResourceReady(BitmapDrawable bitmap, Transition<? super BitmapDrawable> transition) {
    this.view.setImage(bitmap);
  }
};

GlideApp  
    .with(context.getApplicationContext()) // safer!
    .load(eatFoodyImages[2])
    .into(viewTarget);
```


### 调试及`Debug`

可以通过`adb`命令来开启`Glide`的调试`log` 
`adb shell setprop log.tag.GenericRequest DEBUG`


##### 获取异常信息   

`Glide`不能直接通过`GenericRequest`类获取日志，但是我们可以获取异常信息。例如，当一个图片获取不到时，`glide`会抛出一个异常并且显示`.error()` 方法设置的错误图片。如果想明确哪里出了异常，可以通过`.listener()`方法传递一个`listener`对象进去。    

首先，创建一个`listener`对象作为一个成员变量，避免被垃圾回收：

```java
private RequestListener<Bitmap> requestListener = new RequestListener<Bitmap>() {  
  @Override
  public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Bitmap> target, boolean isFirstResource) {
    // todo log exception to central service or something like that

    // important to return false so the error placeholder can be placed
    return false;
  }

  @Override
  public boolean onResourceReady(Bitmap resource, Object model, Target<Bitmap> target, DataSource dataSource, boolean isFirstResource) {
    // everything worked out, so probably nothing to do
    return false;
  }
};

GlideApp  
    .with(context)
    .asBitmap()
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .listener(requestListener)
    .error(R.drawable.cupcake)
    .into(imageViewPlaceholder);
```


#### 配置第三方网络库    

- `OkHttp`  

```java
// image loading library Glide
compile 'com.github.bumptech.glide:glide:4.1.1'  
annotationProcessor 'com.github.bumptech.glide:compiler:4.1.1'

// Glide's OkHttp2 Integration 
compile 'com.github.bumptech.glide:okhttp-integration:4.1.1@aar'  
compile 'com.squareup.okhttp:okhttp:2.7.5'  
```

- `Volley`

```java
// image loading library Glide
compile 'com.github.bumptech.glide:glide:4.1.1'  
annotationProcessor 'com.github.bumptech.glide:compiler:4.1.1'

// Glide's Volley Integration 
compile 'com.github.bumptech.glide:volley-integration:4.1.1@aar'  
compile 'com.android.volley:volley:1.0.0'  
```

- `OkHttp3` 

```java
// image loading library Glide
compile 'com.github.bumptech.glide:glide:4.1.1'  
annotationProcessor 'com.github.bumptech.glide:compiler:4.1.1'

// Glide's OkHttp3 Integration 
compile 'com.github.bumptech.glide:okhttp3-integration:4.1.1@aar'  
compile 'com.squareup.okhttp3:okhttp:3.8.1'  
```

#### 用`Modules`定制`Glide`


`Glide modules`是一个全局改变`Glide`行为的抽象的方式。你需要创建`Glide`的实例，来访问`GlideBuilder`。可以通过创建一个公共的类，实现`AppGlideModule`的接口来定制`Glide`，这个接口提供了两个调整不同参数的方法，大多数情况下我们会用第一个方法`pplyOptions(Context context, GlideBuilder builder).`

```java
@GlideModule
public class FutureStudioAppGlideModule extends AppGlideModule {  
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {

    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {

    }
}
```
在上面的`applyOptions`方法的参数中有`GlideBuilder`对象，我们来看一下他的方法:   
- `.setMemoryCache(MemoryCache memoryCache)`
- `.setBitmapPool(BitmapPool bitmapPool)`
- `.setDiskCache(DiskCache.Factory diskCacheFactory)`
- `.setDiskCacheService(ExecutorService service)`
- `.setResizeService(ExecutorService service)`
- `.setDecodeFormat(DecodeFormat decodeFormat)`

由于`Glide`默认使用将`bitmap`使用`RGB565`解析，下面我们就通过`GlideModule`来将图片设置为`ARGB8888`
```java
@GlideModule
public class FutureStudioAppGlideModule extends AppGlideModule {  
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {

    }
}
```


### 接受自签名Https证书  

`Glide`内部使用标准的`HTTPUrlConnection`去下载图片。`Glide`也提供两个集成库。这三个方法优点是在安全设置上都是相当严格的。唯一的不足之处是当你从一个使用`HTTPS`，还是`self-signed`的服务器下载图片时，`Glide`并不会下载或者显示图片，因为`self-signed`认证会被认为存在安全问题。

这样，你会需要去实现能够接受`self-signed`认证的网络栈。幸运地，我们已经实现并用过一个“不安全的”`OkHttpClient`。由于它提供给了一个需要集成的常规`OkHttpClient`,我们只需要拷贝并粘贴这个类:    
```java
public class UnsafeOkHttpClient {  
    public static OkHttpClient getUnsafeOkHttpClient() {
        try {
            // Create a trust manager that does not validate certificate chains
            final TrustManager[] trustAllCerts = new TrustManager[] {
                    new X509TrustManager() {
                        @Override
                        public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                        }

                        @Override
                        public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) throws CertificateException {
                        }

                        @Override
                        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                            return new java.security.cert.X509Certificate[]{};
                        }
                    }
            };

            // Install the all-trusting trust manager
            final SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, trustAllCerts, new java.security.SecureRandom());

            // Create an ssl socket factory with our all-trusting manager
            final SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();

            OkHttpClient.Builder builder = new OkHttpClient.Builder();
            builder.sslSocketFactory(sslSocketFactory, (X509TrustManager)trustAllCerts[0]);
            builder.hostnameVerifier(new HostnameVerifier() {
                @Override
                public boolean verify(String hostname, SSLSession session) {
                    return true;
                }
            });

            OkHttpClient okHttpClient = builder.build();
            return okHttpClient;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```
该类关闭了所有`SSL`认证检查。   
接下来我们需要将上面的方法集成到`GlideModule`中。`Glide`使用一个`ModelLoader`去链接到数据模型创建一个具体的数据类型。我们的例子中，我们需要创建一个`ModelLoader`，它连接到一个`URL`，通过`GlideUrl`类响应并转化为输入流。`Glide`需要能够创建我们的新`ModelLoader`的实例，所以我们在`.register()`方法中传入一个工厂:  
```java
@GlideModule
public class UnsafeOkHttpGlideModule extends LibraryGlideModule {  
    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {
        OkHttpClient client = UnsafeOkHttpClient.getUnsafeOkHttpClient();
        registry.replace(GlideUrl.class, InputStream.class,
                new OkHttpUrlLoader.Factory(client));
    }
}
```


### 自定义缓存

`Glide`使用`MemorySizeCalculator`类来设置内存缓存的大小和`bitmap pool`。 `bitmap pool`会把图片保存到应用的栈内存中。合适的`bitmap pool`尺寸是非常必要的，因为要避免
太大会导致内存频繁的被回收。   
我们可以很简单的获取到`Glide`的`MemorySizeCalculator`类和默认的值:   
```java
MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context).build();  
int defaultMemoryCacheSize = calculator.getMemoryCacheSize();  
int defaultBitmapPoolSize = calculator.getBitmapPoolSize();  
```

那么如何自定义缓存大小呢？   
```java
@GlideModule
public class CustomCachingGlideModule extends AppGlideModule {  
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context).build();
        int defaultMemoryCacheSize = calculator.getMemoryCacheSize();
        int defaultBitmapPoolSize = calculator.getBitmapPoolSize();

        int customMemoryCacheSize = (int) (1.2 * defaultMemoryCacheSize);
        int customBitmapPoolSize = (int) (1.2 * defaultBitmapPoolSize);

        builder.setMemoryCache(new LruResourceCache(customMemoryCacheSize));
        builder.setBitmapPool(new LruBitmapPool(customBitmapPoolSize));
    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {
        // nothing to do here
    }
}
```

- 自定义硬盘缓存    

`Glide`提供了两种硬盘缓存方式`InternalCacheDiskCacheFactory`和`ExternalCacheDiskCacheFactory`。
```java
@GlideModule
public class CustomCachingGlideModule extends AppGlideModule {  
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // set disk cache size & external vs. internal
        int cacheSize100MegaBytes = 104857600;

        builder.setDiskCache(
                new InternalCacheDiskCacheFactory(context, cacheSize100MegaBytes));

        //builder.setDiskCache(
        //        new ExternalCacheDiskCacheFactory(context, cacheSize100MegaBytes));
    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {
        // nothing to do here
    }
}
```

上面的配置只是更改硬盘缓存的大小，并不会改变缓存目录，如果需要将缓存文件位置修改为指定位置，需要使用`DiskLruCacheFactory`类。 
```java
// or any other path
String downloadDirectoryPath = Environment.getDownloadCacheDirectory().getPath(); 

builder.setDiskCache(  
        new DiskLruCacheFactory( downloadDirectoryPath, cacheSize100MegaBytes )
);

// In case you want to specify a cache sub folder (i.e. "glidecache"):
//builder.setDiskCache(
//    new DiskLruCacheFactory( downloadDirectoryPath, "glidecache", cacheSize100MegaBytes ) 
//);
```

### 缓存总结 

通过上面的设置可以发现，`Glide`一共使用了三种缓存方式:    

- `Memory cache needs to implement: MemoryCache`
- `Bitmap pool needs to implement BitmapPool`
- `Disk cache needs to implement: DiskCache`


参考
===

- [Glide — Getting Started](https://futurestud.io/tutorials/glide-getting-started)
- [Glide — Advanced Loading](https://futurestud.io/tutorials/glide-advanced-loading)
- [Glide — ListAdapter (ListView, GridView)](https://futurestud.io/tutorials/glide-listadapter-listview-gridview)
- [Glide — Placeholders & Fade Animations](https://futurestud.io/tutorials/glide-placeholders-fade-animations)
- [Glide — Image Resizing & Scaling](https://futurestud.io/tutorials/glide-image-resizing-scaling)
- [Glide — Displaying Gifs & Video Thumbnails](https://futurestud.io/tutorials/glide-displaying-gifs-and-videos)
- [Glide — Caching Basics](https://futurestud.io/tutorials/glide-caching-basics)
- [Glide — Request Priorities](https://futurestud.io/tutorials/glide-request-priorities)
- [Glide — Thumbnails](https://futurestud.io/tutorials/glide-thumbnails)
- [Glide — Callbacks: SimpleTarget and ViewTarget for Custom View Classes](https://futurestud.io/tutorials/glide-callbacks-simpletarget-and-viewtarget-for-custom-view-classes)
- [Glide — Exceptions: Debugging and Error Handling](https://futurestud.io/tutorials/glide-exceptions-debugging-and-error-handling)
- [Glide — Custom Transformations](https://futurestud.io/tutorials/glide-custom-transformation)
- [Glide — Custom Animations with animate()](https://futurestud.io/tutorials/glide-custom-animations-with-animate)
- [Glide — Integrating Networking Stacks](https://futurestud.io/tutorials/glide-integrating-networking-stacks)
- [Glide — Customize Glide with Modules](https://futurestud.io/tutorials/glide-customize-glide-with-modules)
- [How to Rotate Images](https://futurestud.io/tutorials/glide-how-to-rotate-images)
- [Glide Module Example: Customize Caching](https://futurestud.io/tutorials/glide-module-example-customize-caching)
- [Glide Module Example: Optimizing By Loading Images In Custom Sizes](https://futurestud.io/tutorials/glide-module-example-optimizing-by-loading-images-in-custom-sizes)


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

