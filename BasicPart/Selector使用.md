Selector使用
===

**Selector**使其能够在不同的状态下更换某个View的背景图片。
```xml
<?xml version="1.0" encoding="utf-8" ?>     
<selector xmlns:android="http://schemas.android.com/apk/res/android">   
  <!-- 触摸时并且当前窗口处于交互状态 -->    
  <item android:state_pressed="true" android:state_window_focused="true" android:drawable= "@drawable/pic1" />  
  <!--  触摸时并且没有获得焦点状态 -->    
  <item android:state_pressed="true" android:state_focused="false" android:drawable="@drawable/pic2" />    
  <!--选中时的图片背景-->    
  <item android:state_selected="true" android:drawable="@drawable/pic3" />     
  <!--获得焦点时的图片背景-->    
  <item android:state_focused="true" android:drawable="@drawable/pic4" />    
  <!-- 窗口没有处于交互时的背景图片 -->    
  <item android:drawable="@drawable/pic5" />   
</selector>
```
`Selector`最终会被`Android`框架解析成`StateListDrawable`类对象。

1. StateListDrawable类介绍    
    该类定义了不同状态值下与之对应的图片资源，即我们可以利用该类保存多种状态值，多种图片资源。     
	方法：
    - `public void addState (int[] stateSet, Drawable drawable)`    
        功能： 给特定的状态集合设置drawable图片资源
    	```java
		//初始化一个空对象  
		StateListDrawable stalistDrawable = new StateListDrawable();  
		//获取对应的属性值 Android框架自带的属性 attr  
		int pressed = android.R.attr.state_pressed;  
		int window_focused = android.R.attr.state_window_focused;  
		int focused = android.R.attr.state_focused;  
		int selected = android.R.attr.state_selected;  
		  
		stalistDrawable.addState(new int []{pressed , window_focused}, getResources().getDrawable(R.drawable.pic1));  
		stalistDrawable.addState(new int []{pressed , -focused}, getResources().getDrawable(R.drawable.pic2);  
		stalistDrawable.addState(new int []{selected }, getResources().getDrawable(R.drawable.pic3);  
		stalistDrawable.addState(new int []{focused }, getResources().getDrawable(R.drawable.pic4);  
		//没有任何状态时显示的图片，我们给它设置我空集合  
		stalistDrawable.addState(new int []{}, getResources().getDrawable(R.drawable.pic5);  
		上面的“-”负号表示对应的属性值为 false
		当我们为某个View使用其作为背景色时，会根据状态进行背景图的转换。
    	```
    - public boolean isStateful ()     
        功能： 表明该状态改变了，对应的drawable图片是否会改变。
        注：在StateListDrawable类中，该方法返回为true，显然状态改变后，我们的图片会跟着改变。

2. GridView之Selector使用：  
    GridView在点击每一个条目的时候黄色的背景,很难看，那么怎么才能让其不显示这个颜色呢?就是在GridView中将listSelector这个属性指定为透明的，
	这样再点击的时候就不显示黄色了，但是这样用户不知道自己点击了没有，所以要让它在点击的时候显示一个我们自定义的颜色     
    ```xml
    <GridView
        android:listSelector="@android:color/transparent"//listSelector用于标示当前的条目被选择的时候的状态
        android:id="@+id/gv_home"
        android:verticalSpacing="10dip"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:numColumns="3" >
    </GridView>
    ``` 
    1. drawable目录新建xml文件
    	```xml
    	<?xml version="1.0" encoding="utf-8"?>
    	<selector xmlns:android="http://schemas.android.com/apk/res/android">
    		<item android:state_pressed="true"
    			  android:drawable="@color/gray" /> <!-- 点击的时候显示的背景 -->
    		<item android:state_focused="true"
    			  android:drawable="@color/gray" /> <!-- 获取焦点的时候显示的背景 -->
    		<item android:state_hovered="true"
    			  android:drawable="@drawable/button_focused" /> <!-- hovered -->
    		<item android:drawable="@android:color/transparent" /> <!-- 平常状态显示的颜色 -->
    	</selector>
    	```	
    *这里android:drawable="@color/gray"必须通过将颜色放到res下的color.xml中然后通过@color/gray这种方式指定而不能通过#000000这样直接写颜色，如果直接写颜色会报错*
    
    2. 在控件中通过背景使用这个状态选择器 		
        对每个GridView的子条目设置相应的背景为改状态选择器
        ```xml
    	<?xml version="1.0" encoding="utf-8"?>
    	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:gravity="center_horizontal"
    		android:background="@drawable/gv_item_selector"
    		android:orientation="vertical" >
    		<ImageView
    			android:id="@+id/iv_home_item_icon"
    			android:layout_width="60dip"
    			android:layout_height="60dip"
    			android:scaleType="fitXY"  //scaleType是指定图片的缩放类型， fitXY就是填充x和y轴
    			android:src="@drawable/safe" />
    		<TextView
    			android:id="@+id/tv_home_item_name"
    			android:layout_width="wrap_content"
    			android:layout_height="wrap_content"
    			android:text="手机防盗"
    			android:textSize="18sp"
    			android:textColor="#000000" />
    	</LinearLayout>
        ```
        
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
