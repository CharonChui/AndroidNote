View绘制过程详解
===

界面窗口的根布局是`DecorView`，该类继承自`FrameLayout`.说到`View`绘制，想到的就是从这里入手，而`FrameLayout`继承自`ViewGroup`。感觉绘制肯定会在`ViewGroup`或者`View`中，
但是木有找到。发现`ViewGroup`实现`ViewParent`接口，而`ViewParent`有一个实现类是`ViewRootImpl`， `ViewGruop`中会使用`ViewRootImpl`...
```java
/**
 * The top of a view hierarchy, implementing the needed protocol between View
 * and the WindowManager.  This is for the most part an internal implementation
 * detail of {@link WindowManagerGlobal}.
 *
 * {@hide}
 */
@SuppressWarnings({"EmptyCatchBlock", "PointlessBooleanExpression"})
public final class ViewRootImpl implements ViewParent,
        View.AttachInfo.Callbacks, HardwareRenderer.HardwareDrawCallbacks {
		
		}
```

`View`的绘制过程从`ViewRootImpl.performTraversals()`方法开始。
首先先说明一下，这部分代码比较多，逻辑也比较麻烦，很容易弄晕，如果感觉看起来费劲，就跳过这一块，直接到下面的Measure、Layout、Draw部分开始看。
我也没有全部弄清楚，我只是把里面的步骤标注了下。
```java
private void performTraversals() {
	// ... 此处省略源代码N行

	// 是否需要Measure
	if (!mStopped) {
		boolean focusChangedDueToTouchMode = ensureTouchModeLocally(
				(relayoutResult&WindowManagerGlobal.RELAYOUT_RES_IN_TOUCH_MODE) != 0);
		if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth()
				|| mHeight != host.getMeasuredHeight() || contentInsetsChanged) {
			// 这里是获取widthMeasureSpec,这俩参数不是一般的尺寸数值，而是将模式和尺寸组合在一起的数值.
			// getRootMeasureSpec方法内部会使用MeasureSpec.makeMeasureSpec()方法来组装一个MeasureSpec，
			// 当lp.width参数等于MATCH_PARENT的时候，MeasureSpec的specMode就等于EXACTLY，当lp.width等于WRAP_CONTENT的时候，MeasureSpec的specMode就等于AT_MOST。
			// 并且MATCH_PARENT和WRAP_CONTENT时的specSize都是等于windowSize的，也就意味着根视图总是会充满全屏的。
			int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
			int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);

			if (DEBUG_LAYOUT) Log.v(TAG, "Ooops, something changed!  mWidth="
					+ mWidth + " measuredWidth=" + host.getMeasuredWidth()
					+ " mHeight=" + mHeight
					+ " measuredHeight=" + host.getMeasuredHeight()
					+ " coveredInsetsChanged=" + contentInsetsChanged);
			
			// 调用PerformMeasure方法。
			 // Ask host how big it wants to be
			performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

			// Implementation of weights from WindowManager.LayoutParams
			// We just grow the dimensions as needed and re-measure if
			// needs be
			int width = host.getMeasuredWidth();
			int height = host.getMeasuredHeight();
			boolean measureAgain = false;

			if (lp.horizontalWeight > 0.0f) {
				width += (int) ((mWidth - width) * lp.horizontalWeight);
				childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,
						MeasureSpec.EXACTLY);
				measureAgain = true;
			}
			if (lp.verticalWeight > 0.0f) {
				height += (int) ((mHeight - height) * lp.verticalWeight);
				childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,
						MeasureSpec.EXACTLY);
				measureAgain = true;
			}

			if (measureAgain) {
				if (DEBUG_LAYOUT) Log.v(TAG,
						"And hey let's measure once more: width=" + width
						+ " height=" + height);
				performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
			}

			layoutRequested = true;
		}
	}

	final boolean didLayout = layoutRequested && !mStopped;
	boolean triggerGlobalLayoutListener = didLayout
			|| mAttachInfo.mRecomputeGlobalAttributes;
	// 是否需要Layout
	if (didLayout) {
		// 调用performLayout方法。
		performLayout(lp, desiredWindowWidth, desiredWindowHeight);

		// By this point all views have been sized and positioned
		// We can compute the transparent area

		if ((host.mPrivateFlags & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) != 0) {
			// start out transparent
			// TODO: AVOID THAT CALL BY CACHING THE RESULT?
			host.getLocationInWindow(mTmpLocation);
			mTransparentRegion.set(mTmpLocation[0], mTmpLocation[1],
					mTmpLocation[0] + host.mRight - host.mLeft,
					mTmpLocation[1] + host.mBottom - host.mTop);

			host.gatherTransparentRegion(mTransparentRegion);
			if (mTranslator != null) {
				mTranslator.translateRegionInWindowToScreen(mTransparentRegion);
			}

			if (!mTransparentRegion.equals(mPreviousTransparentRegion)) {
				mPreviousTransparentRegion.set(mTransparentRegion);
				mFullRedrawNeeded = true;
				// reconfigure window manager
				try {
					mWindowSession.setTransparentRegion(mWindow, mTransparentRegion);
				} catch (RemoteException e) {
				}
			}
		}

		if (DBG) {
			System.out.println("======================================");
			System.out.println("performTraversals -- after setFrame");
			host.debug();
		}
	}

	// 是否需要Draw
	if (!cancelDraw && !newSurface) {
		if (!skipDraw || mReportNextDraw) {
			if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
				for (int i = 0; i < mPendingTransitions.size(); ++i) {
					mPendingTransitions.get(i).startChangingAnimations();
				}
				mPendingTransitions.clear();
			}
			// 调用performDraw方法
			performDraw();
		}
	} else {
		if (viewVisibility == View.VISIBLE) {
			// Try again
			scheduleTraversals();
		} else if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
			for (int i = 0; i < mPendingTransitions.size(); ++i) {
				mPendingTransitions.get(i).endChangingAnimations();
			}
			mPendingTransitions.clear();
		}
	}

	mIsInTraversal = false;
}
```

从上面源码可以看出,`performTraversals()`方法中会依次做三件事：
- `performMeasure()`, 内部是` mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);`测量`View`大小。这里顺便提一下，这个`mView`是什么？它就是`Window`最顶成的`View(DecorView)`,它是`FrameLayout`的子类。
- `performLayout()`, 内部是`mView.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());`视图布局，确定`View`位置。
- `performDraw()`, 内部是`draw(fullRedrawNeeded);` 绘制界面。

至此`View`绘制的三个过程已经展现：

`Measure`
===

`performMeasure`方法如下：
```java
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
	Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
	try {
		mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
	} finally {
		Trace.traceEnd(Trace.TRACE_TAG_VIEW);
	}
}
```

