下拉刷新ListView
===

PullToRefreshListView
---

**原理：**        
拉刷新`ListView`无非就是对普通的`List View`添加一个`HeaderView`,然后通过对`ListView onTouchEvent`来获取当前下拉刷新的状态。然后去改变`HeaderView`的状态。    

1. 自定义`ListView`，在构造方法中去添加`HeaderView`
	通过`ListView.addHeaderView()`去添加`HeaderView`的时候，`HeaderView`会显示在屏幕的最初位置，我们需要它默认的时候是在屏幕的上方，这样默认时是不可见的，
	但是我们下拉`ListView`的时候，它就能够显示出来。这就要通过设置`HeaderView`的`padding`来实现它的隐藏。注意：View的显示最初要经过`Measure`测量宽高，
	我们在构造方法去添加的时候，该View可能并没有被测量，所以在获取`HeaderView`高度的时候会为0，这时候我们要手动的去测量一下`HeaderView`。
	```java
	/**
	* 当前状态
	*/
	private State mState = State.ORIGNAL;

	public PullToRefreshListView(Context context, AttributeSet attrs,
			int defStyle) {
		super(context, attrs, defStyle);
		initView(context);
	}

	public PullToRefreshListView(Context context, AttributeSet attrs) {
		super(context, attrs);
		initView(context);
	}

	public PullToRefreshListView(Context context) {
		super(context);
		initView(context);
	}

	private void initView(Context context) {
		LayoutInflater inflater = (LayoutInflater) context
				.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

		mHeader = inflater.inflate(R.layout.pull_to_refresh_header, null);
		iv_arrow = (ImageView) mHeader.findViewById(R.id.iv_arrow);
		pb_refresh = (ProgressBar) mHeader.findViewById(R.id.pb_refresh);
		tv_title = (TextView) mHeader.findViewById(R.id.tv_title);
		tv_time = (TextView) mHeader.findViewById(R.id.tv_time);
		
		measureHeaderView(mHeader);
		
		mHeaderHeight = mHeader.getMeasuredHeight();
		// To make header view above the window, so use -mHeaderHeight.
		mHeader.setPadding(0, -mHeaderHeight, 0, 0);

		mHeader.invalidate();

		addHeaderView(mHeader);
	}
		
	/**
	 * 下拉刷新所有的状态
	 */
	public enum State {
		ORIGNAL, PULL_TO_REFRESH, REFRESHING, RELEASE_TO_REFRESH;
	}
	```

2. 重写onTouchEvent，来监听手指的下拉
	```java
	public boolean onTouchEvent(MotionEvent ev) {
		int y = (int) ev.getRawY();
		int action = ev.getAction();
		switch (action) {
		case MotionEvent.ACTION_DOWN:
			//记录手指点下的位置
			downPositionY = y;
			break;
		case MotionEvent.ACTION_MOVE:
			//手指当前滑动的位置
			currentPositionY = y;
			//手指移动的距离，由于HeaderView高度固定，但是手指下拉的高度可以最大为屏幕的高度，如手指下拉屏幕高度时，HeaderView会很难看，
			// 所以我们让下拉的距离进行一个缩放。
			pullDistance = (currentPositionY - downPositionY) / RATIO;

			if (mState == State.REFRESHING) {
				break;
			} else if (mState == State.ORIGNAL && pullDistance > 0) {
				//如果现在处理起始的状态，并且距离大于0，就说明是下拉了，这时候状态需要变为下拉刷新的状态
				mState = State.PULL_TO_REFRESH;
				changeState();
			} else if (mState == State.PULL_TO_REFRESH
					&& pullDistance > mHeaderHeight) {
				//当时为下拉刷新的状态，但是下拉的距离大于HeaderView的高度。这时状态要变为松手即可刷新
				mState = State.RELEASE_TO_REFRESH;
				changeState();
			} else if (mState == State.RELEASE_TO_REFRESH) {
			//释放刷新时有三种情况，一是我继续下啦，这时候不用管，因为继续下拉还是释放刷新。二是我手指往上移动，此时HeaderView不完全可见，
			// 这时候状态要改变为下拉刷新了。三是我手指上移的很厉害，导致HeaderView完全不可见了，这是状态要改变为起始状态。
				if (pullDistance < 0) {
					// 如果当时状态为松手刷新，但是这时候我并没有松手，而是直接将手指往上移动，移动回手指最先的位置，这时候状态要变为起始状态。
					mState = State.ORIGNAL;
					changeState();
				} else if (pullDistance < mHeaderHeight) {
					//手指上移，但是并没有移动到HeaderView完全不可见，这时候要将状态改变为下拉刷新
					mState = State.PULL_TO_REFRESH;
					isBack = true;
					changeState();
				}

			}
			
			//在移动的过程中不断的去改版Padding的值，控制其显示的大小
			if (mState != State.REFRESHING) {
				mHeader.setPadding(0, (int) (pullDistance - mHeaderHeight), 0,
						0);
			}

			break;
		case MotionEvent.ACTION_UP:
			if (mState == State.REFRESHING) {
				//如果当前已经是正在刷新中了，再去下拉就不要处理了
				break;
			} else if (mState == State.PULL_TO_REFRESH) {
				//显现下拉刷新时，松手了，这时候要将其改为起始状态
				mState = State.ORIGNAL;
			} else if (mState == State.RELEASE_TO_REFRESH) {
				//松手刷新时松手了，这时候状态要变为正在刷新中。
				mState = State.REFRESHING;
			} else {
				break;
			}
			changeState();
			break;

		default:
			break;
		}

		return super.onTouchEvent(ev);
	}
	```

