自定义View详解
===

虽然之前也分析过[View绘制过程](https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/View%E7%BB%98%E5%88%B6%E8%BF%87%E7%A8%8B%E8%AF%A6%E8%A7%A3.md)，但是如果让我自己集成`ViewGroup`然后自己重新`onMeasure,onLayout,onDraw`方法自定义`View`我还是会头疼。今天索性来系统的学习下。

### `onMeasure`

```java
/**
     * <p>
     * Measure the view and its content to determine the measured width and the
     * measured height. This method is invoked by {@link #measure(int, int)} and
     * should be overridden by subclasses to provide accurate and efficient
     * measurement of their contents.
     * </p>
     *
     * <p>
     * <strong>CONTRACT:</strong> When overriding this method, you
     * <em>must</em> call {@link #setMeasuredDimension(int, int)} to store the
     * measured width and height of this view. Failure to do so will trigger an
     * <code>IllegalStateException</code>, thrown by
     * {@link #measure(int, int)}. Calling the superclass'
     * {@link #onMeasure(int, int)} is a valid use.
     * </p>
     *
     * <p>
     * The base class implementation of measure defaults to the background size,
     * unless a larger size is allowed by the MeasureSpec. Subclasses should
     * override {@link #onMeasure(int, int)} to provide better measurements of
     * their content.
     * </p>
     *
     * <p>
     * If this method is overridden, it is the subclass's responsibility to make
     * sure the measured height and width are at least the view's minimum height
     * and width ({@link #getSuggestedMinimumHeight()} and
     * {@link #getSuggestedMinimumWidth()}).
     * </p>
     *
     * @param widthMeasureSpec horizontal space requirements as imposed by the parent.
     *                         The requirements are encoded with
     *                         {@link android.view.View.MeasureSpec}.
     * @param heightMeasureSpec vertical space requirements as imposed by the parent.
     *                         The requirements are encoded with
     *                         {@link android.view.View.MeasureSpec}.
     *
     * @see #getMeasuredWidth()
     * @see #getMeasuredHeight()
     * @see #setMeasuredDimension(int, int)
     * @see #getSuggestedMinimumHeight()
     * @see #getSuggestedMinimumWidth()
     * @see android.view.View.MeasureSpec#getMode(int)
     * @see android.view.View.MeasureSpec#getSize(int)
     */
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

注释说的非常清楚。但是我还是要强调一下这两个参数:`widthMeasureSpec`和`heightMeasureSpec`这两个int类型的参数，看名字应该知道是跟宽和高有关系，但它们其实不是宽和高，而是由宽、高和各自方向上对应的模式来合成的一个值：其中，在int类型的32位二进制位中，31-30这两位表示模式，0~29这三十位表示宽和高的实际值.其中模式一共有三种，被定义在Android中的View类的一个内部类中：View.MeasureSpec：

```java
android.view
public static class View.MeasureSpec
extends Object
A MeasureSpec encapsulates the layout requirements passed from parent to child. Each MeasureSpec represents a requirement for either the width or the height. A MeasureSpec is comprised of a size and a mode. There are three possible modes:
UNSPECIFIED
The parent has not imposed any constraint on the child. It can be whatever size it wants.
EXACTLY
The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
AT_MOST
The child can be as large as it wants up to the specified size.
MeasureSpecs are implemented as ints to reduce object allocation. This class is provided to pack and unpack the <size, mode> tuple into the int.
```

- MeasureSpec.UNSPECIFIED The parent has not imposed any constraint on the child. It can be whatever size it wants. 这种情况比较少，一般用不到。标示父控件没有给子View任何显示- - - -对应的二进制表示: 00
- MeasureSpec.EXACTLY The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
理解成MATCH_PARENT或者在布局中指定了宽高值，如layout:width=’50dp’. - - - - 对应的二进制表示:01
- MeasureSpec.AT_MOST The child can be as large as it wants up to the specified size.理解成WRAP_CONTENT,这是的值是父View可以允许的最大的值，只要不超过这个值都可以。- - - - 对应的二进制表示:10


那具体`MeasureSpec`是怎么把宽和高的实际值以及模式组合起来变成一个int类型的值呢？ 这部分是在`MeasureSpce.makeMeasureSpec()`方法中处理的:    
```java
public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
```
那我们如何从MeasureSpec值中提取模式和大小呢？该方法内部是采用位移计算.
```java
/**
 * Extracts the mode from the supplied measure specification.
 *
 * @param measureSpec the measure specification to extract the mode from
 * @return {@link android.view.View.MeasureSpec#UNSPECIFIED},
 *         {@link android.view.View.MeasureSpec#AT_MOST} or
 *         {@link android.view.View.MeasureSpec#EXACTLY}
 */