在`performMeasure()`方法中会调用`View.measure()`方法， 源码如下：
```java
/**
 * <p>
 * This is called to find out how big a view should be. The parent
 * supplies constraint information in the width and height parameters.
 * </p>
 *
 * <p>
 * The actual measurement work of a view is performed in
 * {@link #onMeasure(int, int)}, called by this method. Therefore, only
 * {@link #onMeasure(int, int)} can and must be overridden by subclasses.
 * </p>
 *
 *
 * @param widthMeasureSpec Horizontal space requirements as imposed by the
 *        parent
 * @param heightMeasureSpec Vertical space requirements as imposed by the
 *        parent
 *
 * @see #onMeasure(int, int)
 */
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
	boolean optical = isLayoutModeOptical(this);
	if (optical != isLayoutModeOptical(mParent)) {
		Insets insets = getOpticalInsets();
		int oWidth  = insets.left + insets.right;
		int oHeight = insets.top  + insets.bottom;
		widthMeasureSpec  = MeasureSpec.adjust(widthMeasureSpec,  optical ? -oWidth  : oWidth);
		heightMeasureSpec = MeasureSpec.adjust(heightMeasureSpec, optical ? -oHeight : oHeight);
	}

	// Suppress sign extension for the low bytes
	long key = (long) widthMeasureSpec << 32 | (long) heightMeasureSpec & 0xffffffffL;
	if (mMeasureCache == null) mMeasureCache = new LongSparseLongArray(2);

	if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
			widthMeasureSpec != mOldWidthMeasureSpec ||
			heightMeasureSpec != mOldHeightMeasureSpec) {

		// first clears the measured dimension flag
		mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

		resolveRtlPropertiesIfNeeded();

		int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
				mMeasureCache.indexOfKey(key);
		if (cacheIndex < 0 || sIgnoreMeasureCache) {
			// 调用onMeasure方法
			// measure ourselves, this should set the measured dimension flag back
			onMeasure(widthMeasureSpec, heightMeasureSpec);
			mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
		} else {
			long value = mMeasureCache.valueAt(cacheIndex);
			// Casting a long to int drops the high 32 bits, no mask needed
			setMeasuredDimensionRaw((int) (value >> 32), (int) value);
			mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
		}

		// flag not set, setMeasuredDimension() was not invoked, we raise
		// an exception to warn the developer
		if ((mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET) != PFLAG_MEASURED_DIMENSION_SET) {
			// 重写onMeausre方法的时，必须调用setMeasuredDimension或者super.onMeasure方法，不然就会走到这里报错。
			// setMeasuredDimension中回去改变mPrivateFlags的值
			throw new IllegalStateException("onMeasure() did not set the"
					+ " measured dimension by calling"
					+ " setMeasuredDimension()");
		}

		mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
	}

	mOldWidthMeasureSpec = widthMeasureSpec;
	mOldHeightMeasureSpec = heightMeasureSpec;

	mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
			(long) mMeasuredHeight & 0xffffffffL); // suppress sign extension
}
```

在`measure`方法中会调用`onMeasure`方法。`ViewGroup`的子类会重写该方法来进行测量大小，因为`mView`是`DecorView`，
而`DecorView`是`FrameLayout`的子类。所以我们看一下`FrameLayout.onMeasure`方法：
`FrameLayout.onMeasure`源码如下：
```java
/**
 * {@inheritDoc}
 */
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	int count = getChildCount();

	final boolean measureMatchParentChildren =
			MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
			MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
	mMatchParentChildren.clear();

	int maxHeight = 0;
	int maxWidth = 0;
	int childState = 0;

	for (int i = 0; i < count; i++) {
		final View child = getChildAt(i);
		if (mMeasureAllChildren || child.getVisibility() != GONE) {
			// 调用该方法去测量每个子View
			measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
			final LayoutParams lp = (LayoutParams) child.getLayoutParams();
			maxWidth = Math.max(maxWidth,
					child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);
			maxHeight = Math.max(maxHeight,
					child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);
			childState = combineMeasuredStates(childState, child.getMeasuredState());
			if (measureMatchParentChildren) {
				if (lp.width == LayoutParams.MATCH_PARENT ||
						lp.height == LayoutParams.MATCH_PARENT) {
					mMatchParentChildren.add(child);
				}
			}
		}
	}

	// Account for padding too
	maxWidth += getPaddingLeftWithForeground() + getPaddingRightWithForeground();
	maxHeight += getPaddingTopWithForeground() + getPaddingBottomWithForeground();

	// Check against our minimum height and width
	maxHeight = Math.max(maxHeight, getSuggestedMinimumHeight());
	maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());

	// Check against our foreground's minimum height and width
	final Drawable drawable = getForeground();
	if (drawable != null) {
		maxHeight = Math.max(maxHeight, drawable.getMinimumHeight());
		maxWidth = Math.max(maxWidth, drawable.getMinimumWidth());
	}

	setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
			resolveSizeAndState(maxHeight, heightMeasureSpec,
					childState << MEASURED_HEIGHT_STATE_SHIFT));

	count = mMatchParentChildren.size();
	if (count > 1) {
		for (int i = 0; i < count; i++) {
			final View child = mMatchParentChildren.get(i);

			final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
			int childWidthMeasureSpec;
			int childHeightMeasureSpec;
			
			if (lp.width == LayoutParams.MATCH_PARENT) {
				childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(getMeasuredWidth() -
						getPaddingLeftWithForeground() - getPaddingRightWithForeground() -
						lp.leftMargin - lp.rightMargin,
						MeasureSpec.EXACTLY);
			} else {
				childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,
						getPaddingLeftWithForeground() + getPaddingRightWithForeground() +
						lp.leftMargin + lp.rightMargin,
						lp.width);
			}
			
			if (lp.height == LayoutParams.MATCH_PARENT) {
				childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(getMeasuredHeight() -
						getPaddingTopWithForeground() - getPaddingBottomWithForeground() -
						lp.topMargin - lp.bottomMargin,
						MeasureSpec.EXACTLY);
			} else {
				childHeightMeasureSpec = getChildMeasureSpec(heightMeasureSpec,
						getPaddingTopWithForeground() + getPaddingBottomWithForeground() +
						lp.topMargin + lp.bottomMargin,
						lp.height);
			}

			child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
		}
	}
}
```

