
ListView专题
===

1.`ListView`属性：
---

1. `fadingEdge`属性            
	`ListView`上边和下边有黑色的阴影,`android : fadingEdge = "none"`后就不会有阴影了
	
2. `scrollbars`属性，隐藏滚动条        
	`android : scrollbars = "none"`
	`setVerticalScrollBarEnabled(true);`
	
3. `fadeScrollbars`属性           
	`android : fadeScrollbars = "true"`
	设置此值为true就可以实现滚动条的自动隐藏和显示。
	
4. `fastScrollEnabled`属性           
	快速滚动滑块 
	`android : fastScrollEnabled = "true"` 
	`mListView.setFastScrollEnabled(true);`
	
5. `drawSelectorOnTop`属性    
	When set to true, the selector will be drawn over the selecteditem. Otherwise the selector is drawn behind the selected item. Thedefault value is false.
	`android:drawSelectorOnTop = "false"` 点击某条记录不放，颜色会在记录的后面，成为背景色，但是记录内容的文字是可见的

2.`ListView.setEmptyView()`没有效果
---

有时调用`setEmptyView`没有效果，这是因为我们设置的这个`EmptyView`必须和该`ListView`在同一个**布局体系中**    
如：下面这样的代码有些时候会没有效果
```java
View loadingView = View.inflate(getActivity(), R.layout.loading,  null); 
mPullLoadListView.setEmptyView(loadingView);          
mPullLoadListView.setAdapter(adapter);
```

- `Fragment`中添加下面代码就可以了。         
	```java
	View loadingView = View.inflate(getActivity(), R.layout.loading, null);
	//添加到同一布局体系中
	getActivity().addContentView(loadingView,
			  new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT ));
	mPullLoadListView.setEmptyView(loadingView);
	mPullLoadListView.setAdapter(adapter);
	```

- `Activity`中
	```java
	View empty = getLayoutInflater().inflate(R.layout.empty_list_item, null, false);
	addContentView(empty, new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
	mPullLoadListView.setEmptyView(empty);
	```
 
3.`ListView`调用`addHeaderView`后,`onItemClick`时位置不正确
---

`addHeaderView()`以及`addFooterView()`一定要在调用`setAdapter()`方法之前调用，不然会报错。
当`ListView`通过`addHeaderView`添后，在o`nItemClick`中的`position`会加上`Header`的个数，所以这时候在获取数据的时候要对位置进行处理。

下面两种方法都可以：    

1. 第一种
	```java
	public void onItemClick(AdapterView <?> parent, View v, int position, long id) {
		//parent.getAdapter().getItem(position)能得到真正位置的数据
		doSomething(parent.getAdapter().getItem(position));
	}
	```

2. 第二种
	```java
	mListView.setOnItemClickListener(new OnItemClickListener() {
		@Override
		public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
			
			int headerViewCount = mListView.getHeaderViewsCount();
			int realPos = position - mListView.getHeaderViewsCount();
			if (realPos < 0)
				return;
			......这样realPos就是真是的位置
			
		}
	});
	```

4.`ListView.addHeadrView()`添加`ViewPager`不显示的问题
---

`addHeaderView()`添加`ViewPager`后不能显示出来的问题：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical" >
    <android.support.v4.view.ViewPager
        android:id="@+id/vp_auto_circle"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
    </android.support.v4.view.ViewPager>
    <com.ifeng.padvideo.widget.IndicatorTabsView
        android:id="@+id/stv_auto_circle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" >
    </com.ifeng.padvideo.widget.IndicatorTabsView>
</LinearLayout>
```
```java
mHeaderView = View.inflate(this, R.layout.auto_circle_viewpager, null);
mAutoCircleViewPager = (ViewPager) mHeaderView.findViewById(R.id.vp_auto_circle);
//addHeaderView要在ListView的setAdapter前添加            
mListView.addHeaderView(mHeaderView);
```
**注意**ViewPager的布局中宽高不能够使用`wrap_content`可以使用`match_parent`但是上面显示不出来也是由于match_parent的问题，
如果我们将布局中的`layout_height="200dip"`,这样就能够显示出来`ViewPager`
 
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
