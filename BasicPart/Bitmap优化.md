Bitmap优化
===

1. 一个进程的内存可以由2个部分组成：`native和dalvik`
    `dalvik`就是我们平常说的`java`堆，我们创建的对象是在这里面分配的，而`bitmap`是直接在`native`上分配的。   
    一旦内存分配给`Java`后，以后这块内存即使释放后，也只能给`Java`的使用，所以如果`Java`突然占用了一个大块内存，
	即使很快释放了,`C`能用的内存也是16M减去`Java`最大占用的内存数。
    而`Bitmap`的生成是通过`malloc`进行内存分配的，占用的是`C`的内存，这个也就说明了，上述的`4MBitmap`无法生成的原因，
	因为在`13M`被`Java`用过后，剩下`C`能用的只有`3M`了。    

2. 在`Android`应用里，最耗费内存的就是图片资源。    
    在`Android`系统中，读取位图`Bitmap`时，分给虚拟机中的图片的堆栈大小只有8M，如果超出了，就会出现`OutOfMemory`异常。

3. 及时回收Bitmap的内存
    ```java
    // 先判断是否已经回收
    if(bitmap != null && !bitmap.isRecycled()){
        // 回收并且置为null
        bitmap.recycle();
        bitmap = null;
    }
    System.gc();
    ```

4. 捕获异常     
    在实例化`Bitmap`的代码中，一定要对`OutOfMemory`异常进行捕获。下面对初始化`Bitmap`对象过程中可能发生的`OutOfMemory`异常进行了捕获。
	如果发生了异常，应用不会崩溃，而是得到了一个默认的图片。
    ```java
    Bitmap bitmap = null;
    try {
        // 实例化Bitmap
        bitmap = BitmapFactory.decodeFile(path);
    } catch (OutOfMemoryError e) {
    //
    }
    if (bitmap == null) {
        // 如果实例化失败 返回默认的Bitmap对象
        return defaultBitmapMap;
    }
    ```
	
5. 缓存通用的Bitmap对象

6. 压缩图片
    如果图片像素过大可以将图片缩小，以减少载入图片过程中的内存的使用，避免异常发生。
    使用`BitmapFactory.Options.inSampleSize`就可以缩小图片。属性值`inSampleSize`表示缩略图大小为原始图片大小的几分之一。
	即如果这个值为2，则取出的缩略图的宽和高都是原始图片的1/2，图片的大小就为原始大小的1/4。
    如果知道图片的像素过大，就可以对其进行缩小。那么如何才知道图片过大呢?
    使用`BitmapFactory.Options`设置`inJustDecodeBounds`为`true`后，并不会真正的分配空间，即解码出来的`Bitmap`为`null`，
	但是可计算出原始图片的宽度和高度，即`options.outWidth`和`options.outHeight`。
    通过这两个值，就可以知道图片是否过大了。
    ```java
    BitmapFactory.Options opts = new BitmapFactory.Options();
    // 设置inJustDecodeBounds为true
    opts.inJustDecodeBounds = true;
    // 使用decodeFile方法得到图片的宽和高
    BitmapFactory.decodeFile(path, opts);
    // 打印出图片的宽和高
    Log.d("example", opts.outWidth + "," + opts.outHeight);
    ```
    在实际项目中，可以利用上面的代码，先获取图片真实的宽度和高度，然后判断是否需要跑缩小。如果不需要缩小，设置inSampleSize的值为1。如果需要缩小，则动态计算并设置inSampleSize的值，对图片进行缩小。需要注意的是，在下次使用BitmapFactory的decodeFile()等方法实例化Bitmap对象前，别忘记将opts.inJustDecodeBound设置回false。否则获取的bitmap对象还是null。

    以从Gallery获取一个图片为例讲解缩放:   
    ```java
    public class MainActivity extends Activity {
        private ImageView iv;
        private WindowManager wm;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            wm = getWindowManager();
            iv = (ImageView) findViewById(R.id.iv);
        }
    
        // 从系统的图库里面 获取一张照片
        public void click(View view) {
            Intent intent = new Intent();
            intent.setAction("android.intent.action.PICK");
            intent.addCategory("android.intent.category.DEFAULT");
            intent.setType("image/*");
            startActivityForResult(intent, 0);
        }
    
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            if (data != null) {
                // 获取到系统图库返回回来图片的uri
                Uri uri = data.getData();
                System.out.println(uri.toString());
    
                try {
                    InputStream is = getContentResolver().openInputStream(uri);
                    // 1.计算出来屏幕的宽高.
                    int windowWidth = wm.getDefaultDisplay().getWidth();
                    int windowHeight = wm.getDefaultDisplay().getHeight();
                    //2. 计算图片的宽高.
                    BitmapFactory.Options opts = new Options();
                    // 设置 不去真正的解析位图 不把他加载到内存 只是获取这个图片的宽高信息
                    opts.inJustDecodeBounds = true;
                    BitmapFactory.decodeStream(is, null, opts);
                    int bitmapHeight = opts.outHeight;
                    int bitmapWidth = opts.outWidth;
    
                    if (bitmapHeight > windowHeight || bitmapWidth > windowWidth) {
                        int scaleX = bitmapWidth/windowWidth;
                        int scaleY = bitmapHeight/windowHeight;
                        if(scaleX>scaleY){//按照水平方向的比例缩放
                            opts.inSampleSize = scaleX;
                        }else{//按照竖直方向的比例缩放
                            opts.inSampleSize = scaleY;
                        }
    
                    }else{//如果图片比手机屏幕小 不去缩放了.
                        opts.inSampleSize = 1;
                    }
                    //让位图工厂真正的去解析图片
                    opts.inJustDecodeBounds = false;
                    //注意: 流的操作
                    is = getContentResolver().openInputStream(uri);
                    Bitmap bitmap = BitmapFactory.decodeStream(is, null, opts);
                    iv.setImageBitmap(bitmap);
    
                } catch (Exception e) {
                    e.printStackTrace();
            }
            }
            super.onActivityResult(requestCode, resultCode, data);
        }
    }
    ```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 