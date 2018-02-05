Glide简介(上)
===

![image](https://github.com/bumptech/glide/blob/master/static/glide_logo.png)


官网:[Glide](https://github.com/bumptech/glide)[![Maven Central](https://img.shields.io/badge/version-4.1.1-brightgreen.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide) 

`Glide`是`Google`员工的开源项目，`Google`官方`App`中已经使用，在2015年的`Google I/O`上被推荐。

`Glide`的优点:   

- 使用简单
- 可配置度高，自适应程度高
- 支持多种数据源，网络、本地、资源、`Assets`等
- 支持`Gif`图片。
- 支持`WebP`。
- 加载速度快、流畅度高。
- `Glide`的`with()`方法不光接受`Context`，还接受`Activity`和`Fragment`，这样图片加载会和`Activity/Fragment`的生命周期保持一致，比如`Pause`状态在暂停加载，在`Resume`的时候又自动重新加载。
- 支持设置图片加载优先级。
- 支持缩略图，可以在同一时间加载多张图片到同一个`ImageView`中，例如可以首先加载只有`ImageView`十分之一大小的缩略图，然后等再加载完整大小的图片后会再显示到该`ImageView`上。
- 内存占用低，`Glide`默认的`Bitmap`格式是`RGB_565`，比`ARGB_8888`格式的内存开销要小一半,所以图片质量会稍微差一些，当然这些配置都是可以修改的。
- `Glide`缓存的图片大小是根据`ImageView`尺寸来缓存的的。这种方式优点是加载显示非常快。且可以设置缓存图片的尺寸  
- 默认使用`HttpUrlConnection`下载图片，可以配置为`OkHttp`或者`Volley`下载，也可以自定义下载方式。
- 默认使用两个线程池来分别执行读取缓存和下载任务，且可以自定义。
- 默认使用手机内置存储进行磁盘缓存，可以配置为外部存储，可以配置缓存大小，图片池大小。
- 在加载同样配置的图片时，`Glide`内存占用更少，因为`Glide`是针对每个`ImageView`适配图片大小后再存储到磁盘的，这样加载进内存的是压缩过的图片，内存占用自然就比较少。这种做法有助于减少`OutOfMemoryError`的出现。
- 高效处理`Bitmap`，使用`Bitmap Pool`来对`Bitmap`进行复用，主动调用`recycle`回收需要回收的`Bitmap`，减小系统回收压力

缺点:   

- 体积相对来说比较大，目前最新版的大小在`500k`左右  
- 当我们从远程`URL`地址下载图片时，`Picasso`相比`Glide`要快很多。可能的原因是`Picasso`下载完图片后直接将整个图片加载进内存，而`Glide`还需要针对每个`ImageView`的大小来适配压缩下载到的图片，这个过程需要耗费一定的时间。（当然我们可以使用`thumbnail()`来减少压缩的时间）

### 使用方法：   

```java
repositories {
  mavenCentral()
  maven { url 'https://maven.google.com' }
}

dependencies {
  compile 'com.github.bumptech.glide:glide:4.1.1'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.1.1'
}
```

`Java`代码使用:   

和`Picasso`一样，`Glide`使用`fluent interface`的模式，想要加载图片，需要至少传入三个参数:    

- `with(Context context)`:`Context`是很多`android api`所必要的参数，`glide`也一样。可以传递`Activity/Fragment`，而且
它会由`Activity/Fragment`的生命周期进行绑定。  
- `load(String imageUrl)`:图片的`URL`地址。
- `into(ImageView targetImageView)`:需要将加载的图片显示到的对应的`ImageView`。


```java
// For a simple view:
@Override public void onCreate(Bundle savedInstanceState) {
  ...
  ImageView imageView = (ImageView) findViewById(R.id.my_image_view);

  GlideApp.with(this).load("http://goo.gl/gEgYUd").into(imageView);
}

// For a simple image list:
@Override public View getView(int position, View recycled, ViewGroup container) {
  final ImageView myImageView;
  if (recycled == null) {
    myImageView = (ImageView) inflater.inflate(R.layout.my_image_view, container, false);
  } else {
    myImageView = (ImageView) recycled;
  }

  String url = myUrls.get(position);

  GlideApp
    .with(myFragment)
    .load(url)
    .centerCrop()
    .placeholder(R.drawable.loading_spinner)
    .into(myImageView);

  return myImageView;
}
```


`Proguard`防混淆配置:   
```java
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public class * extends com.bumptech.glide.AppGlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}

# for DexGuard only
-keepresourcexmlelements manifest/application/meta-data@value=GlideModule
```


### 接下来是一些常用的方法:    

- 从网络加载图片    

```java
GlideApp.with(context).load(internetUrl).into(targetImageView);
```

- 从文件加载图片  

```java
File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),"Test.jpg");
GlideApp.with(context).load(file).into(imageViewFile);
```

- 加载`resource`资源   

```java
int resourceId = R.mipmap.ic_launcher;
GlideApp.with(context).load(resourceId).into(imageViewResource);
```

- 加载`URI`地址 

```java
GlideApp.with(context).load(uri).into(imageViewUri);
```

- 设置占位图      

```java
GlideApp  
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .into(imageViewPlaceholder);
```

- 设置出错时的图片  

```java
GlideApp  
    .with(context)
    .load("http://futurestud.io/non_existing_image.png")
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .error(R.mipmap.future_studio_launcher) // will be displayed if the image cannot be loaded
    .into(imageViewError);
```

- `fallback`占位图  

上面的`error()`展位图是当资源无法获取时使用的，例如图片地址无效，但是还有另外一种情况是你传递了`null`值。比如在一个显示用户资料列表中，由于并不是
每个人都有头像图片，所有这时可能会传递`null`值。如果想要指定一个在传递`null`值时显示的错误图片可以使用`.fallback()`.    
```java
String nullString = null; // could be set to null dynamically

GlideApp  
    .with(context)
    .load( nullString )
    .fallback( R.drawable.floorplan )
    .into( imageViewNoFade );
```

- 结合`ListView`使用 

```java
public static String[] eatFoodyImages = {
        "http://i.imgur.com/rFLNqWI.jpg",
        "http://i.imgur.com/C9pBVt7.jpg",
        "http://i.imgur.com/rT5vXE1.jpg",
        "http://i.imgur.com/aIy5R2k.jpg",
        "http://i.imgur.com/MoJs9pT.jpg",
        "http://i.imgur.com/S963yEM.jpg",
        "http://i.imgur.com/rLR2cyc.jpg",
        "http://i.imgur.com/SEPdUIx.jpg",
        "http://i.imgur.com/aC9OjaM.jpg",
        "http://i.imgur.com/76Jfv9b.jpg",
        "http://i.imgur.com/fUX7EIB.jpg",
        "http://i.imgur.com/syELajx.jpg",
        "http://i.imgur.com/COzBnru.jpg",
        "http://i.imgur.com/Z3QjilA.jpg",
};

public class UsageExampleAdapter extends AppCompatActivity {  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_usage_example_adapter);

        listView.setAdapter(
            new ImageListAdapter(
                UsageExampleAdapter.this, 
                eatFoodyImages
            )
        );
    }
}

public class ImageListAdapter extends ArrayAdapter {  
    private Context context;
    private LayoutInflater inflater;

    private String[] imageUrls;

    public ImageListAdapter(Context context, String[] imageUrls) {
        super(context, R.layout.listview_item_image, imageUrls);

        this.context = context;
        this.imageUrls = imageUrls;

        inflater = LayoutInflater.from(context);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (null == convertView) {
            convertView = inflater.inflate(R.layout.listview_item_image, parent, false);
        }

        GlideApp
            .with(context)
            .load(imageUrls[position])
            .into((ImageView) convertView);

        return convertView;
    }
}
```

- 图片过度`Transitions`      

无论你是否使用占位图，在`UI`过程中改变`ImageView`的图片都是一个很大的动作。有一个简单的方法可以使这种改变变的更平滑，更容易让人接受，那就是使用
`crossfade`动画。   
```java
GlideApp  
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .error(R.mipmap.future_studio_launcher) // will be displayed if the image cannot be loaded
    .transition(DrawableTransitionOptions.withCrossFade())// withCrossFade(int duration)方法可以传入时间，默认时间是300毫秒
    .into(imageViewCombined);
```

- 自定义过度动画

上面提供了`crossfade`动画，但是有些时候我们需要自定义更多的样式。 
`Glide`也是支持`xml`中自定义的动画文件的。    
```java
GlideApp  
    .with(context)
      .load(eatFoodyImages[0])
      .transition(GenericTransitionOptions.with(R.anim.zoom_in))
      .into(imageView1);
```

- 图片大小调整     

理想情况下，你的服务器或者`API`能够返回所需分辨率的图片，这是在网络带宽、内存消耗和图片质量下的完美方案。

跟`Picasso`比起来，`Glide`在内存上占用更优化。`Glide`在缓存和内存里自动限制图片的大小去适配`ImageView`的尺寸。`Picasso`也有同样的能力，但需要调用`fit()`方法。用`Glide`时，如果图片不需要自动适配`ImageView`，调用`override(horizontalSize, verticalSize)`，它会在将图片显示在`ImageView`之前调整图片的大小。

这个设置也有利于没有明确目标，但已知尺寸的视图上。例如，如果`app`想要预先缓存在`splash`屏幕上，还没法测量出`ImageVIews`具体宽高。但是如果你已经知道图片应当为多大，使用`override`可以提供一个指定的大小的图片。

```java
GlideApp  
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200) // resizes the image to these dimensions (in pixel). resize does not respect aspect ratio
    .into(imageViewResize);
```

- 缩放图片

对于任何图像的任何处理，调整图像的大小可能会扭曲长宽比，丑化图片的显示。在大多数情况下，你希望防止这种事情发生。`Glide`提供了变换去处理图片显示，通过设置`centerCrop`和`fitCenter`，可以得到两个不同的效果。`CenterCrop()`会缩放图片让图片充满整个`ImageView`的边框，然后裁掉超出的部分。`ImageVIew`会被完全填充满，但是图片可能不能完全显示出。
`fitCenter()`会缩放图片让两边都相等或小于`ImageView`的所需求的边框。图片会被完整显示，可能不能完全填充整个`ImageView`。
```java
GlideApp  
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200) // resizes the image to these dimensions (in pixel)
    .centerCrop() // this cropping technique scales the image so that it fills the requested bounds and then crops the extra.
    .into(imageViewResizeCenterCrop);

GlideApp  
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200)
    .fitCenter() 
    .into(imageViewResizeFitCenter);    
```

- 更变图片转换 

上面介绍了两种自带的图片转换方式`fitCenter`和`centerCrop`。 但有些时候我们需要自定义一些别的转换方式，自定义转换方式需要继承`BitmapTransformation`类。  
```java
public class BlurTransformation extends BitmapTransformation {

    public BlurTransformation(Context context) {
        super( context );
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return null; // todo
    }

    @Override
    public String getId() {
        return null; // todo
    }
}
```
接下来使用`Renderscript`来实现图片的模糊处理。    

```java
public class BlurTransformation extends BitmapTransformation {

    private RenderScript rs;

    public BlurTransformation(Context context) {
        super( context );

        rs = RenderScript.create( context );
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        Bitmap blurredBitmap = toTransform.copy( Bitmap.Config.ARGB_8888, true );

        // Allocate memory for Renderscript to work with
        Allocation input = Allocation.createFromBitmap(
            rs, 
            blurredBitmap, 
            Allocation.MipmapControl.MIPMAP_FULL, 
            Allocation.USAGE_SHARED
        );
        Allocation output = Allocation.createTyped(rs, input.getType());

        // Load up an instance of the specific script that we want to use.
        ScriptIntrinsicBlur script = ScriptIntrinsicBlur.create(rs, Element.U8_4(rs));
        script.setInput(input);

        // Set the blur radius
        script.setRadius(10);

        // Start the ScriptIntrinisicBlur
        script.forEach(output);

        // Copy the output to the blurred bitmap
        output.copyTo(blurredBitmap);

        toTransform.recycle();

        return blurredBitmap;
    }

    @Override
    public String getId() {
    	// getId()方法为这个变换描述了一个独有的识别。Glide使用那个关键字作为缓存系统的一部分。防止出现异常问题，确保其唯一。
        return "blur";
    }
}
```
传递你的类的实例作为`.transform()`的参数。不管是图片还是`gif`都可以进行变换。

```java
GlideApp  
	.with(context)
	.load(eatFoodyImages[0])
	.transform(new BlurTransformation(context))
	.into(imageView2);
```

通常，`Glide`的`fluent interface`允许方法被连接在一起，然而变换并不是这样的。确保你只调用`.transform()`一次，不然之前的设置将会被覆盖！然而，你可以通过传递多个转换对象当作参数到`.transform()`中来进行多重变换:   

```java
GlideApp  
    .with(context)
    .load(eatFoodyImages[0])
    .transform(
        new MultiTransformation(
        new GrayscaleTransformation(context),
        new BlurTransformation(context)))
    .into(imageView3);
```

- 播放`gif`动画    

```java
String gifUrl = "http://i.kinja-img.com/gawker-media/image/upload/s--B7tUiM5l--/gf2r69yorbdesguga10i.gif";

GlideApp  
    .with(context)
    .load(gifUrl)
    .into(imageViewGif);
```

- `gif`检查     

`Glide`接受`Gif`和图片作为`load()`的参数。上面代码中潜在的一个问题，如果提供的源不是`Gif`，可能是一个普通的图片。即使是一个完好的图片(非`Gif`)，`Glide`也会加载失败。`.error()`回调方法会被调用，并加载错误占位图。这样引入了一个额外的方法`.asGif()`强迫生成一个`Gif`。 

```java
GlideApp  
    .with(context)
    .asGif()
    .load(gifUrl)
    .error(R.drawable.full_cake)
    .into(imageViewGifAsGif);
```

- 把`Gif`当作`Bitmap`播放

如果你的`app`需要显示一组网络`URL`，可能包括普通的图片或者`Gif`。在一些情况下，你可能并不在意是否要播放完整的`Gif`。如果你只是想要显示`Gif`的第一帧，当`URl`指向的的确是`Gif`，你可以调用`asBitmap()`将其作为常规图片显示。

```java
GlideApp  
    .with(context)
    .asBitmap()
    .load(gifUrl)
    .into(imageViewGifAsBitmap);
```

- 显示本地视频缩略图   

```java
String filePath = "/storage/emulated/0/Pictures/example_video.mp4";

GlideApp  
    .with(context)
    .asBitmap()
      .load(Uri.fromFile(new File(filePath)))
    .into(imageViewGifAsBitmap);
```

- 缓存设置   

```java
GlideApp  
    .with(context)
    .load(gifUrl)
    .asGif()
    .error(R.drawable.full_cake)
    .diskCacheStrategy(DiskCacheStrategy.DATA)
    .into(imageViewGif);
```


- 内存缓存     

默认情况下，我们不用去特意的操作缓存设置，因为`Glide`默认会使用内存和硬盘缓存，但是如果在知道某一个图片会快速变化时，你可能会关闭缓存功能。 

```java
GlideApp  
    .with(context)
      .load(eatFoodyImages[0])
      .skipMemoryCache(true) 
      .into(imageView1);
```
上面调用了`.skipMemoryCache(true)`方法来告诉`Glide`禁用内存缓存功能。这就意味着`Glide`不会将图片缓存到内存中，但是这只是影响内存缓存，`Glide`仍然会将图片
缓存到硬盘中来避免下一次显示该图片时重复请求。   


- 硬盘缓存 

如上面所讲到的，即使你关闭了内存缓存，所请求的图片仍然会被保存在设备的磁盘存储上。如果你有一张不段变化的图片，但是都是用的同一个`URL`，你可能需要禁止磁盘缓存了。
你可以用`.diskCacheStrategy()`方法改变`Glide`的行为。不同于`.skipMemoryCache()`方法，它将需要从枚举型变量中选择一个，而不是一个简单的`boolean`。如果你想要禁止请求的磁盘缓存，使用枚举型变量`DiskCacheStrategy.NONE`作为参数。

```java
GlideApp  
    .with(context)
      .load(eatFoodyImages[1])
      .diskCacheStrategy(DiskCacheStrategy.NONE)
      .into(imageView2);
```
上面的这种方式只是会禁用硬盘缓存，`Glide`还会使用内存缓存。如果想把内存缓存和硬盘缓存都禁用，需要把上面的两个方法都设置   
```java
GlideApp  
    .with(context)
      .load(eatFoodyImages[1])
      .diskCacheStrategy(DiskCacheStrategy.NONE)
      .skipMemoryCache(true)
      .into(imageView2);
```

- 自定义磁盘缓存行为

如从上面提到的，`Glide`为硬盘缓存提供了多种设置方式。`Glide`的磁盘缓存是相当复杂的。例如，`Picasso`只缓存全尺寸图片。`Glide`，会缓存原始，全尺寸的图片和额外的小版本图片。例如，如果你请求一个`1000x1000`像素的图片，你的`ImageView`是`500x500`像素，`Glide`会保存两个版本的图片到缓存里。

- `DiskCacheStrategy.NONE`禁用硬盘缓存功能
- `DiskCacheStrategy.DATA`只缓存原始的全尺寸图. 例如上面例子中`1000x1000`像素的图片
- `DiskCacheStrategy.RESOURCE`只缓存最终剪辑转换后降低分辨的图片。例如上面离职中`500x500`像素的图片
- `DiskCacheStrategy.AUTOMATIC`基于资源只能选择缓存策略(默认的行为)
- `DiskCacheStrategy.ALL`缓存所有分辨率对应的类型的图片

- 图片请求优先级     

`Glide`支持使用`.priority()`方法来设置图片请求的优先级。   
- `Priority.LOW`
- `Priority.NORMAL`
- `Priority.HIGH`
- `Priority.IMMEDIATE`

```java
private void loadImageWithHighPriority() {  
    GlideApp
        .with(context)
        .load(UsageExampleListViewAdapter.eatFoodyImages[0])
        .priority(Priority.HIGH)
        .into(imageViewHero);
}

private void loadImagesWithLowPriority() {  
    GlideApp
        .with(context)
        .load(UsageExampleListViewAdapter.eatFoodyImages[1])
        .priority(Priority.LOW)
        .into(imageViewLowPrioLeft);

    GlideApp
        .with(context)
        .load(UsageExampleListViewAdapter.eatFoodyImages[2])
        .priority(Priority.LOW)
        .into(imageViewLowPrioRight);
}
```

- 缩略图  

缩略图不同于前面文章中提到的占位图。占位图应当是跟`app`绑定在一起的资源。缩略图是一个动态的占位图，可以从网络加载。缩略图也会被先加载，直到实际图片请求加载完毕。如果因为某些原因，缩略图获得的时间晚于原始图片，它并不会替代原始图片，而是简单地被忽略掉。

`Glide`提供了两种产生缩略图的方式。第一种，是通过在加载的时候指定一个小的分辨率，产生一个缩略图。这个方法在`ListView`和详细视图的组合中非常有用。如果你已经在`ListView`中用到了`250x250`像素的图片，那么在在详细视图中会需要一个更大分辨率的图片。然而从用户的角度，我们已经看见了一个小版本的图片，为什么需要好几秒，同样的图片（高分辨率的）才能被再次加载出来呢？

在这种情况下，从显示`250x250`像素版本的图片平滑过渡到详细视图里查看大图更有意义。`Glide`里的`.thumbnail()`方法让这个变为可能。这里`.thumbnal()`的参数是一个浮点乘法运算。


```java
String internetUrl = "http://i.imgur.com/DvpvklR.png";

GlideApp  
    .with(context)
    .load(internetUrl)
    .thumbnail(0.1f)
    .into(imageView);
```
例如，如果你传递一个`0.1f`作为参数，`Glide`会加载原始图片大小的`10%`的图片。如果原始图片有`1000x1000`像素，缩略图的分辨率为`100x100`像素。

为`.thumbnail()`传入一个浮点类型的参数，非常简单有效，但并不是总是有意义。如果缩略图的生成也需要从网络加载同样全分辨率图片后才可以，那这样加载速度并不会比不用缩略图快。
因此`Glide`提供了另一个方法去加载和显示缩略图。第二种方式需要传递一个新的`Glide`请求作为参数。      

```java
String internetUrl = "http://i.imgur.com/DvpvklR.png";

// setup Glide request without the into() method
RequestBuilder<Drawable> thumbnailRequest = GlideApp  
    .with(context)
    .load(internetUrl);

// pass the request as a a parameter to the thumbnail request
GlideApp  
    .with(context)
      .load(UsageExampleGifAndVideos.gifUrl)
      .thumbnail(thumbnailRequest)
      .into(imageView);
```
区别在于第一个缩略图请求是完全独立于第二个原始请求的。缩略图可以来自不同资源或者图片URL，你可以在它上面应用不同的变换。



- 旋转图片    

`android.graphics.Matrix`提供了旋转的方法:  
```java
Bitmap toTransform = ... // your bitmap source

Matrix matrix = new Matrix();  
matrix.postRotate(rotateRotationAngle);

Bitmap.createBitmap(toTransform, 0, 0, toTransform.getWidth(), toTransform.getHeight(), matrix, true);  
```
但是如果要在`Glide`中使用旋转方法，需要用`BitmapTransformation()`进行包裹:   
```java
public class RotateTransformation extends BitmapTransformation {

    private float rotateRotationAngle = 0f;

    public RotateTransformation(Context context, float rotateRotationAngle) {
        super( context );

        this.rotateRotationAngle = rotateRotationAngle;
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        Matrix matrix = new Matrix();

        matrix.postRotate(rotateRotationAngle);

        return Bitmap.createBitmap(toTransform, 0, 0, toTransform.getWidth(), toTransform.getHeight(), matrix, true);
    }

    @Override
    public String getId() {
        return "rotate" + rotateRotationAngle;
    }
}
```
然后将上面的方法传递到`.transform()`中:  
```java
private void loadImageOriginal() {  
    Glide
        .with( context )
        .load( eatFoodyImages[0] )
        .into( imageView1 );
}

private void loadImageRotated() {  
    Glide
        .with( context )
        .load( eatFoodyImages[0] )
        .transform( new RotateTransformation( context, 90f ))
        .into( imageView3 );
}
```


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

