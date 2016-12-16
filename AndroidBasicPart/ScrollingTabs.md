ScrollingTabs
===

自定义ScrollingTabs结合ViewPager实现指引的效果。    
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/ScrollingTabs.png?raw=true)        
       
**原理:**         
由于`ScrollingTabs`即可以点击又可以实现左右滑动，首先想到的就是继承`HorizontalScrollView`来实现滑动，至于点击的实现需要通过对`View`
设置点击。          
通过对`ViewPager`设置`OnPageChangeListener`来监听页面变化，从而实现对`ScrollingTabs`的改变，而在每个`Tab`上设置
点击事件，当点击的时候就去设置`ViewPager`的当前页面

1. 继承HorizontalScrollView，并且添加一个水平方向的线性布局，作为Tab的父布局
	```java
	public class ScrollingTabs extends HorizontalScrollView {

		private LinearLayout mContainer;

		public ScrollingTabs(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			init(context);
		}

		public ScrollingTabs(Context context, AttributeSet attrs) {
			super(context, attrs);
			init(context);
		}

		public ScrollingTabs(Context context) {
			super(context);
			init(context);
		}

		private void init(Context context) {
			this.setHorizontalScrollBarEnabled(false);
			this.setHorizontalFadingEdgeEnabled(false);

			mContainer = new LinearLayout(context);
			LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(
					android.view.ViewGroup.LayoutParams.MATCH_PARENT,
					android.view.ViewGroup.LayoutParams.MATCH_PARENT);
			mContainer.setLayoutParams(params);
			mContainer.setOrientation(LinearLayout.HORIZONTAL);

			addView(mContainer);
		}
	}
	```

2. 提供接口供调用者设置每个Tab的视图。
	```java
	public interface TabAdapter {
		/**
		 * 每个Tab的视图
		 */
		public View getView(int position);
		/**
		 * Tab之间的分割线
		 */
		public View getSeparator();
	}
	```
	
3. 暴露方法，初始化Tab。
	```java
	public void setTabAdapter(TabAdapter adapter) {
		this.mTabAdapter = adapter;
		initTabView();
	}

	public void setViewPager(ViewPager pager) {
		this.mViewPager = pager;
		mViewPager.setOnPageChangeListener(this);
		initTabView();
	}

	/**
	 * 必须等到ViewPager和TabAdapter都设置完成后才可以调用
	 */
	private void initTabView() {
		if (mViewPager != null && mTabAdapter != null) {
			//清空父布局，保险起见
			mContainer.removeAllViews();
			//根据ViewPager的页数去设置Tab
			for (int i = 0; i < mViewPager.getAdapter().getCount(); i++) {
				final View tab = mTabAdapter.getView(i);
				tab.setTag(i);

				mContainer.addView(tab);

				// Segmentation view
				if (mTabAdapter.getSeparator() != null
						&& i != mViewPager.getAdapter().getCount() - 1) {
					//Tabs之间使用分割线
					isUseSeperator = true;
					mContainer.addView(mTabAdapter.getSeparator());
				}

				// 对每个Tab设置点击事件
				tab.setOnClickListener(new OnClickListener() {

					@Override
					public void onClick(View v) {
						int index = (Integer) tab.getTag();
						if (mTabClickListener != null) {
							//暴露接口
							mTabClickListener.onClick(index);
						} else {
							if (mViewPager.getCurrentItem() == index) {
								//如果当前ViewPager已经显示到了该Tab也，就直接让其选中
								selectTab(index);
							} else {
								//当前ViewPager并没有显示该Tab页，要让ViewPager去显示相应的Tab页
								mViewPager.setCurrentItem(index, true);
							}
						}
					}
				});

			}

			// 初始化时核对一下Tab
			selectTab(mViewPager.getCurrentItem());
		}
	}
	```
	
4. selectTab的实现，选中相应的Tab，并且实现滑动到屏幕中间位置
	```java
	private void selectTab(int position) {
		if (!isUseSeperator) {
			//没有分割线
			for (int i = 0; i < mContainer.getChildCount(); i++) {
				View tab = mContainer.getChildAt(i);
				tab.setSelected(i == position);
			}
		} else {
			//有分割线
			for (int i = 0, pos = 0; i < mContainer.getChildCount(); i += 2, pos++) {
				View tab = mContainer.getChildAt(i);
				tab.setSelected(pos == position);
			}
		}
		//得到当前的Tab
		View selectedView = null;
		if (!isUseSeperator) {
			selectedView = mContainer.getChildAt(position);
		} else {
			selectedView = mContainer.getChildAt(position * 2);
		}

		int tabWidth = selectedView.getMeasuredWidth();
		int tabLeft = selectedView.getLeft();

		//距离左边屏幕的位置加上该Tab宽度的一半正好是该Tab中心点的位置。我们需要让该Tab的中心点移动到屏幕的中心点。
		int distance = (tabLeft + tabWidth / 2) - mWindowWidth / 2;
		//移动
		smoothScrollTo(distance, this.getScrollY());
	}
	```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