我们看到内部会调用`measureChildWithMargins()`方法,该方法源码如下：
```java
/**
 * Ask one of the children of this view to measure itself, taking into
 * account both the MeasureSpec requirements for this view and its padding
 * and margins. The child must have MarginLayoutParams The heavy lifting is
 * done in getChildMeasureSpec.
 *
 * @param child The child to measure
 * @param parentWidthMeasureSpec The width requirements for this view
 * @param widthUsed Extra space that has been used up by the parent
 *        horizontally (possibly by other children of the parent)
 * @param parentHeightMeasureSpec The height requirements for this view
 * @param heightUsed Extra space that has been used up by the parent
 *        vertically (possibly by other children of the parent)
 */
protected void measureChildWithMargins(View child,
		int parentWidthMeasureSpec, int widthUsed,
		int parentHeightMeasureSpec, int heightUsed) {
	final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

	final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
			mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
					+ widthUsed, lp.width);
	final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
			mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
					+ heightUsed, lp.height);

	child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
里面就是对该子`View`调用了`measure`方法，我们假设这个`View`已经不是`ViewGroup`了，就会又和上面一样，又调用`onMeasure`方法，
下面我们直接看一下`View.onMeasure()`方法：
`View.onMeasure()`方法的源码如下：
```java
/**
 * <p>
 * Measure the view and its content to determine the measured width and the
 * measured height. This method is invoked by {@link #measure(int, int)} and
 * should be overriden by subclasses to provide accurate and efficient
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
	// 如果不重写onMeasure方法，默认会调用getDefaultSize获取大小，下面会说getDefaultSize这个方法。
	setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
			getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

`setMeasuredDimension()`方法如下： 
```java
/**
 * <p>This method must be called by {@link #onMeasure(int, int)} to store the
 * measured width and measured height. Failing to do so will trigger an
 * exception at measurement time.</p>
 *
 * @param measuredWidth The measured width of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 * @param measuredHeight The measured height of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 */
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
	boolean optical = isLayoutModeOptical(this);
	if (optical != isLayoutModeOptical(mParent)) {
		Insets insets = getOpticalInsets();
		int opticalWidth  = insets.left + insets.right;
		int opticalHeight = insets.top  + insets.bottom;

		measuredWidth  += optical ? opticalWidth  : -opticalWidth;
		measuredHeight += optical ? opticalHeight : -opticalHeight;
	}
	setMeasuredDimensionRaw(measuredWidth, measuredHeight);
}
```
`setMeasuredDimensionRaw()`方法如下： 
```java
/**
 * Sets the measured dimension without extra processing for things like optical bounds.
 * Useful for reapplying consistent values that have already been cooked with adjustments
 * for optical bounds, etc. such as those from the measurement cache.
 *
 * @param measuredWidth The measured width of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 * @param measuredHeight The measured height of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 */
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
	// 赋值给mMeasuredWidth，getMeasuredWidth就会调用该值。
	mMeasuredWidth = measuredWidth;
	mMeasuredHeight = measuredHeight;

	// 这就是重写onMeasure方法时如果不调用setMeasuredDimension方法时为什么会报错的原因。
	mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```

我们接着看一下上面用到的`getDefaultSize()`方法，源码如下：
```java
/**
 * Utility to return a default size. Uses the supplied size if the
 * MeasureSpec imposed no constraints. Will get larger if allowed
 * by the MeasureSpec.
 *
 * @param size Default size for this view
 * @param measureSpec Constraints imposed by the parent
 * @return The size this view should be.
 */
public static int getDefaultSize(int size, int measureSpec) {
	int result = size;
	// measureSpec值用于获取宽度(高度)的规格和大小，解析出对应的size和mode
	int specMode = MeasureSpec.getMode(measureSpec);
	int specSize = MeasureSpec.getSize(measureSpec);

	switch (specMode) {
	case MeasureSpec.UNSPECIFIED:
		result = size;
		break;
	case MeasureSpec.AT_MOST:
	case MeasureSpec.EXACTLY:
		result = specSize;
		break;
	}
	return result;
}
```
`getDefaultSize`方法又会使用到`MeasureSpec`类，文档中对`MeasureSpec`是这样介绍的`A MeasureSpec is comprised of a size and a mode. There are three possible modes:`
- MeasureSpec.EXACTLY The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be. 
    理解成MATCH_PARENT或者在布局中指定了宽高值，如layout:width='50dp'
- MeasureSpec.AT_MOST The child can be as large as it wants up to the specified size.理解成WRAP_CONTENT,这是的值是父View可以允许的最大的值，只要不超过这个值都可以。
- MeasureSpec.UNSPECIFIED The parent has not imposed any constraint on the child. It can be whatever size it wants. 这种情况比较少，一般用不到。

这里简单总结一下上面的过程:
```java
performMeasure() {
	- 1.调用View.measure方法
	mView.measure():
		- 2.measure内部会调用onMeasure方法,但是因为这里mView是DecorView，所以会调用FrameLayout的onMeasure方法。
	        onMeasure(FrameLayout)
			- 3. 内部设置ViewGroup的宽高
				setMeasuredDimension
				并且对每个子View进行遍历测量
				for (int i = 0; i < count; i++) {
					final View child = getChildAt(i);
					- 4. 对每个子View调用measureChildWithMargins方法
					measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);	
					    -5. measureChildWithMargins内部调用子View的measure方法
					        meausre
							- 6. measure方法内部又调用onMeasure方法
					            onMeasure(View)
					            - 7. onMeasure方法内部调用setMeasuredDimension
									setMeasuredDimension
									- 8. setMeasuredDimension内部调用setMeasuredDimensionRaw
									    setMeasuredDimensionRaw
				}
}
```

从上面代码中能看到`measure`是`final`的，我们可以重写`onMeasure`来实现`measure`过程。
到这里基本都讲完了，我们在开发中会按照需要重写`onMeasure`方法，然后调用`setMeasuredDimension`方法设置大小，
ps:譬如我们设置了`setMeasuredDimension(10, 10)`,那么不管布局中怎么设置这个`View`的大小
都是没用的，最后显示出来大小都是10*10。

`Layout`
===
`performLayout`方法源码如下：
```java
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
		int desiredWindowHeight) {
	mLayoutRequested = false;
	mScrollMayChange = true;
	mInLayout = true;

	final View host = mView;
	if (DEBUG_ORIENTATION || DEBUG_LAYOUT) {
		Log.v(TAG, "Laying out " + host + " to (" +
				host.getMeasuredWidth() + ", " + host.getMeasuredHeight() + ")");
	}

	Trace.traceBegin(Trace.TRACE_TAG_VIEW, "layout");
	try {
		// 把刚才测量的宽高设置进来
		host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());

		mInLayout = false;
		int numViewsRequestingLayout = mLayoutRequesters.size();
		if (numViewsRequestingLayout > 0) {
			// requestLayout() was called during layout.
			// If no layout-request flags are set on the requesting views, there is no problem.
			// If some requests are still pending, then we need to clear those flags and do
			// a full request/measure/layout pass to handle this situation.
			ArrayList<View> validLayoutRequesters = getValidLayoutRequesters(mLayoutRequesters,
					false);
			if (validLayoutRequesters != null) {
				// Set this flag to indicate that any further requests are happening during
				// the second pass, which may result in posting those requests to the next
				// frame instead
				mHandlingLayoutInLayoutRequest = true;

				// Process fresh layout requests, then measure and layout
				int numValidRequests = validLayoutRequesters.size();
				for (int i = 0; i < numValidRequests; ++i) {
					final View view = validLayoutRequesters.get(i);
					Log.w("View", "requestLayout() improperly called by " + view +
							" during layout: running second layout pass");
					view.requestLayout();
				}
				measureHierarchy(host, lp, mView.getContext().getResources(),
						desiredWindowWidth, desiredWindowHeight);
				mInLayout = true;
				host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());

				mHandlingLayoutInLayoutRequest = false;

				// Check the valid requests again, this time without checking/clearing the
				// layout flags, since requests happening during the second pass get noop'd
				validLayoutRequesters = getValidLayoutRequesters(mLayoutRequesters, true);
				if (validLayoutRequesters != null) {
					final ArrayList<View> finalRequesters = validLayoutRequesters;
					// Post second-pass requests to the next frame
					getRunQueue().post(new Runnable() {
						@Override
						public void run() {
							int numValidRequests = finalRequesters.size();
							for (int i = 0; i < numValidRequests; ++i) {
								final View view = finalRequesters.get(i);
								Log.w("View", "requestLayout() improperly called by " + view +
										" during second layout pass: posting in next frame");
								view.requestLayout();
							}
						}
					});
				}
			}

		}
	} finally {
		Trace.traceEnd(Trace.TRACE_TAG_VIEW);
	}
	mInLayout = false;
}
```

内部会调用`layout()`方法，因为`host`是`mView`，`ViewGroup`中重写了`layout`方法，并调用了`super.layout`.
所以我们直接看`View.layout()`方法，该方法源码如下： 
```java
/**
 * Assign a size and position to a view and all of its
 * descendants
 *
 * <p>This is the second phase of the layout mechanism.
 * (The first is measuring). In this phase, each parent calls
 * layout on all of its children to position them.
 * This is typically done using the child measurements
 * that were stored in the measure pass().</p>
 *
 * <p>Derived classes should not override this method.
 * Derived classes with children should override
 * onLayout. In that method, they should
 * call layout on each of their children.</p>
 *
 * @param l Left position, relative to parent
 * @param t Top position, relative to parent
 * @param r Right position, relative to parent
 * @param b Bottom position, relative to parent
 */