3. 上面的写法仍有些问题。就是我们在`onTouchEvent`中`Move`里面对移动的距离进行了判断，但是`ListView`本身就是一个可以上下滑动的组件，
	如果我们直接这样判断，那`ListView`本上上下滑动的功能就被我们给抹去了。
	```java
	@Override
	public boolean onTouchEvent(MotionEvent ev) {
		int y = (int) ev.getRawY();
		int action = ev.getAction();
		switch (action) {
		case MotionEvent.ACTION_DOWN:
			downPositionY = y;
			break;
		case MotionEvent.ACTION_MOVE:
			//一定要加上这句话，来判断当前是否可以下拉刷新
			if (!isCanPullToRefresh) {
				break;
			}
			.....
			break;
	```
	而对于`isCanPullToRefresh`的判断是通过`ListView.setOnScrollListener`去进行判断当前第一个可见条目是不是`ListView`的第一个条目，
	只有第一个条目在最顶端位置的时候才可以进行下拉刷新。
	```java
	super.setOnScrollListener(new OnScrollListener() {

		@Override
		public void onScrollStateChanged(AbsListView view, int scrollState) {
		}

		@Override
		public void onScroll(AbsListView view, int firstVisibleItem,
				int visibleItemCount, int totalItemCount) {
			
			if (firstVisibleItem == 0) {
				isCanPullToRefresh = true;
			} else {
				isCanPullToRefresh = false;
			}
		}
	});
	```
4. 提供刷新接口

	```java
	public interface OnRefreshListener {
		abstract void onRefresh();
	}
	```
	
5. ChangeState方法

	```java
	/**
	 * Change the state of header view when ListView in different state.
	 */
	private void changeState() {
		//在该方法中对mState进行判断，根据不同状态作出处理，并且去调用刷新方法
	}

	/**
	 *数据刷新完成后需要调用此方法去恢复装填
	 */
	@SuppressWarnings("deprecation")
	public void onRefreshComplete() {
		mState = State.ORIGNAL;
		changeState();
		tv_time.setText(getResources().getString(R.string.update_time)
				+ new Date().toLocaleString());
	}
	```

LoadMoreListView
---
**原理：**    
滑动到底部自动加载更多的`ListView`，无非就是通过对其滑动过程进行监听，一旦滑动到底部的时候我们就去加载新的数据。通过对`ListView`添加`FooterView`，
然后在不同的状态控制它的显示与隐藏。     

1.  自定义ListView的子类，在构造方法中去添加FooterView.  
	```java
	public class LoadMoreListView extends ListView {

	public LoadMoreListView(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		init(context);
	}

	public LoadMoreListView(Context context, AttributeSet attrs) {
		super(context, attrs);
		init(context);
	}

	public LoadMoreListView(Context context) {
		super(context);
		init(context);
	}

	private void init(Context context) {
		mFooterView = View.inflate(context, R.layout.load_more_footer, null);
		addFooterView(mFooterView);
		hideFooterView();
	}
	```	

2. 监听滑动状态，通过`setOnScrollListener`即可实现对状态的监听。
	```java
	private void init(Context context) {
		mFooterView = View.inflate(context, R.layout.load_more_footer, null);
		addFooterView(mFooterView);
		hideFooterView();
		//为了防止在使用时调用setOnScrollListener会覆盖此时设置的Listener，我们在此使用super.setOnScrollListenr()
		super.setOnScrollListener(superOnScrollListener);
	}
	```

3. 在`ScrollListener`中去判断当前滑动的状态。从而依据不同的状态去控制`FooterView`的显示与隐藏。
	```java
	private OnScrollListener superOnScrollListener = new OnScrollListener() {
		
		@Override
		public void onScrollStateChanged(AbsListView view, int scrollState) {
			mCurrentScrollState = scrollState;
		}

		@Override
		public void onScroll(AbsListView view, int firstVisibleItem,
				int visibleItemCount, int totalItemCount) {
			if (visibleItemCount == totalItemCount) {
				//此时说明当前ListView所有的条目比较少，不足一屏
				hideFooterView();
			} else if (!mIsLoading
					&& (firstVisibleItem + visibleItemCount >= totalItemCount)
					&& mCurrentScrollState != SCROLL_STATE_IDLE) {
				//当第一个可见的条目位置加上当前也所有可见的条目数 等于 ListView当前总的条目数时，就说明已经滑动到了底部，这时候就要去显示FooterView。
				showFooterView();
				mIsLoading = true;
				if (mOnLoadMoreListener != null) {
					mOnLoadMoreListener.onLoadMore();
				}
			}
		}
	};
	```

4. 提供自动加载更多的接口。
	```java
	public interface OnLoadMoreListener {
		/**
		 * Load more data.
		 */
		void onLoadMore();
	}
	```

在使用的时候，与ListView使用方法相同，只需调用`setOnLoadMoreListener`对其设置`OnLoadMoreListener`即可，然后在数据加载完成后调用`onLoadComplete`方法去恢复状态。


PullAndLoadMoreListView
---
将下拉刷新和自动加载更多进行整合。就不多说了

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 