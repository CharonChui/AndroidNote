Android Touch事件分发详解
===

先说一些基本的知识，方便后面分析源码时能更好理解。
- 所有`Touch`事件都被封装成`MotionEvent`对象，包括`Touch`的位置、历史记录、第几个手指等.

- 事件类型分为`ACTION_DOWN`,`ACTION_UP`,`ACTION_MOVE`,`ACTION_POINTER_DOWN`,`ACTION_POINTER_UP`,`ACTION_CANCEL`, 每个
一个完整的事件以`ACTION_DOWN`开始`ACTION_UP`结束，并且`ACTION_CANCEL`只能由代码引起.一般对于`CANCEL`的处理和`UP`的相同。
`CANCEL`的一个简单例子：手指在移动的过程中突然移动到了边界外，那么这时`ACTION_UP`事件了，所以这是的`CANCEL`和`UP`的处理是一致的。

- 事件的处理分别为`dispatchTouchEveent()`分发事件(`TextView`等这种最小的`View`中不会有该方式)、`onInterceptTouchEvent()`拦截事件(`ViewGroup`中拦截事件)、`onTouchEvent()`消费事件.

- 事件从`Activity.dispatchTouchEveent()`开始传递，只要没有停止拦截，就会从最上层(`ViewGroup`)开始一直往下传递，子`View`通过`onTouchEvent()`消费事件。(隧道式向下分发).

- 如果时间从上往下一直传递到最底层的子`View`，但是该`View`没有消费该事件，那么该事件会反序网上传递(从该`View`传递给自己的`ViewGroup`，然后再传给更上层的`ViewGroup`直至传递给`Activity.onTouchEvent()`).
(冒泡式向上处理).

- 如果`View`没有消费`ACTION_DOWN`事件，之后其他的`MOVE`、`UP`等事件都不会传递过来.

- 事件由父`View(ViewGroup)`传递给子`View`,`ViewGroup`可以通过`onInterceptTouchEvent()`方法对事件进行拦截，停止其往下传递，如果拦截(返回`true`)后该事件
会直接走到该`ViewGroup`中的`onTouchEvent()`中，不会再往下传递给子`View`.如果从`DOWN`开始，之后的`MOVE`、`UP`都会直接在该`ViewGroup.onTouchEvent()`中进行处理。
如果子`View`之前在处理某个事件，但是后续被`ViewGroup`拦截，那么子`View`会接收到`ACTION_CANCEL`.

- `OnTouchListener`优先于`onTouchEvent()`对事件进行消费。

- `TouchTarget`是保存手指点击区域属性的一个类，手指的所有移动过程都会被它记录下来, 包含被`touch`的`View`。  

废话不多说，直接上源码，源码妥妥的是最新版5.0：
我们先从`Activity.dispatchTouchEveent()`说起：

```java
/**
 * Called to process touch screen events.  You can override this to
 * intercept all touch screen events before they are dispatched to the
 * window.  Be sure to call this implementation for touch screen events
 * that should be handled normally.
 *
 * @param ev The touch screen event.
 *
 * @return boolean Return true if this event was consumed.
 */
public boolean dispatchTouchEvent(MotionEvent ev) {
	if (ev.getAction() == MotionEvent.ACTION_DOWN) {
		onUserInteraction();
	}
	if (getWindow().superDispatchTouchEvent(ev)) {
		return true;
	}
	return onTouchEvent(ev);
}
```