@SuppressWarnings({"unchecked"})
public void layout(int l, int t, int r, int b) {
	if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
		onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
		mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
	}

	int oldL = mLeft;
	int oldT = mTop;
	int oldB = mBottom;
	int oldR = mRight;

	// 这部分是判断这个View的大小是否已经发生了变化，来判断是否需要重绘。
	boolean changed = isLayoutModeOptical(mParent) ?
			setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

	if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
		// 内部调用onLayout方法
		onLayout(changed, l, t, r, b);
		mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

		ListenerInfo li = mListenerInfo;
		if (li != null && li.mOnLayoutChangeListeners != null) {
			ArrayList<OnLayoutChangeListener> listenersCopy =
					(ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
			int numListeners = listenersCopy.size();
			for (int i = 0; i < numListeners; ++i) {
				listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
			}
		}
	}

	mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
	mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
```
这里会调用`onLayout`方法，同样因为`mView`是`FrameLayout`的子类，所以我们要看`FrameLayout`的`onLayout`方法，
这里我们先看一下`ViewGroup.onLayout`方法:
```java
/**
 * {@inheritDoc}
 */
@Override
protected abstract void onLayout(boolean changed,
		int l, int t, int r, int b);
```
是个抽象方法，所以`ViewGroup`的子类都需要实现该方法。
我们看一下`FrameLayout.onLayout`方法,源码如下：
```java
 /**
 * {@inheritDoc}
 */
@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
	layoutChildren(left, top, right, bottom, false /* no force left gravity */);
}

