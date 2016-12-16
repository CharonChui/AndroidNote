Scroller简介
===

在`SlidingMenu`项目中为了实现控件的滑动，需要用到`Scroller`类来实现缓慢的滑动过程，至于有人说`View`类可以直接调用`scrollTo()`方法，
这里`scrollTo()`方法也能实现移动，但是它的移动是很快一下子就移过去了，就像穿越一样，直接从现实回到了过去，而`Scroller`类能够实现过程的移动。
可以理解为一步步的走。    

1. 查看Scroller源码
    ```java
    public class Scroller  {
    	//...
    }
    ```
    发现`Scroller`类并不是`View`的子类，只是一个普通的类，这个类中封装了滚动的操作，记录了滚动的位置以及时间等。     
该类有两个重要的方法：
	- `computeScrollOffset()`:    
	    文档的说明为`Call this when you want to know the new location.`查看源码可以发现，如果在移动到指定位置后就会返回false.正在移动的过程中返回true。
	- `startScroll()`:     
	    该方法的内部实现，并没有具体的移动方法，而是设置了一些移动所需的数据，包括移动持续的时间、开始位置、结束位置等。从而我们可以知道调用`Scroller.startScroll()`方法并没有真正的移动，而是设置了一些数据。

2. `Scroller.startScoll()`是如何与`View`的移动相关联呢？在`View`的源码中：
    ```java
    /**
     * Called by a parent to request that a child update its values for mScrollX
     * and mScrollY if necessary. This will typically be done if the child is
     * animating a scroll using a {@link android.widget.Scroller Scroller}
     * object.
     */
    public void computeScroll() {
    }
    ```
    通过注释我们可以看到该方法又父类调用根据滚动的值去更新`View`，在使用`Scroller`的时候通常都要实现该方法。来达到子`View`的滚动效果。      
	继续往下跟发现在`draw()`方法中回去调用`computeScroll()`，而`draw()`方法会在父布局调用`drawChild()`的时候使用。

3. 具体关联   
    通过上面两步大体能得到`Scroller`与`View`的移动要通过`computeScroll()`来完成，但是在究竟如何进行代码实现。     
    `Scroller.startScroll()`方法被调用后会储存要滚动的起始位置、结束位置、持续时间。所以我们可以在`computeScroll()`方法中去判断一下当前是否已经滚动完成，如果没有滚动完成，
	我们就去不断的获取当前`Scroller的位置`，根据这个位置，来把相应的`View`移动到这里。
    ```java
    public void computeScroll() {
    	if (mScroller.computeScrollOffset()) {
    		//如果还没有滚动完成，我们就去让当前的View移动到指定位置去
    		mCenterView.scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
    		//移动完后，我们应该继续调用computeScoll方法去获取并且移动当前View。所以我们调用invalidate方法去请求重绘，这样父类就会调用computeScroll
    		postInvalidate();
    	}
    }
    ```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