代码一看能感觉出来`DOWN`事件比较特殊。我们继续走到`onUserInteraction()`代码中.        
```java
/**
 * Called whenever a key, touch, or trackball event is dispatched to the
 * activity.  Implement this method if you wish to know that the user has
 * interacted with the device in some way while your activity is running.
 * This callback and {@link #onUserLeaveHint} are intended to help
 * activities manage status bar notifications intelligently; specifically,
 * for helping activities determine the proper time to cancel a notfication.
 *
 * <p>All calls to your activity's {@link #onUserLeaveHint} callback will
 * be accompanied by calls to {@link #onUserInteraction}.  This
 * ensures that your activity will be told of relevant user activity such
 * as pulling down the notification pane and touching an item there.
 *
 * <p>Note that this callback will be invoked for the touch down action
 * that begins a touch gesture, but may not be invoked for the touch-moved
 * and touch-up actions that follow.
 *
 * @see #onUserLeaveHint()
 */
public void onUserInteraction() {
}
```
但是该方法是空方法，没有具体实现。 我们往下看`getWindow().superDispatchTouchEvent(ev)`.      
`getWindow()`获取到当前`Window`对象，表示顶层窗口，管理界面的显示和事件的响应；每个Activity 均会创建一个PhoneWindow对象，
是Activity和整个View系统交互的接口，但是该类是一个抽象类。
从文档中可以看到`The only existing implementation of this abstract class is android.policy.PhoneWindow, which you should instantiate when needing a Window. `，
所以我们找到`PhoneWindow`类，查看它的`superDispatchTouchEvent()`方法。
```java
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
	return mDecor.superDispatchTouchEvent(event);
}
```
该方法又是调用了`mDecor.superDispatchTouchEvent(event)`, `mDecor`是什么呢？ 从名字中我们大概也能猜出来是当前窗口最顶层的`DecorView`，
`Window`界面的最顶层的`View`对象。
```java
// This is the top-level view of the window, containing the window decor.
private DecorView mDecor;
```
讲到这里不妨就提一下`DecorView`.      
```java
private final class DecorView extends FrameLayout implements RootViewSurfaceTaker {
	...
}
```
它集成子`FrameLayout`所有很多时候我们在用布局工具查看的时候发现`Activity`的布局`FrameLayout`的。就是这个原因。       
好了，我们接着看`DecorView`中的`superDispatchTouchEvent()`方法。 
```java
public boolean superDispatchTouchEvent(MotionEvent event) {
	return super.dispatchTouchEvent(event);
}
```
是调用了`super.dispatchTouchEveent()`，而`DecorView`的父类是`FrameLayout`所以我们找到`FrameLayout.dispatchTouchEveent()`.
我们看到`FrameLayout`中没有重写`dispatchTouchEveent()`方法，所以我们再找到`FrameLayout`的父类`ViewGroup`.看`ViewGroup.dispatchTouchEveent()`实现。
新大陆浮现了...     
```java
/**
 * {@inheritDoc}
 */
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {

	// Consistency verifier for debugging purposes.是调试使用的，我们不用管这里了。
	if (mInputEventConsistencyVerifier != null) {
		mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
	}

	boolean handled = false;
	// onFilterTouchEventForSecurity()用安全机制来过滤触摸事件，true为不过滤分发下去，false则销毁掉该事件。
	// 方法具体实现是去判断是否被其它窗口遮挡住了，如果遮挡住就要过滤掉该事件。
	if (onFilterTouchEventForSecurity(ev)) {
		// 没有被其它窗口遮住
		final int action = ev.getAction();
		final int actionMasked = action & MotionEvent.ACTION_MASK;

		// 下面这一块注释说的很清楚了，就是在`Down`的时候把所有的状态都重置，作为一个新事件的开始。
		// Handle an initial down.
		if (actionMasked == MotionEvent.ACTION_DOWN) {
			// Throw away all previous state when starting a new touch gesture.
			// The framework may have dropped the up or cancel event for the previous gesture
			// due to an app switch, ANR, or some other state change.
			cancelAndClearTouchTargets(ev);
			resetTouchState();
			// 如果是`Down`，那么`mFirstTouchTarget`到这里肯定是`null`.因为是新一系列手势的开始。
			// `mFirstTouchTarget`是处理第一个事件的目标。
		}

		// 检查是否拦截该事件(如果`onInterceptTouchEvent()`返回true就拦截该事件)
		// Check for interception.
		final boolean intercepted;
		if (actionMasked == MotionEvent.ACTION_DOWN
				|| mFirstTouchTarget != null) {
			// 标记事件不允许被拦截， 默认是`false`， 该值可以通过`requestDisallowInterceptTouchEvent(true)`方法来设置，
			// 通知父`View`不要拦截该`View`上的事件。
			final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
			if (!disallowIntercept) {
				// 判断该`ViewGroup`是否要拦截该事件。`onInterceptTouchEvent()`方法默认返回`false`即不拦截。
				intercepted = onInterceptTouchEvent(ev);
				ev.setAction(action); // restore action in case it was changed
			} else {
				// 子`View`通知父`View`不要拦截。这样就不会走到上面`onInterceptTouchEvent()`方法中了，
				// 所以父`View`就不会拦截该事件。
				intercepted = false;
			}
		} else {
			// 注释比较清楚了，就是没有目标来处理该事件，而且也不是一个新的事件`Down`事件(新事件的开始), 
			// 我们应该拦截下他。
			// There are no touch targets and this action is not an initial down
			// so this view group continues to intercept touches.
			intercepted = true;
		}

		// Check for cancelation.检查当前是否是`Cancel`事件或者是有`Cancel`标记。
		final boolean canceled = resetCancelNextUpFlag(this)
				|| actionMasked == MotionEvent.ACTION_CANCEL;

		// Update list of touch targets for pointer down, if needed. 这行代码为是否需要将当前的触摸事件分发给多个子`View`，
		// 默认为`true`，分发给多个`View`（比如几个子`View`位置重叠）。默认是true
		final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
		
		// 保存当前要分发给的目标
		TouchTarget newTouchTarget = null;
		boolean alreadyDispatchedToNewTouchTarget = false;
		
		// 如果没取消也不拦截，进入方法内部
		if (!canceled && !intercepted) {
		
			// 下面这部分代码的意思其实就是找到该事件位置下的`View`(可见或者是在动画中的View), 并且与`pointID`关联。
			if (actionMasked == MotionEvent.ACTION_DOWN
					|| (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
					|| actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
				final int actionIndex = ev.getActionIndex(); // always 0 for down
				final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
						: TouchTarget.ALL_POINTER_IDS;

				// Clean up earlier touch targets for this pointer id in case they
				// have become out of sync.
				removePointersFromTouchTargets(idBitsToAssign);

				final int childrenCount = mChildrenCount;
				if (newTouchTarget == null && childrenCount != 0) {
					final float x = ev.getX(actionIndex);
					final float y = ev.getY(actionIndex);
					// Find a child that can receive the event.
					// Scan children from front to back.
					final ArrayList<View> preorderedList = buildOrderedChildList();
					final boolean customOrder = preorderedList == null
							&& isChildrenDrawingOrderEnabled();
					// 遍历找子`View`进行分发了。
					final View[] children = mChildren;
					for (int i = childrenCount - 1; i >= 0; i--) {
						final int childIndex = customOrder
								? getChildDrawingOrder(childrenCount, i) : i;
						final View child = (preorderedList == null)
								? children[childIndex] : preorderedList.get(childIndex);
								
						// `canViewReceivePointerEvents()`方法会去判断这个`View`是否可见或者在播放动画，
						// 只有这两种情况下可以接受事件的分发
						
						// `isTransformedTouchPointInView`判断这个事件的坐标值是否在该`View`内。
						if (!canViewReceivePointerEvents(child)
								|| !isTransformedTouchPointInView(x, y, child, null)) {
							continue;
						}

						// 找到该`View`对应的在`mFristTouchTarget`中的存储的目标， 判断这个`View`可能已经不是之前`mFristTouchTarget`中的`View`了。
						// 如果找不到就返回null, 这种情况是用于多点触摸， 比如在同一个`View`上按下了多跟手指。
						newTouchTarget = getTouchTarget(child);
						if (newTouchTarget != null) {
							// Child View已经接受了这个事件了
							// Child is already receiving touch within its bounds.
							// Give it the new pointer in addition to the ones it is handling.
							newTouchTarget.pointerIdBits |= idBitsToAssign;
							// 找到该View了，不用再循环找了
							break;
						}

						resetCancelNextUpFlag(child);
						// 如果上面没有break，只有newTouchTarget为null，说明上面我们找到的Child View和之前的肯定不是同一个了， 
						// 是新增的， 比如多点触摸的时候，一个手指按在了这个`View`上，另一个手指按在了另一个`View`上。
						// 这时候我们就看child是否分发该事件。dispatchTransformedTouchEvent如果child为null，就直接该ViewGroup出来事件
						// 如果child不为null，就调用child.dispatchTouchEvent
						if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
							// 如果这个Child View能分发，那我们就要把之前存储的值改变成现在的Child View。
							// Child wants to receive touch within its bounds.
							mLastTouchDownTime = ev.getDownTime();
							if (preorderedList != null) {
								// childIndex points into presorted list, find original index
								for (int j = 0; j < childrenCount; j++) {
									if (children[childIndex] == mChildren[j]) {
										mLastTouchDownIndex = j;
										break;
									}
								}
							} else {
								mLastTouchDownIndex = childIndex;
							}
							mLastTouchDownX = ev.getX();
							mLastTouchDownY = ev.getY();
							// 赋值成现在的Child View对应的值，并且会把`mFirstTouchTarget`也改成该值(mFristTouchTarget`与`newTouchTarget`是一样的)。
							newTouchTarget = addTouchTarget(child, idBitsToAssign);
							// 分发给子`View`了，不用再继续循环了
							alreadyDispatchedToNewTouchTarget = true;
							break;
						}
					}
					if (preorderedList != null) preorderedList.clear();
				}

				// `newTouchTarget == null`就是没有找到新的可以分发该事件的子`View`，那我们只能用上一次的分发对象了。
				if (newTouchTarget == null && mFirstTouchTarget != null) {
					// Did not find a child to receive the event.
					// Assign the pointer to the least recently added target.
					newTouchTarget = mFirstTouchTarget;
					while (newTouchTarget.next != null) {
						newTouchTarget = newTouchTarget.next;
					}
					newTouchTarget.pointerIdBits |= idBitsToAssign;
				}
			}
		}

		// DOWN事件在上面会去找touch target
		// Dispatch to touch targets.
		if (mFirstTouchTarget == null) {
			// dispatchTransformedTouchEvent方法中如果child为null，那么就调用super.dispatchTouchEvent(transformedEvent);否则调用child.dispatchTouchEvent(transformedEvent)。
			// `super.dispatchTouchEvent()`也就是说，此时`Viewgroup`处理`touch`消息跟普通`view`一致。普通`View`类内部会调用`onTouchEvent()`方法
			// No touch targets so treat this as an ordinary view. 自己处理
			handled = dispatchTransformedTouchEvent(ev, canceled, null,
					TouchTarget.ALL_POINTER_IDS);
		} else {
			// 分发
			// Dispatch to touch targets, excluding the new touch target if we already
			// dispatched to it.  Cancel touch targets if necessary.
			TouchTarget predecessor = null;
			TouchTarget target = mFirstTouchTarget;
			while (target != null) {
				final TouchTarget next = target.next;
				// 找到了新的子`View`，并且这个是新加的对象，上面已经处理过了。
				if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
					handled = true;
				} else {
					// 否则都调用dispatchTransformedTouchEvent处理，传递给child
					final boolean cancelChild = resetCancelNextUpFlag(target.child)
							|| intercepted;
							
					// 正常分发
					if (dispatchTransformedTouchEvent(ev, cancelChild,
							target.child, target.pointerIdBits)) {
						handled = true;
					}
					
					// 如果是onInterceptTouchEvent返回true就会遍历mFirstTouchTarget全部给销毁，这就是为什么onInterceptTouchEvent返回true，之后所有的时间都不会再继续分发的了。
					if (cancelChild) {
						if (predecessor == null) {
							mFirstTouchTarget = next;
						} else {
							predecessor.next = next;
						}
						target.recycle();
						target = next;
						continue;
					}
				}
				predecessor = target;
				target = next;
			}
		}

		// Update list of touch targets for pointer up or cancel, if needed.
		if (canceled
				|| actionMasked == MotionEvent.ACTION_UP
				|| actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
			resetTouchState();
		} else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
			// 当某个手指抬起的时候，清除他相关的数据。
			final int actionIndex = ev.getActionIndex();
			final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
			removePointersFromTouchTargets(idBitsToRemove);
		}
	}

	if (!handled && mInputEventConsistencyVerifier != null) {
		mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
	}
	return handled;
}
```   

接下来还要说说`dispatchTransformedTouchEvent()`方法，虽然上面也说了大体功能，但是看一下源码能说明另一个问题：   
```java
/**
 * Transforms a motion event into the coordinate space of a particular child view,
 * filters out irrelevant pointer ids, and overrides its action if necessary.
 * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
 */
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
		View child, int desiredPointerIdBits) {
	final boolean handled;

	// Canceling motions is a special case.  We don't need to perform any transformations
	// or filtering.  The important part is the action, not the contents.
	final int oldAction = event.getAction();
	
	// 这就是为什么时间被拦截之后，之前处理过该事件的`View`会收到`CANCEL`.
	if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
		event.setAction(MotionEvent.ACTION_CANCEL);
		if (child == null) {
			handled = super.dispatchTouchEvent(event);
		} else {
			// 子`View`去处理，如果子`View`仍然是`ViewGroup`那还是同样的处理，如果子`View`是普通`View`，普通`View`的`dispatchTouchEveent()`会调用`onTouchEvent()`.
			handled = child.dispatchTouchEvent(event);
		}
		event.setAction(oldAction);
		return handled;
	}

	// Calculate the number of pointers to deliver.
	final int oldPointerIdBits = event.getPointerIdBits();
	final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

	// If for some reason we ended up in an inconsistent state where it looks like we
	// might produce a motion event with no pointers in it, then drop the event.
	if (newPointerIdBits == 0) {
		return false;
	}

	// If the number of pointers is the same and we don't need to perform any fancy
	// irreversible transformations, then we can reuse the motion event for this
	// dispatch as long as we are careful to revert any changes we make.
	// Otherwise we need to make a copy.
	final MotionEvent transformedEvent;
	if (newPointerIdBits == oldPointerIdBits) {
		if (child == null || child.hasIdentityMatrix()) {
			if (child == null) {
				handled = super.dispatchTouchEvent(event);
			} else {
				final float offsetX = mScrollX - child.mLeft;
				final float offsetY = mScrollY - child.mTop;
				event.offsetLocation(offsetX, offsetY);

				handled = child.dispatchTouchEvent(event);

				event.offsetLocation(-offsetX, -offsetY);
			}
			return handled;
		}
		transformedEvent = MotionEvent.obtain(event);
	} else {
		transformedEvent = event.split(newPointerIdBits);
	}

	// Perform any necessary transformations and dispatch.
	if (child == null) {
		handled = super.dispatchTouchEvent(transformedEvent);
	} else {
		final float offsetX = mScrollX - child.mLeft;
		final float offsetY = mScrollY - child.mTop;
		transformedEvent.offsetLocation(offsetX, offsetY);
		if (! child.hasIdentityMatrix()) {
			transformedEvent.transform(child.getInverseMatrix());
		}

		handled = child.dispatchTouchEvent(transformedEvent);
	}

	// Done.
	transformedEvent.recycle();
	return handled;
}
```


上面讲了`ViewGroup`的`dispatchTouchEveent()`有些地方会调用`super.dispatchTouchEveent()`，而`ViewGroup`的父类就是`View`，接下来我们看一下`View.dispatchTouchEveent()`方法：
```java
/**
 * Pass the touch screen motion event down to the target view, or this
 * view if it is the target.
 *
 * @param event The motion event to be dispatched.
 * @return True if the event was handled by the view, false otherwise.
 */