void layoutChildren(int left, int top, int right, int bottom,
							  boolean forceLeftGravity) {
	final int count = getChildCount();

	final int parentLeft = getPaddingLeftWithForeground();
	final int parentRight = right - left - getPaddingRightWithForeground();

	final int parentTop = getPaddingTopWithForeground();
	final int parentBottom = bottom - top - getPaddingBottomWithForeground();

	mForegroundBoundsChanged = true;
	
	for (int i = 0; i < count; i++) {
		final View child = getChildAt(i);
		if (child.getVisibility() != GONE) {
			final LayoutParams lp = (LayoutParams) child.getLayoutParams();

			final int width = child.getMeasuredWidth();
			final int height = child.getMeasuredHeight();

			int childLeft;
			int childTop;

			int gravity = lp.gravity;
			if (gravity == -1) {
				gravity = DEFAULT_CHILD_GRAVITY;
			}

			final int layoutDirection = getLayoutDirection();
			final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
			final int verticalGravity = gravity & Gravity.VERTICAL_GRAVITY_MASK;

			switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
				case Gravity.CENTER_HORIZONTAL:
					childLeft = parentLeft + (parentRight - parentLeft - width) / 2 +
					lp.leftMargin - lp.rightMargin;
					break;
				case Gravity.RIGHT:
					if (!forceLeftGravity) {
						childLeft = parentRight - width - lp.rightMargin;
						break;
					}
				case Gravity.LEFT:
				default:
					childLeft = parentLeft + lp.leftMargin;
			}

			switch (verticalGravity) {
				case Gravity.TOP:
					childTop = parentTop + lp.topMargin;
					break;
				case Gravity.CENTER_VERTICAL:
					childTop = parentTop + (parentBottom - parentTop - height) / 2 +
					lp.topMargin - lp.bottomMargin;
					break;
				case Gravity.BOTTOM:
					childTop = parentBottom - height - lp.bottomMargin;
					break;
				default:
					childTop = parentTop + lp.topMargin;
			}
			//调用子View的layout方法
			child.layout(childLeft, childTop, childLeft + width, childTop + height);
		}
	}
}
```
而`View.layout`方法，又会调用到`View.onLayout`方法，我们假设这个子`View`不是`ViewGroup`.
看一下`View.onLayout`方法源码如下： 
```java
/**
 * Called from layout when this view should
 * assign a size and position to each of its children.
 *
 * Derived classes with children should override
 * this method and call layout on each of
 * their children.
 * @param changed This is a new size or position for this view
 * @param left Left position, relative to parent
 * @param top Top position, relative to parent
 * @param right Right position, relative to parent
 * @param bottom Bottom position, relative to parent
 */
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```
是一个空方法，这是因为`Layout`需要`ViewGroup`来控制进行。      

这里也总结一下`layout`的过程。
```java
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
		int desiredWindowHeight) {
	- 1. host.layout
	host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
		-2. layout方法会分别调用setFrame()和onLayout()方法
			setFrame()
			onLayout()
			-3. 因为host是mView也就是DecorView也就是FrameLayout的子类。FrameLayout的onLayout方法如下
				for (int i = 0; i < count; i++) {
					final View child = getChildAt(i);
					if (child.getVisibility() != GONE) {
					    -4. 遍历每个子View，并分别调用layout方法。
						child.layout(childLeft, childTop, childLeft + width, childTop + height);
					}
				}
}
```

`Draw`
===

绘制阶段是从`ViewRootImpl`中的`performDraw`方法开始的：
```java
private void performDraw() {
	if (mAttachInfo.mDisplayState == Display.STATE_OFF && !mReportNextDraw) {
		return;
	}

	final boolean fullRedrawNeeded = mFullRedrawNeeded;
	mFullRedrawNeeded = false;

	mIsDrawing = true;
	Trace.traceBegin(Trace.TRACE_TAG_VIEW, "draw");
	try {
		// 开始draw了
		draw(fullRedrawNeeded);
	} finally {
		mIsDrawing = false;
		Trace.traceEnd(Trace.TRACE_TAG_VIEW);
	}

	// For whatever reason we didn't create a HardwareRenderer, end any
	// hardware animations that are now dangling
	if (mAttachInfo.mPendingAnimatingRenderNodes != null) {
		final int count = mAttachInfo.mPendingAnimatingRenderNodes.size();
		for (int i = 0; i < count; i++) {
			mAttachInfo.mPendingAnimatingRenderNodes.get(i).endAllAnimators();
		}
		mAttachInfo.mPendingAnimatingRenderNodes.clear();
	}

	if (mReportNextDraw) {
		mReportNextDraw = false;
		if (mAttachInfo.mHardwareRenderer != null) {
			mAttachInfo.mHardwareRenderer.fence();
		}

		if (LOCAL_LOGV) {
			Log.v(TAG, "FINISHED DRAWING: " + mWindowAttributes.getTitle());
		}
		if (mSurfaceHolder != null && mSurface.isValid()) {
			mSurfaceHolderCallback.surfaceRedrawNeeded(mSurfaceHolder);
			SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
			if (callbacks != null) {
				for (SurfaceHolder.Callback c : callbacks) {
					if (c instanceof SurfaceHolder.Callback2) {
						((SurfaceHolder.Callback2)c).surfaceRedrawNeeded(
								mSurfaceHolder);
					}
				}
			}
		}
		try {
			mWindowSession.finishDrawing(mWindow);
		} catch (RemoteException e) {
		}
	}
}
```
内部会调用`draw`方法，`draw`方法源码如下：
```java
private void draw(boolean fullRedrawNeeded) {
	Surface surface = mSurface;
	if (!surface.isValid()) {
		return;
	}

	if (DEBUG_FPS) {
		trackFPS();
	}

	if (!sFirstDrawComplete) {
		synchronized (sFirstDrawHandlers) {
			sFirstDrawComplete = true;
			final int count = sFirstDrawHandlers.size();
			for (int i = 0; i< count; i++) {
				mHandler.post(sFirstDrawHandlers.get(i));
			}
		}
	}

	scrollToRectOrFocus(null, false);

	if (mAttachInfo.mViewScrollChanged) {
		mAttachInfo.mViewScrollChanged = false;
		mAttachInfo.mTreeObserver.dispatchOnScrollChanged();
	}

	boolean animating = mScroller != null && mScroller.computeScrollOffset();
	final int curScrollY;
	if (animating) {
		curScrollY = mScroller.getCurrY();
	} else {
		curScrollY = mScrollY;
	}
	if (mCurScrollY != curScrollY) {
		mCurScrollY = curScrollY;
		fullRedrawNeeded = true;
	}

	final float appScale = mAttachInfo.mApplicationScale;
	final boolean scalingRequired = mAttachInfo.mScalingRequired;

	int resizeAlpha = 0;
	if (mResizeBuffer != null) {
		long deltaTime = SystemClock.uptimeMillis() - mResizeBufferStartTime;
		if (deltaTime < mResizeBufferDuration) {
			float amt = deltaTime/(float) mResizeBufferDuration;
			amt = mResizeInterpolator.getInterpolation(amt);
			animating = true;
			resizeAlpha = 255 - (int)(amt*255);
		} else {
			disposeResizeBuffer();
		}
	}

	final Rect dirty = mDirty;
	if (mSurfaceHolder != null) {
		// The app owns the surface, we won't draw.
		dirty.setEmpty();
		if (animating) {
			if (mScroller != null) {
				mScroller.abortAnimation();
			}
			disposeResizeBuffer();
		}
		return;
	}

	if (fullRedrawNeeded) {
		mAttachInfo.mIgnoreDirtyState = true;
		dirty.set(0, 0, (int) (mWidth * appScale + 0.5f), (int) (mHeight * appScale + 0.5f));
	}

	if (DEBUG_ORIENTATION || DEBUG_DRAW) {
		Log.v(TAG, "Draw " + mView + "/"
				+ mWindowAttributes.getTitle()
				+ ": dirty={" + dirty.left + "," + dirty.top
				+ "," + dirty.right + "," + dirty.bottom + "} surface="
				+ surface + " surface.isValid()=" + surface.isValid() + ", appScale:" +
				appScale + ", width=" + mWidth + ", height=" + mHeight);
	}

	mAttachInfo.mTreeObserver.dispatchOnDraw();

	int xOffset = 0;
	int yOffset = curScrollY;
	final WindowManager.LayoutParams params = mWindowAttributes;
	final Rect surfaceInsets = params != null ? params.surfaceInsets : null;
	if (surfaceInsets != null) {
		xOffset -= surfaceInsets.left;
		yOffset -= surfaceInsets.top;

		// Offset dirty rect for surface insets.
		dirty.offset(surfaceInsets.left, surfaceInsets.right);
	}

	if (!dirty.isEmpty() || mIsAnimating) {
		if (mAttachInfo.mHardwareRenderer != null && mAttachInfo.mHardwareRenderer.isEnabled()) {
			// Draw with hardware renderer.
			mIsAnimating = false;
			boolean invalidateRoot = false;
			if (mHardwareYOffset != yOffset || mHardwareXOffset != xOffset) {
				mHardwareYOffset = yOffset;
				mHardwareXOffset = xOffset;
				mAttachInfo.mHardwareRenderer.invalidateRoot();
			}
			mResizeAlpha = resizeAlpha;

			dirty.setEmpty();

			mBlockResizeBuffer = false;
			mAttachInfo.mHardwareRenderer.draw(mView, mAttachInfo, this);
		} else {
			// If we get here with a disabled & requested hardware renderer, something went
			// wrong (an invalidate posted right before we destroyed the hardware surface
			// for instance) so we should just bail out. Locking the surface with software
			// rendering at this point would lock it forever and prevent hardware renderer
			// from doing its job when it comes back.
			// Before we request a new frame we must however attempt to reinitiliaze the
			// hardware renderer if it's in requested state. This would happen after an
			// eglTerminate() for instance.
			if (mAttachInfo.mHardwareRenderer != null &&
					!mAttachInfo.mHardwareRenderer.isEnabled() &&
					mAttachInfo.mHardwareRenderer.isRequested()) {

				try {
					mAttachInfo.mHardwareRenderer.initializeIfNeeded(
							mWidth, mHeight, mSurface, surfaceInsets);
				} catch (OutOfResourcesException e) {
					handleOutOfResourcesException(e);
					return;
				}

				mFullRedrawNeeded = true;
				scheduleTraversals();
				return;
			}
			
			// draw的部分在这里。。。内部会用canvas去画
			if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset, scalingRequired, dirty)) {
				return;
			}
		}
	}

	if (animating) {
		mFullRedrawNeeded = true;
		scheduleTraversals();
	}
}
```
我们看一下`drawSoftware`方法： 
```java
/**
 * @return true if drawing was successful, false if an error occurred
 */