public static int getMode(int measureSpec) {
    return (measureSpec & MODE_MASK);
}

/**
 * Extracts the size from the supplied measure specification.
 *
 * @param measureSpec the measure specification to extract the size from
 * @return the size in pixels defined in the supplied measure specification
 */
public static int getSize(int measureSpec) {
    return (measureSpec & ~MODE_MASK);
}
```


### `onLayout`

为了能合理的去绘制定义`View`,你需要制定它的大小。复杂的自定义`View`通常需要根据屏幕的样式和大小来进行复杂的布局计算。你不应该假设你的屏幕上的`View`的大小。即使只有一个应用使用你的自定义`View`，也需要处理不同的屏幕尺寸、屏幕密度和横屏以及竖屏下的多种比率等。

虽然`View`有很多处理测量的方法，但他们中的大部分都不需要被重写。如果你的`View`不需要特别的控制它的大小，你只需要重写一个方法:`onSizeChanged()`。

`onSizeChanged()`方法会在你的`View`第一次指定大小后调用，在因某些原因改变大小后会再次调用。在上面`PieChart`的例子中，`onSizeChanged()`方法就是它需要重新计算表格样式和大小以及其他元素的地方。
下面就是`PieChart.onSizeChanged()`方法的内容:      

```java
// Account for padding
float xpad = (float)(getPaddingLeft() + getPaddingRight());
float ypad = (float)(getPaddingTop() + getPaddingBottom());

// Account for the label
if (mShowText) xpad += mTextWidth;

float ww = (float)w - xpad;
float hh = (float)h - ypad;

// Figure out how big we can make the pie.
float diameter = Math.min(ww, hh);
```


### `onDraw`

自定义`View`最重要的就是展现样式。

##### 重写`onDraw()`方法

绘制自定义`View`最重要的步骤就是重写`onDraw()`方法。`onDraw()`方法的参数是`Canvas`对象。可以用它来绘制自身。`Canvas`类定义了绘制文字、线、位图和很多其他图形的方法。你可以在`onDraw()`方法中使用这些方法来指定`UI`.

在使用任何绘制方法之前，你都必须要创建一个`Paint`对象。  

##### 创建绘制的对象

`android.graphics`框架将绘制分为两步:     

- 绘制什么，由`Canvas`处理。
- 怎么去绘制，由`Paint`处理。


##### `Canvas`

> The Canvas class holds the "draw" calls. To draw something, you need 4 basic components: A Bitmap to hold the pixels,  
 a Canvas to host the draw calls (writing into the bitmap), a drawing primitive (e.g. Rect, Path, text, Bitmap),   
and a paint (to describe the colors and styles for the drawing).  


- `Canvas()`:创建一个空的画布，可以使用`setBitmap()`方法来设置绘制的具体画布； 
- `Canvas(Bitmap bitmap)`:以`bitmap`对象创建一个画布，则将内容都绘制在`bitmap`上，`bitmap`不得为`null`; 
- `canvas.drawRect(RectF,Paint)`方法用于画矩形，第一个参数为图形显示区域，第二个参数为画笔，设置好图形显示区域`Rect`和画笔`Paint`后，即可画图； 
- `canvas.drawRoundRect(RectF, float, float, Paint)`方法用于画圆角矩形，第一个参数为图形显示区域，第二个参数和第三个参数分别是水平圆角半径和垂直圆角半径。 
- `canvas.drawLine(startX, startY, stopX, stopY, paint)`：前四个参数的类型均为`float`，最后一个参数类型为`Paint`。表示用画笔`paint`从点`（startX,startY）`到点`（stopX,stopY）`画一条直线； 
- `canvas.drawLines (float[] pts, Paint paint)``pts`:是点的集合，大家下面可以看到，这里不是形成连接线，而是每两个点形成一条直线，`pts`的组织方式为`｛x1,y1,x2,y2,x3,y3,……｝`，例如`float []pts={10,10,100,100,200,200,400,400};`就是有四个点：（10，10）、（100，100），（200，200），（400，400）），两两连成一条直线；
- `canvas.drawArc(oval, startAngle, sweepAngle, useCenter, paint)`：第一个参数`oval`为`RectF`类型，即圆弧显示区域，`startAngle`和`sweepAngle`均为`float`类型，分别表示圆弧起始角度和圆弧度数,3点钟方向为0度，`useCenter`设置是否显示圆心，`boolean`类型，`paint`为画笔； 
- `canvas.drawCircle(float,float, float, Paint)`方法用于画圆，前两个参数代表圆心坐标，第三个参数为圆半径，第四个参数是画笔； 
- `canvas.drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)` 位图，参数一就是我们常规的`Bitmap`对象，参数二是源区域(这里是`bitmap`)，参数三是目标区域(应该在`canvas`的位置和大小)，参数四是`Paint`画刷对象，因为用到了缩放和拉伸的可能，当原始`Rect`不等于目标`Rect`时性能将会有大幅损失。
- `canvas.drawText(String text, float x, floaty, Paint paint)`渲染文本，`Canvas`类除了上
面的还可以描绘文字，参数一是`String`类型的文本，参数二`x`轴，参数三`y`轴，参数四是`Paint`对象。
- `canvas.drawPath (Path path, Paint paint)`，根据`Path`去画.
    ```java
    Path path = new Path();  
    path.moveTo(10, 10); //设定起始点  
    path.lineTo(10, 100);//第一条直线的终点，也是第二条直线的起点  
    path.lineTo(300, 100);//画第二条直线  
    path.lineTo(500, 100);//第三条直线  
    path.close();//闭环  
    canvas.drawPath(path, paint);  
    ```

##### `Paint`

- `setARGB(int a, int r, int g, int b)` 设置`Paint`对象颜色，参数一为`alpha`透明值
- `setAlpha(int a)` 设置`alpha`不透明度，范围为0~255
- `setAntiAlias(boolean aa)`是否抗锯齿
- `setColor(int color)`设置颜色
- `setTextScaleX(float scaleX)`设置文本缩放倍数，1.0f为原始
- `setTextSize(float textSize)`设置字体大小
- `setUnderlineText(String underlineText)`设置下划线



例如，`Canvas`提供了一个画一条线的方法，而`Paint`提供了指定这条线的颜色的方法。`Canvas`提供了绘制长方形的方法，而`Paint`提供了是用颜色填充整个长方形还是空着的方法。简单的说，`Canvas`指定了你想在屏幕上绘制的形状，而`Paint`指定了你要绘制的形状的颜色、样式、字体和样式等等。   

所以，在你`draw`任何东西之前，你都需要创建一个或者多个`Paint`对象。下面的`PieChart`例子就是在构造函数中调用的`init`方法:     

```java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }

   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);

   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));

   ...