public boolean dispatchTouchEvent(MotionEvent event) {
	boolean result = false;
	// 调试用
	if (mInputEventConsistencyVerifier != null) {
		mInputEventConsistencyVerifier.onTouchEvent(event, 0);
	}

	final int actionMasked = event.getActionMasked();
	if (actionMasked == MotionEvent.ACTION_DOWN) {
		// Defensive cleanup for new gesture
		stopNestedScroll();
	}

	// 判断该`View`是否被其它`View`遮盖住。
	if (onFilterTouchEventForSecurity(event)) {
		//noinspection SimplifiableIfStatement
		ListenerInfo li = mListenerInfo;
		if (li != null && li.mOnTouchListener != null
				&& (mViewFlags & ENABLED_MASK) == ENABLED
				&& li.mOnTouchListener.onTouch(this, event)) {
			// 先执行`listener`.
			result = true;
		}

		if (!result && onTouchEvent(event)) {
			// 执行`onTouchEvent()`.
			result = true;
		}
	}

	if (!result && mInputEventConsistencyVerifier != null) {
		mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
	}

	// Clean up after nested scrolls if this is the end of a gesture;
	// also cancel it if we tried an ACTION_DOWN but we didn't want the rest
	// of the gesture.
	if (actionMasked == MotionEvent.ACTION_UP ||
			actionMasked == MotionEvent.ACTION_CANCEL ||
			(actionMasked == MotionEvent.ACTION_DOWN && !result)) {
		stopNestedScroll();
	}

	return result;
}
```

通过上面的分析我们看到`View.dispatchTouchEvent()`里面会调用到`onTouchEvent()`来消耗事件。那么`onTouchEvent()`是如何处理的呢？下面我们看一下
`View.onTouchEvent()`源码： 
```java
/**
 * Implement this method to handle touch screen motion events.
 * <p>
 * If this method is used to detect click actions, it is recommended that
 * the actions be performed by implementing and calling
 * {@link #performClick()}. This will ensure consistent system behavior,
 * including:
 * <ul>
 * <li>obeying click sound preferences
 * <li>dispatching OnClickListener calls
 * <li>handling {@link AccessibilityNodeInfo#ACTION_CLICK ACTION_CLICK} when
 * accessibility features are enabled
 * </ul>
 *
 * @param event The motion event.
 * @return True if the event was handled, false otherwise.
 */