private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
		boolean scalingRequired, Rect dirty) {

	// Draw with software renderer.
	final Canvas canvas;
	try {
		final int left = dirty.left;
		final int top = dirty.top;
		final int right = dirty.right;
		final int bottom = dirty.bottom;

		canvas = mSurface.lockCanvas(dirty);

		// The dirty rectangle can be modified by Surface.lockCanvas()
		//noinspection ConstantConditions
		if (left != dirty.left || top != dirty.top || right != dirty.right
				|| bottom != dirty.bottom) {
			attachInfo.mIgnoreDirtyState = true;
		}

		// TODO: Do this in native
		canvas.setDensity(mDensity);
	} catch (Surface.OutOfResourcesException e) {
		handleOutOfResourcesException(e);
		return false;
	} catch (IllegalArgumentException e) {
		Log.e(TAG, "Could not lock surface", e);
		// Don't assume this is due to out of memory, it could be
		// something else, and if it is something else then we could
		// kill stuff (or ourself) for no reason.
		mLayoutRequested = true;    // ask wm for a new surface next time.
		return false;
	}

	try {
		if (DEBUG_ORIENTATION || DEBUG_DRAW) {
			Log.v(TAG, "Surface " + surface + " drawing to bitmap w="
					+ canvas.getWidth() + ", h=" + canvas.getHeight());
			//canvas.drawARGB(255, 255, 0, 0);
		}

		// If this bitmap's format includes an alpha channel, we
		// need to clear it before drawing so that the child will
		// properly re-composite its drawing on a transparent
		// background. This automatically respects the clip/dirty region
		// or
		// If we are applying an offset, we need to clear the area
		// where the offset doesn't appear to avoid having garbage
		// left in the blank areas.
		if (!canvas.isOpaque() || yoff != 0 || xoff != 0) {
			canvas.drawColor(0, PorterDuff.Mode.CLEAR);
		}

		dirty.setEmpty();
		mIsAnimating = false;
		attachInfo.mDrawingTime = SystemClock.uptimeMillis();
		mView.mPrivateFlags |= View.PFLAG_DRAWN;

		if (DEBUG_DRAW) {
			Context cxt = mView.getContext();
			Log.i(TAG, "Drawing: package:" + cxt.getPackageName() +
					", metrics=" + cxt.getResources().getDisplayMetrics() +
					", compatibilityInfo=" + cxt.getResources().getCompatibilityInfo());
		}
		try {
			canvas.translate(-xoff, -yoff);
			if (mTranslator != null) {
				mTranslator.translateCanvas(canvas);
			}
			canvas.setScreenDensity(scalingRequired ? mNoncompatDensity : 0);
			attachInfo.mSetIgnoreDirtyState = false;
			
			// 内部会去调用View.draw()；
			mView.draw(canvas);
		} finally {
			if (!attachInfo.mSetIgnoreDirtyState) {
				// Only clear the flag if it was not set during the mView.draw() call
				attachInfo.mIgnoreDirtyState = false;
			}
		}
	} finally {
		try {
			surface.unlockCanvasAndPost(canvas);
		} catch (IllegalArgumentException e) {
			Log.e(TAG, "Could not unlock surface", e);
			mLayoutRequested = true;    // ask wm for a new surface next time.
			//noinspection ReturnInsideFinallyBlock
			return false;
		}

		if (LOCAL_LOGV) {
			Log.v(TAG, "Surface " + surface + " unlockCanvasAndPost");
		}
	}
	return true;
}
```
代码中调用了`mView.draw()`方法，所以我们看一下`FrameLayout.draw()`方法：
```java
/**
 * {@inheritDoc}
 */
@Override
public void draw(Canvas canvas) {
	super.draw(canvas);

	if (mForeground != null) {
		final Drawable foreground = mForeground;

		if (mForegroundBoundsChanged) {
			mForegroundBoundsChanged = false;
			final Rect selfBounds = mSelfBounds;
			final Rect overlayBounds = mOverlayBounds;

			final int w = mRight-mLeft;
			final int h = mBottom-mTop;

			if (mForegroundInPadding) {
				selfBounds.set(0, 0, w, h);
			} else {
				selfBounds.set(mPaddingLeft, mPaddingTop, w - mPaddingRight, h - mPaddingBottom);
			}

			final int layoutDirection = getLayoutDirection();
			Gravity.apply(mForegroundGravity, foreground.getIntrinsicWidth(),
					foreground.getIntrinsicHeight(), selfBounds, overlayBounds,
					layoutDirection);
			foreground.setBounds(overlayBounds);
		}
		
		foreground.draw(canvas);
	}
}
```
内部调用了`super.draw()`，而`ViewGroup`没有重写该方法，所以直接看`View`的`draw()`方法.
`View.draw()`方法如下：
```java
/**
 * Manually render this view (and all of its children) to the given Canvas.
 * The view must have already done a full layout before this function is
 * called.  When implementing a view, implement
 * {@link #onDraw(android.graphics.Canvas)} instead of overriding this method.
 * If you do need to override this method, call the superclass version.
 *
 * @param canvas The Canvas to which the View is rendered.
 */