```

下面是`PieChart`完整的`onDraw()`方法:      
```java
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   // Draw the shadow
   canvas.drawOval(
           mShadowBounds,
           mShadowPaint
   );

   // Draw the label text
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);

   // Draw the pie slices
   for (int i = 0; i < mData.size(); ++i) {
       Item it = mData.get(i);
       mPiePaint.setShader(it.mShader);
       canvas.drawArc(mBounds,
               360 - it.mEndAngle,
               it.mEndAngle - it.mStartAngle,
               true, mPiePaint);
   }

   // Draw the pointer
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```


下面是一张`View`绘制过程中框架调用的一些标准方法概要图:      
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/custom_view_methods.png?raw=true)

下面来几个例子:    

自定义开关:      

```java
public class ToogleView extends View {
    private int mSlideMarginLeft = 0;
    private Bitmap backgroundBitmap;
    private Bitmap slideButton;


    public ToogleView(Context context) {
        super(context);
        init(context);
    }

    public ToogleView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context);
    }

    public ToogleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
        backgroundBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.toogle_bg);
        slideButton = BitmapFactory.decodeResource(getResources(),
                R.drawable.toogle_slide);
        this.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSlideMarginLeft == 0) {
                    mSlideMarginLeft = backgroundBitmap.getWidth() - slideButton.getWidth();
                } else {
                    mSlideMarginLeft = 0;
                }
                invalidate();
            }
        });
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Paint paint = new Paint();
        paint.setAntiAlias(true);
        // 先画背景图
        canvas.drawBitmap(backgroundBitmap, 0, 0, paint);
        // 再画滑块，用mSlideMarginLeft来控制滑块距离左边的距离。
        canvas.drawBitmap(slideButton, mSlideMarginLeft, 0, paint);
    }
```

```xml
<LinearLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android" >

<com.charon.recyclerviewdemo.ToogleView
        android:paddingLeft="50dp"
        android:background="@android:color/holo_green_light"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    </LinearLayout>
```

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/toogle_1.png?raw=true)
很明显显示的不对，因为高设置为`warp_content`了，但是界面显示的确实整个屏幕，而且`paddingLeft`也没生效，那该怎么做呢？ 当然是重写`onMeasure()` 方法:   
```java
public class ToogleView extends View {
    private int mSlideMarginLeft = 0;
    private Bitmap backgroundBitmap;
    private Bitmap slideButton;