public boolean onTouchEvent(MotionEvent event) {
	final float x = event.getX();
	final float y = event.getY();
	final int viewFlags = mViewFlags;

	// 对disable按钮的处理，注释说的比较明白，一个disable但是clickable的view仍然会消耗事件,只是不响应而已。
	if ((viewFlags & ENABLED_MASK) == DISABLED) {
		if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
			setPressed(false);
		}
		// A disabled view that is clickable still consumes the touch
		// events, it just doesn't respond to them.
		return (((viewFlags & CLICKABLE) == CLICKABLE ||
				(viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
	}

	// 关于TouchDelegate,文档中是这样说的The delegate to handle touch events that are physically in this view
    // but should be handled by another view. 就是说如果两个View, View2在View1中，View1比较大，如果我们想点击
	// View1的时候，让View2去响应点击事件，这时候就需要使用TouchDelegate来设置。
	// 简单的理解就是如果这个View有自己的时间委托处理人，就交给委托人处理。
	if (mTouchDelegate != null) {
		if (mTouchDelegate.onTouchEvent(event)) {
			return true;
		}
	}

	if (((viewFlags & CLICKABLE) == CLICKABLE ||
			(viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
	    // 这个View可点击
		switch (event.getAction()) {
			case MotionEvent.ACTION_UP:
			    // 最好先看DOWN后再看MOVE最后看UP。
				// PFLAG_PREPRESSED 表示在一个可滚动的容器中,要稍后才能确定是按下还是滚动.
                // PFLAG_PRESSED 表示不是在一个可滚动的容器中,已经可以确定按下这一操作.
				boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
				if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
					// 处理点击或长按事件
					// take focus if we don't have it already and we should in
					// touch mode.
					boolean focusTaken = false;
					if (isFocusable() && isFocusableInTouchMode() && !isFocused()) 
						// 如果现在还没获取到焦点，就再获取一次焦点
						focusTaken = requestFocus();
					}

					// 在前面`DOWN`事件的时候会延迟显示`View`的`pressed`状态,用户可能在我们还没有显示按下状态效果时就不按了.我们还是得在进行实际的点击操作时,让用户看到效果。
					if (prepressed) {
						// The button is being released before we actually
						// showed it as pressed.  Make it show the pressed
						// state now (before scheduling the click) to ensure
						// the user sees it.
						setPressed(true, x, y);
				   }
					
					
					if (!mHasPerformedLongPress) {
						// 判断不是长按
						
						// This is a tap, so remove the longpress check
						removeLongPressCallback();

						// Only perform take click actions if we were in the pressed state
						if (!focusTaken) {
							// Use a Runnable and post this rather than calling
							// performClick directly. This lets other visual state
							// of the view update before click actions start.
							if (mPerformClick == null) {
								mPerformClick = new PerformClick();
							}
							// PerformClick就是个Runnable,里面执行performClick()方法。performClick()方法中怎么执行呢？我们在后面再说。
							if (!post(mPerformClick)) {
								performClick();
							}
						}
					}

					if (mUnsetPressedState == null) {
						mUnsetPressedState = new UnsetPressedState();
					}
					// 取消按下状态，UnsetPressedState也是个Runnable,里面执行setPressed(false)
					if (prepressed) {
						postDelayed(mUnsetPressedState,
								ViewConfiguration.getPressedStateDuration());
					} else if (!post(mUnsetPressedState)) {
						// If the post failed, unpress right now
						mUnsetPressedState.run();
					}

					removeTapCallback();
				}
				break;

			case MotionEvent.ACTION_DOWN:
				mHasPerformedLongPress = false;
				// performButtonActionOnTouchDown()处理鼠标右键菜单，有些View显示右键菜单就直接弹菜单.一般设备用不到鼠标，所以返回false。
				if (performButtonActionOnTouchDown(event)) {
					break;
				}

				// Walk up the hierarchy to determine if we're inside a scrolling container.
				boolean isInScrollingContainer = isInScrollingContainer();

				// For views inside a scrolling container, delay the pressed feedback for
				// a short period in case this is a scroll.
				// 就是遍历下View层级，判断这个View是不是在一个能scroll的View中。
				if (isInScrollingContainer) {
				    // 因为用户可能是点击或者是滚动，所以我们不能立马判断，先给用户设置一个要点击的事件。
					mPrivateFlags |= PFLAG_PREPRESSED;
					if (mPendingCheckForTap == null) {
						mPendingCheckForTap = new CheckForTap();
					}
					mPendingCheckForTap.x = event.getX();
					mPendingCheckForTap.y = event.getY();
					// 发送一个延时的操作，用于判断用户到底是点击还是滚动。其实就是在tapTimeout中如果用户没有滚动，那就是点击了。
					postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
				} else {
				    // 设置成点击状态
					// Not inside a scrolling container, so show the feedback right away
					setPressed(true, x, y);
					// 检查是否是长按，就是过一段时间后如果还在按住，那就是长按了。长按的时间是ViewConfiguration.getLongPressTimeout()
					// 也就是500毫秒
					checkForLongClick(0);
				}
				break;

			case MotionEvent.ACTION_CANCEL:
				// 取消按下状态，移动点击消息，移动长按消息。
				setPressed(false);
				removeTapCallback();
				removeLongPressCallback();
				break;

			case MotionEvent.ACTION_MOVE:
				drawableHotspotChanged(x, y);
				
				// Be lenient about moving outside of buttons， 检查是否移动到View外面了。
				if (!pointInView(x, y, mTouchSlop)) {
				    // 移动到区域外面去了，就要取消点击。
					// Outside button
					removeTapCallback();
					if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
						// Remove any future long press/tap checks
						removeLongPressCallback();

						setPressed(false);
					}
				}
				break;
		}

		return true;
	}

	return false;
}
```


上面讲了`Touch`事件的分发和处理，随便说一下点击事件:         
我们平时使用的时候都知道给`View`设置点击事件是`setOnClickListener()`
```java
/**
 * Register a callback to be invoked when this view is clicked. If this view is not
 * clickable, it becomes clickable.
 *
 * @param l The callback that will run
 *
 * @see #setClickable(boolean)
 */
public void setOnClickListener(OnClickListener l) {
	if (!isClickable()) {
		setClickable(true);
	}
	// `getListenerInfo()`就是判断成员变量`mListenerInfo`是否是null，不是就返回，是的话就初始化一个。
	getListenerInfo().mOnClickListener = l;
}
```

那什么地方会调用`mListenerInfo.mOnClickListener`呢？
```java
/**
 * Call this view's OnClickListener, if it is defined.  Performs all normal
 * actions associated with clicking: reporting accessibility event, playing
 * a sound, etc.
 *
 * @return True there was an assigned OnClickListener that was called, false
 *         otherwise is returned.
 */
public boolean performClick() {
	final boolean result;
	final ListenerInfo li = mListenerInfo;
	if (li != null && li.mOnClickListener != null) {
		playSoundEffect(SoundEffectConstants.CLICK);
		li.mOnClickListener.onClick(this);
		result = true;
	} else {
		result = false;
	}

	sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
	return result;
}
```

讲到这里就明白了。`onTouchEvent()`中的`ACTION_UP`中会调用`performClick()`方法。


到这里，就全部分析完了，这一块还是比较麻烦的，中间查了很多资料，有些地方自己可能也理解的不太对，如果有哪里理解的不对的地方，还请大家指出来。谢谢。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 