public void draw(Canvas canvas) {
	final int privateFlags = mPrivateFlags;
	final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
			(mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
	mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

	// 这里注释说的很明白了，draw的6个步骤。
	/*
	 * Draw traversal performs several drawing steps which must be executed
	 * in the appropriate order:
	 *
	 *      1. Draw the background
	 *      2. If necessary, save the canvas' layers to prepare for fading
	 *      3. Draw view's content, 调用onDraw方法绘制自身
	 *      4. Draw children, 调用dispatchDraw方法绘制子View
	 *      5. If necessary, draw the fading edges and restore layers
	 *      6. Draw decorations (scrollbars for instance)
	 */

	// Step 1, draw the background, if needed
	int saveCount;

	if (!dirtyOpaque) {
		drawBackground(canvas);
	}

	// skip step 2 & 5 if possible (common case)
	final int viewFlags = mViewFlags;
	boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
	boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
	if (!verticalEdges && !horizontalEdges) {
		// Step 3, draw the content
		if (!dirtyOpaque) onDraw(canvas);

		// Step 4, draw the children
		dispatchDraw(canvas);

		// Step 6, draw decorations (scrollbars)
		onDrawScrollBars(canvas);

		if (mOverlay != null && !mOverlay.isEmpty()) {
			mOverlay.getOverlayView().dispatchDraw(canvas);
		}

		// we're done...
		return;
	}

	/*
	 * Here we do the full fledged routine...
	 * (this is an uncommon case where speed matters less,
	 * this is why we repeat some of the tests that have been
	 * done above)
	 */

	boolean drawTop = false;
	boolean drawBottom = false;
	boolean drawLeft = false;
	boolean drawRight = false;

	float topFadeStrength = 0.0f;
	float bottomFadeStrength = 0.0f;
	float leftFadeStrength = 0.0f;
	float rightFadeStrength = 0.0f;

	// Step 2, save the canvas' layers
	int paddingLeft = mPaddingLeft;

	final boolean offsetRequired = isPaddingOffsetRequired();
	if (offsetRequired) {
		paddingLeft += getLeftPaddingOffset();
	}

	int left = mScrollX + paddingLeft;
	int right = left + mRight - mLeft - mPaddingRight - paddingLeft;
	int top = mScrollY + getFadeTop(offsetRequired);
	int bottom = top + getFadeHeight(offsetRequired);

	if (offsetRequired) {
		right += getRightPaddingOffset();
		bottom += getBottomPaddingOffset();
	}

	final ScrollabilityCache scrollabilityCache = mScrollCache;
	final float fadeHeight = scrollabilityCache.fadingEdgeLength;
	int length = (int) fadeHeight;

	// clip the fade length if top and bottom fades overlap
	// overlapping fades produce odd-looking artifacts
	if (verticalEdges && (top + length > bottom - length)) {
		length = (bottom - top) / 2;
	}

	// also clip horizontal fades if necessary
	if (horizontalEdges && (left + length > right - length)) {
		length = (right - left) / 2;
	}

	if (verticalEdges) {
		topFadeStrength = Math.max(0.0f, Math.min(1.0f, getTopFadingEdgeStrength()));
		drawTop = topFadeStrength * fadeHeight > 1.0f;
		bottomFadeStrength = Math.max(0.0f, Math.min(1.0f, getBottomFadingEdgeStrength()));
		drawBottom = bottomFadeStrength * fadeHeight > 1.0f;
	}

	if (horizontalEdges) {
		leftFadeStrength = Math.max(0.0f, Math.min(1.0f, getLeftFadingEdgeStrength()));
		drawLeft = leftFadeStrength * fadeHeight > 1.0f;
		rightFadeStrength = Math.max(0.0f, Math.min(1.0f, getRightFadingEdgeStrength()));
		drawRight = rightFadeStrength * fadeHeight > 1.0f;
	}

	saveCount = canvas.getSaveCount();

	int solidColor = getSolidColor();
	if (solidColor == 0) {
		final int flags = Canvas.HAS_ALPHA_LAYER_SAVE_FLAG;

		if (drawTop) {
			canvas.saveLayer(left, top, right, top + length, null, flags);
		}

		if (drawBottom) {
			canvas.saveLayer(left, bottom - length, right, bottom, null, flags);
		}

		if (drawLeft) {
			canvas.saveLayer(left, top, left + length, bottom, null, flags);
		}

		if (drawRight) {
			canvas.saveLayer(right - length, top, right, bottom, null, flags);
		}
	} else {
		scrollabilityCache.setFadeColor(solidColor);
	}

	// Step 3, draw the content
	if (!dirtyOpaque) onDraw(canvas);

	// Step 4, draw the children
	dispatchDraw(canvas);

	// Step 5, draw the fade effect and restore layers
	final Paint p = scrollabilityCache.paint;
	final Matrix matrix = scrollabilityCache.matrix;
	final Shader fade = scrollabilityCache.shader;

	if (drawTop) {
		matrix.setScale(1, fadeHeight * topFadeStrength);
		matrix.postTranslate(left, top);
		fade.setLocalMatrix(matrix);
		p.setShader(fade);
		canvas.drawRect(left, top, right, top + length, p);
	}

	if (drawBottom) {
		matrix.setScale(1, fadeHeight * bottomFadeStrength);
		matrix.postRotate(180);
		matrix.postTranslate(left, bottom);
		fade.setLocalMatrix(matrix);
		p.setShader(fade);
		canvas.drawRect(left, bottom - length, right, bottom, p);
	}

	if (drawLeft) {
		matrix.setScale(1, fadeHeight * leftFadeStrength);
		matrix.postRotate(-90);
		matrix.postTranslate(left, top);
		fade.setLocalMatrix(matrix);
		p.setShader(fade);
		canvas.drawRect(left, top, left + length, bottom, p);
	}

	if (drawRight) {
		matrix.setScale(1, fadeHeight * rightFadeStrength);
		matrix.postRotate(90);
		matrix.postTranslate(right, top);
		fade.setLocalMatrix(matrix);
		p.setShader(fade);
		canvas.drawRect(right - length, top, right, bottom, p);
	}

	canvas.restoreToCount(saveCount);

	// Step 6, draw decorations (scrollbars)
	onDrawScrollBars(canvas);

	if (mOverlay != null && !mOverlay.isEmpty()) {
		mOverlay.getOverlayView().dispatchDraw(canvas);
	}
}
```

上面会调用`onDraw`和`dispatchDraw`方法。
我们先看一下`View.onDraw`方法：
```java
/**
 * Implement this to do your drawing.
 *
 * @param canvas the canvas on which the background will be drawn
 */
protected void onDraw(Canvas canvas) {
}
```
是空方法，这是也很好理解，因为每个`View`的展现都不一样，例如`TextView`、`ProgressBar`等，
所以`View`不会去实现`onDraw`方法，具体是要子类去根据自己的显示要求实现该方法。

再看一下`dispatchDraw`方法，这个方法是用来绘制子`View`的，所以要看`ViewGroup.dispatchDraw`方法，`View.dispatchDraw`是空的。 
```java
/**
 * {@inheritDoc}
 */
@Override
protected void dispatchDraw(Canvas canvas) {
	boolean usingRenderNodeProperties = canvas.isRecordingFor(mRenderNode);
	final int childrenCount = mChildrenCount;
	final View[] children = mChildren;
	int flags = mGroupFlags;

	if ((flags & FLAG_RUN_ANIMATION) != 0 && canAnimate()) {
		final boolean cache = (mGroupFlags & FLAG_ANIMATION_CACHE) == FLAG_ANIMATION_CACHE;

		final boolean buildCache = !isHardwareAccelerated();
		for (int i = 0; i < childrenCount; i++) {
			final View child = children[i];
			if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE) {
				final LayoutParams params = child.getLayoutParams();
				attachLayoutAnimationParameters(child, params, i, childrenCount);
				bindLayoutAnimation(child);
				if (cache) {
					child.setDrawingCacheEnabled(true);
					if (buildCache) {
						child.buildDrawingCache(true);
					}
				}
			}
		}

		final LayoutAnimationController controller = mLayoutAnimationController;
		if (controller.willOverlap()) {
			mGroupFlags |= FLAG_OPTIMIZE_INVALIDATE;
		}

		controller.start();

		mGroupFlags &= ~FLAG_RUN_ANIMATION;
		mGroupFlags &= ~FLAG_ANIMATION_DONE;

		if (cache) {
			mGroupFlags |= FLAG_CHILDREN_DRAWN_WITH_CACHE;
		}

		if (mAnimationListener != null) {
			mAnimationListener.onAnimationStart(controller.getAnimation());
		}
	}

	int clipSaveCount = 0;
	final boolean clipToPadding = (flags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK;
	if (clipToPadding) {
		clipSaveCount = canvas.save();
		canvas.clipRect(mScrollX + mPaddingLeft, mScrollY + mPaddingTop,
				mScrollX + mRight - mLeft - mPaddingRight,
				mScrollY + mBottom - mTop - mPaddingBottom);
	}

	// We will draw our child's animation, let's reset the flag
	mPrivateFlags &= ~PFLAG_DRAW_ANIMATION;
	mGroupFlags &= ~FLAG_INVALIDATE_REQUIRED;

	boolean more = false;
	final long drawingTime = getDrawingTime();

	if (usingRenderNodeProperties) canvas.insertReorderBarrier();
	// Only use the preordered list if not HW accelerated, since the HW pipeline will do the
	// draw reordering internally
	final ArrayList<View> preorderedList = usingRenderNodeProperties
			? null : buildOrderedChildList();
	final boolean customOrder = preorderedList == null
			&& isChildrenDrawingOrderEnabled();
	for (int i = 0; i < childrenCount; i++) {
		int childIndex = customOrder ? getChildDrawingOrder(childrenCount, i) : i;
		final View child = (preorderedList == null)
				? children[childIndex] : preorderedList.get(childIndex);
		if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
			// 调用drawChild方法
			more |= drawChild(canvas, child, drawingTime);
		}
	}
	if (preorderedList != null) preorderedList.clear();

	// Draw any disappearing views that have animations
	if (mDisappearingChildren != null) {
		final ArrayList<View> disappearingChildren = mDisappearingChildren;
		final int disappearingCount = disappearingChildren.size() - 1;
		// Go backwards -- we may delete as animations finish
		for (int i = disappearingCount; i >= 0; i--) {
			final View child = disappearingChildren.get(i);
			more |= drawChild(canvas, child, drawingTime);
		}
	}
	if (usingRenderNodeProperties) canvas.insertInorderBarrier();

	if (debugDraw()) {
		onDebugDraw(canvas);
	}

	if (clipToPadding) {
		canvas.restoreToCount(clipSaveCount);
	}

	// mGroupFlags might have been updated by drawChild()
	flags = mGroupFlags;

	if ((flags & FLAG_INVALIDATE_REQUIRED) == FLAG_INVALIDATE_REQUIRED) {
		invalidate(true);
	}

	if ((flags & FLAG_ANIMATION_DONE) == 0 && (flags & FLAG_NOTIFY_ANIMATION_LISTENER) == 0 &&
			mLayoutAnimationController.isDone() && !more) {
		// We want to erase the drawing cache and notify the listener after the
		// next frame is drawn because one extra invalidate() is caused by
		// drawChild() after the animation is over
		mGroupFlags |= FLAG_NOTIFY_ANIMATION_LISTENER;
		final Runnable end = new Runnable() {
		   public void run() {
			   notifyAnimationListener();
		   }
		};
		post(end);
	}
}
```
可以看到上面的方法中会调用`drawChild`方法，该方法如下： 
```java
/**
 * Draw one child of this View Group. This method is responsible for getting
 * the canvas in the right state. This includes clipping, translating so
 * that the child's scrolled origin is at 0, 0, and applying any animation
 * transformations.
 *
 * @param canvas The canvas on which to draw the child
 * @param child Who to draw
 * @param drawingTime The time at which draw is occurring
 * @return True if an invalidate() was issued
 */
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
	return child.draw(canvas, this, drawingTime);
}
```

这里也简单总结一下`draw`的过程：
```java
// 1. ViewRootImpl.performDraw()
private void performDraw() {
	// 2. ViewRootImpl.draw()
	draw(fullRedrawNeeded);	
		// 3. ViewRootImpl.drawSoftware
		drawSoftware
			// 4. 内部调用mView.draw,也就是FrameLayout.draw(). 
			mView.draw()(FrameLayout)
				// 5. FrameLayout.draw方法内部会调用super.draw方法，也就是View.draw方法.
				super.draw(canvas);
					// 6. View.draw方法内部会分别调用onDraw绘制自己以及dispatchDraw绘制子View.
					onDraw
					// 绘制子View
					dispatchDraw
						// 7. dispatchDraw方法内部会遍历所有子View.
						for (int i = 0; i < childrenCount; i++) {
							// 8. 对每个子View分别调用drawChild方法
							drawChild()
								// 9. drawChild方法内部会对该子View调用draw方法，进行绘制。然后draw又会调用onDraw等，循环就开始了。 
									child.draw()
						}
}
```

最后补充一个小问题： `getWidth()`与`getMeasuredWidth()`有什么区别呢？                          
一般情况下这两个的值是相同的，`getMeasureWidth()`方法在`measure()`过程结束后就可以获取到了，而`getWidth()`方法要在`layout()`过程结束后才能获取到。
而且`getMeasureWidth()`的值是通过`setMeasuredDimension()`设置的，但是`getWidth()`的值是通过视图右边的坐标减去左边的坐标计算出来的。如果我们在`layout`的时候将宽高
不传`getMeasureWidth`的值，那么这时候`getWidth()`与`getMeasuredWidth`的值就不会再相同了，当然一般也不会这么干...

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 