    public ToogleView(Context context) {
        super(context);
        init(context);
    }

    public ToogleView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context);
    }

    public ToogleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
        backgroundBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.toogle_bg);
        slideButton = BitmapFactory.decodeResource(getResources(),
                R.drawable.toogle_slide);
        this.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSlideMarginLeft == 0) {
                    mSlideMarginLeft = backgroundBitmap.getWidth() - slideButton.getWidth();
                } else {
                    mSlideMarginLeft = 0;
                }
                invalidate();
            }
        });
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
        int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);

        int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
        int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);
        int width;
        int height;
        if (MeasureSpec.EXACTLY == measureWidthMode) {
            width = measureWidth;
        } else {
            width = backgroundBitmap.getWidth();
        }

        if (MeasureSpec.EXACTLY == measureHeightMode) {
            height = measureHeight;
        } else {
            height = backgroundBitmap.getHeight();
        }

        setMeasuredDimension(width, height);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Paint paint = new Paint();
        paint.setAntiAlias(true);
        canvas.drawBitmap(backgroundBitmap, getPaddingLeft(), 0, paint);
        canvas.drawBitmap(slideButton, mSlideMarginLeft + getPaddingLeft(), 0, paint);
    }

}

```
这样就可以了。简单的说明一下，就是如果当前的模式是`EXACTLY`那就把父`View`传递进来的宽高设置进来，如果是`AT_MOST`或者`UNSPECIFIED`的话就使用背景图片的宽高。


最后再来一个自定义`ViewGroup`的例子:       

之前的引导页面都是通过类似`ViewPager`这种方法左右滑动，现在想让他上下滑动，该怎么弄呢？    
```java
public class VerticalLayout extends ViewGroup {
    public VerticalLayout(Context context) {
        super(context);
    }
    public VerticalLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public VerticalLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        
    }
}
```
继承`ViewGroup`必须要重写`onLayout`方法。其实这也很好理解，因为每个`ViewGroup`的排列方式不一样，所以让子类来自己实现是最好的。      
当然畜类重写`onLayout`之外，也要重写`onMeasure`。
代码如下，滑动手势处理的部分就不贴了。     
```java
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		int measureSpec = MeasureSpec.makeMeasureSpec(mScreenHeight
				* getChildCount(), MeasureSpec.getMode(heightMeasureSpec));
		super.onMeasure(widthMeasureSpec, measureSpec);
		measureChildren(widthMeasureSpec, heightMeasureSpec);
	}

	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
        // 就像猴子捞月一样，让他们一个个的从上往下排就好了
		if (changed) {
			int childCount = getChildCount();
			for (int i = 0; i < childCount; i++) {
				View child = getChildAt(i);
				if (child.getVisibility() != View.GONE) {
					child.layout(l, i * mScreenHeight, r, (i + 1)
							* mScreenHeight);
				}
			}
		}
	}
```

上面介绍了通过继承`View`以及`ViewGroup`的方式来自定义`View`，平时开发过程中有时不需要继承他俩，我们直接继承功能接近
的类进行扩展就好，例如:我想自定义一个`Meterial Design`样式的`EditText`。那我们该怎么实现呢？ 当然是继承`EditText`了，它比`EditText`多了一条底下的线，那我们给它`draw`上就可以了。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/meterial_edittext.png?raw=true)

```java
public class MetrailEditText extends EditText {
    private NinePatchDrawable mDrawable;

    public MetrailEditText(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public MetrailEditText(Context context) {
        super(context);
        init();
    }

    private void init() {
        setBackgroundResource(0);
        mDrawable = (NinePatchDrawable) getResources().getDrawable(R.drawable.edittext_meterial_bg_activated);
    }

    @Override
    protected void onDraw(final Canvas canvas) {
        super.onDraw(canvas);
        mDrawable.setBounds(-getCompoundPaddingLeft(), 0, getWidth() + getCompoundPaddingRight(), getHeight());
        mDrawable.draw(canvas);
    }
}
```

看到这里你可能会糊涂，这哪行啊？ 我们的`edittext_meterial_bg_activated`可不是普通的图，当然是`9 patch`图了。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/meterial_edittext_line.png?raw=true)

当然你可以在`onDraw()`的时候加一个自定义线的颜色`mDrawable.setColorFilter(mLineColor, PorterDuff.Mode.SRC_ATOP);`等。



参考部分:   	
- http://blog.csdn.net/cyp331203/article/details/40736027

	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! I