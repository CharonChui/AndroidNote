SlidingMenu
===

先看一下图片      
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/slidingmenu_1.png?raw=true)    
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/slidingmenu_2.png?raw=true)   
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/slidingmenu_3.png?raw=true)   


原理
---

`SlidingMenu`无非就是一个包含三个`View`的控件，**左边View**、**中间View(默认时全屏)**、**右边View**，默认的情况下中间`View`会把两边的`View`覆盖住，
在手指滑动的时候，会根据手指的滑动方向以及滑动距离去移动中间的那个`View`，从而能让两边`View`完全可见。    
在定义该View的时候，首先会想到继承`RelativeLayout`，能简单的实现这种左、中、右三个View的布局。

1. 继承RelativeLayout
	```java
	public class SlidingMenu extends RelativeLayout {
		
		public SlidingMenu(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		init(context);
		}
	
		public SlidingMenu(Context context, AttributeSet attrs) {
			super(context, attrs);
			init(context);
		}
	
		public SlidingMenu(Context context) {
			super(context);
			init(context);
		}
	
		private void init(Context context) {
			mContext = context;
			mScroller = new Scroller(context);
			mWindowWidth = getWindowWidth(context);
		}
	}
	```

2. 具体的三个View需要暴露给外界调用，所以我们要提供一个setView()的方法。
	```java
	public void setView(View leftView, View rightView, View centerView,
			int leftViewWidth, int rightViewWidth) {
		//添加左边View
		RelativeLayout.LayoutParams leftParams = new LayoutParams(
				(int) convertDpToPixel(leftViewWidth, mContext),
				LayoutParams.MATCH_PARENT);
		leftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT);
		addView(leftView, leftParams);
		
		//右边的View
		RelativeLayout.LayoutParams rightParams = new LayoutParams(
				(int) convertDpToPixel(rightViewWidth, mContext),
				LayoutParams.MATCH_PARENT);
		rightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT);
		addView(rightView, rightParams);
	
		//添加中间的View
		RelativeLayout.LayoutParams centerParams = new LayoutParams(
				LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
		addView(centerView, centerParams);
	
		mLeftView = leftView;
		mRightView = rightView;
		mCenterView = centerView;
	}
	```
	外界使用`SlidingMenu`类的时候需要首先调用该方法去设置相应的View，一旦调用该方法后，我们就将布局设置完了，下一步就是对`touch`事件进行处理，然后去移动中间的View。

3. 处理Touch事件
	在手指按下的时候，我们去控制两边View的显示与隐藏
	```java
	public boolean onInterceptTouchEvent(MotionEvent ev) {
		int x = (int) ev.getRawX();
		int y = (int) ev.getRawY();
	
		int action = ev.getAction();
	
		switch (action) {
		case MotionEvent.ACTION_DOWN:
			mLastPostionX = x;
			mLastPostionY = y;
			//通过变量记录当前可以显示左边的View还是可以显示右边的View
			if (mCanLeftViewShow) {
				//如果当前，中间的View往右滑，那么这时候左边的View就要能显示了
				mLeftView.setVisibility(View.VISIBLE);
				mRightView.setVisibility(View.GONE);
			} else if (mCanRightViewShow) {
				mLeftView.setVisibility(View.GONE);
				mRightView.setVisibility(View.VISIBLE);
			}
	
			break;
		case MotionEvent.ACTION_MOVE:
	
			break;
		case MotionEvent.ACTION_UP:
	
			break;
	
		default:
			break;
		}
	
		return false;
	}
	```
	在`onTouch()`中，我们去获取手指移动的距离
	```java
		public boolean onTouchEvent(MotionEvent event) {
		int x = (int) event.getRawX();
		int y = (int) event.getRawY();
	
		int action = event.getAction();
		switch (action) {
		case MotionEvent.ACTION_DOWN:
			mLastPostionX = x;
			mLastPostionY = y;
	
			if (!mScroller.isFinished()) {
				mScroller.abortAnimation();
			}
	
			break;
		case MotionEvent.ACTION_MOVE:
			int distance = x - mLastPostionX;
			int targetPositon = mCenterView.getScrollX() - distance;
			mLastPostionX = x;
	
			if (mCanLeftViewShow) {
				if (targetPositon > 0) {
					targetPositon = 0;
				}
	
				if (targetPositon < -mLeftViewWidth) {
					targetPositon = -mLeftViewWidth;
				}
			}
	
			if (mCanRightViewShow) {
				if (targetPositon < 0) {
					targetPositon = 0;
				}
	
				if (targetPositon > mRightViewWidth) {
					targetPositon = mRightViewWidth;
				}
			}
	
			mClicked = false;
			//让中间的View随着手指的移动而移动
			mCenterView.scrollTo(targetPositon, 0);
	
			break;
		case MotionEvent.ACTION_UP:
			//你手指移动后抬起的时候需要注意，如果现在左边的View已经超过一半可见了，这时候就算你抬起手指了，SlidingMenu也要滑动到右边让左边View完全可见。当然还有就是你滑动的飞快，然后突然抬起了手指，这时候就要进行速率的计算了，我们先不说速率
			int dx = 0;
			if (mCanLeftViewShow) {
				if (mCenterView.getScrollX() <= -mLeftViewWidth / 2) {
					//已经超过左边View的一般了，应该让中间View继续移动，移动到左边View完全可见
					dx = -mLeftViewWidth - mCenterView.getScrollX();
				} else {
					// 滚回原来的位置
					dx = -mCenterView.getScrollX();
					resumeLeftViewClickState();
				}
	
			} else if (mCanRightViewShow) {
				if (mCenterView.getScrollX() >= mRightViewWidth / 2) {
					dx = mRightViewWidth - mCenterView.getScrollX();
				} else {
					dx = -mCenterView.getScrollX();
					resumeRightViewClickState();
				}
			}
			//手指抬起后，要让中间View有过程的滑过去，所以要用到Scroller类
			smoothScrollTo(dx);
			break;
	
		default:
			break;
		}
	
		return true;
	}
	```

	`Scroller`的实现       
	```java
	private void smoothScrollTo(int distance) {
		mScroller.startScroll(mCenterView.getScrollX(), 0, distance, 0,
				sDuration);
		invalidate();
	}

	@Override
	public void computeScroll() {
		if (mScroller.computeScrollOffset()) {
			mCenterView.scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
			postInvalidate();
		}
	}
	```

4. 到这里SlidingMenu的大体实现已经完成了        
	剩下的就是对速率的计算，已经添加显示左边与显示右边的View的按钮。当左边View完全显示的时候，点击中间View可见部分时需要让中间View全屏。
	至于这些细节的东西就不再仔细说了，大家自己看源码吧。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
