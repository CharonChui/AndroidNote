ListView源码分析
===


一直都想写一篇文章分析下`ListView`的实现，总是忙，一直拖到现在，快到年底了，写出来希望能帮助一些面试跳槽的人。      
当然写这篇文章也是有原因的，当时有同事在面试的时候，被对方要求当场实现一个`ListView`，同事简单的答了一些实现原理后，很显然对方不满意，经过几轮PK后，就有了不河蟹的结局。    
不欢而散，哈哈。听说后当时我想着要把`ListView`源码仔细分析后，我自己也去面试试试，可以拖到了现在也没机会了。

`GridView`与`ListView`都继承自`AbsListView`，他俩的实现也比较类似，这里就不说`GridView`了。只说一下`ListView`。
首先看一下`ListView`的文档:     
```java
/*
 * Implementation Notes:
 *
 * Some terminology:
 *
 *     index    - index of the items that are currently visible
 *     position - index of the items in the cursor
 */


/**
 * A view that shows items in a vertically scrolling list. The items
 * come from the {@link ListAdapter} associated with this view.
 */
```

然后再说一下`ListView`的继承关系`ListView extends AbsListView extends AdapterView<ListAdapter> extends ViewGroup extends View`。
`ListView`与其他空间不一样，不是拖到界面上就能用，而是要通过设置适配器来添加条目。这就是`AdapterView`的主要功能。既然通过适配器来操作数据。
这里自然想到的就是`MVC设计模式。
- `Data`     - `M`
- `ListView` - `V`
- `Adapter`  - `C`

到这里我们写罗列一下我们将要分析的内容:      

- `ListView`与`Adapter`间的调用。
- `ListView`上下滑动时操作及缓存。 

我们平时使用`ListView`的时候都是调用`setAdapter()`方法，所以我们这里就从该方法入手:     
```java
    /**
     * Sets the data behind this ListView.
     *
     * The adapter passed to this method may be wrapped by a {@link WrapperListAdapter},
     * depending on the ListView features currently in use. For instance, adding
     * headers and/or footers will cause the adapter to be wrapped.
     *
     * @param adapter The ListAdapter which is responsible for maintaining the
     *        data backing this list and for producing a view to represent an
     *        item in that data set.
     *
     * @see #getAdapter() 
     */
    @Override
    public void setAdapter(ListAdapter adapter) {
        if (mAdapter != null && mDataSetObserver != null) {
		    // 取消之前注册过的Adapter
            mAdapter.unregisterDataSetObserver(mDataSetObserver);
        }
        // 清空所有的数据
        resetList();	
        // The data set used to store unused views that should be reused during the next layout to avoid creating new ones.
        // 从注释中可以很明显的看出`View`的复用是通过`RecycleBin`类来实现的。它就决定了为什么`ListView`可以显示很多数据缺不会内存溢出。
        mRecycler.clear();

        if (mHeaderViewInfos.size() > 0|| mFooterViewInfos.size() > 0) {
            // 如果有HeaderView或者FooterView就把Adapter重新封装下,这是什么设计模式？－装饰
			// mHeaderViewInfos和mFooterViewInfos分别是header view和foooter view装饰类FixedViewInfo的集合。
			// 会在addHeaderView()和addFooterView()中进行添加。
            mAdapter = new HeaderViewListAdapter(mHeaderViewInfos, mFooterViewInfos, adapter);
        } else {
            mAdapter = adapter;
        }

        mOldSelectedPosition = INVALID_POSITION;
        mOldSelectedRowId = INVALID_ROW_ID;

        // AbsListView#setAdapter will update choice mode states.
        super.setAdapter(adapter);

        if (mAdapter != null) {
            mAreAllItemsSelectable = mAdapter.areAllItemsEnabled();
            mOldItemCount = mItemCount;
            mItemCount = mAdapter.getCount();
            checkFocus();

            mDataSetObserver = new AdapterDataSetObserver();
            mAdapter.registerDataSetObserver(mDataSetObserver); 
            // 将viewtypecount设置给RecycleBin
            mRecycler.setViewTypeCount(mAdapter.getViewTypeCount());

            int position;
            // mStackFromBottom Indicates whether the list is stacked from the bottom edge or the top edge.
            if (mStackFromBottom) {
                position = lookForSelectablePosition(mItemCount - 1, false);
            } else {
                position = lookForSelectablePosition(0, true);
            }
            setSelectedPositionInt(position);
            setNextSelectedPositionInt(position);

            if (mItemCount == 0) {
                // Nothing selected
                checkSelectionChanged();
            }
        } else {
            mAreAllItemsSelectable = true;
            checkFocus();
            // Nothing selected
            checkSelectionChanged();
        }
		// 请求layout了，这个就很重要了啊...我们稍后看一下。
        requestLayout();
    }
	
	/**
     * The list is empty. Clear everything out.
     */
    @Override
    void resetList() {
        // The parent's resetList() will remove all views from the layout so we need to
        // cleanup the state of our footers and headers
        clearRecycledState(mHeaderViewInfos);
        clearRecycledState(mFooterViewInfos);
        // The list is empty. Clear everything out.
        super.resetList();

        mLayoutMode = LAYOUT_NORMAL;
    }
	
	private void clearRecycledState(ArrayList<FixedViewInfo> infos) {
        if (infos != null) {
            final int count = infos.size();

            for (int i = 0; i < count; i++) {
                final View child = infos.get(i).view;
                final LayoutParams p = (LayoutParams) child.getLayoutParams();
                if (p != null) {
				    // 这里p就是AbsListView.LayoutParams。对于recycledHeaderFooter在注释中是这样说的. 
					/**
					 * When this boolean is set, the view has been added to the AbsListView
					 * at least once. It is used to know whether headers/footers have already
					 * been added to the list view and whether they should be treated as
					 * recycled views or not.
					 */
                    p.recycledHeaderFooter = false;
                }
            }
        }
    }
```

我们看到`setAdapter()`方法中会调用`requestLayout()`方法。而`requestLayout()`方法会调用`onLayout()`方法，但是我们在`ListView`中找不到`onLayout()`方法。
那就去他的父类`AbsListView`中找。我们接着来看一下`AbsListView.onLayout()`方法的实现。                   
```java
/**
 * Subclasses should NOT override this method but
 *  {@link #layoutChildren()} instead.
 */
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
	super.onLayout(changed, l, t, r, b);

	mInLayout = true;

	final int childCount = getChildCount();
	if (changed) {
		for (int i = 0; i < childCount; i++) {
			getChildAt(i).forceLayout();
		}
		mRecycler.markChildrenDirty();
	}

	// 调用layoutChildren()方法。
	layoutChildren();
	mInLayout = false;

	mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;

	// TODO: Move somewhere sane. This doesn't belong in onLayout().
	if (mFastScroll != null) {
		mFastScroll.onItemCountChanged(getChildCount(), mItemCount);
	}
}
```

我们看一下`AbsListView.layoutChildren()`方法的实现:     
```java
/**
 * Subclasses must override this method to layout their children.
 */
protected void layoutChildren() {
}
```

实现是空的,文档说的很明白了，子类必须要去实现该方法，这也是应该的，因为`ListView`和`GridView`的展现是不一样的.所以我们看一下`ListView.layoutChildren()`方法的实现:      
```java
@Override
protected void layoutChildren() {
    // When set to true, calls to requestLayout() will not propagate up the parent hierarchy.
    // This is used to layout the children during a layout pass.
	final boolean blockLayoutRequests = mBlockLayoutRequests;
	if (blockLayoutRequests) {
		return;
	}

	// 开始了，置为true
	mBlockLayoutRequests = true;

	try {
		super.layoutChildren();

		invalidate();

		if (mAdapter == null) {
			resetList();
			invokeOnItemScrollListener();
			return;
		}

		final int childrenTop = mListPadding.top;
		final int childrenBottom = mBottom - mTop - mListPadding.bottom;
		final int childCount = getChildCount();

		int index = 0;
		int delta = 0;

		View sel;
		View oldSel = null;
		View oldFirst = null;
		View newSel = null;

		// Remember stuff we will need down below
		// mLayoutMode的注释为Controls how the next layout will happen，默认为LAYOUT_NORMAL
		switch (mLayoutMode) {
		case LAYOUT_SET_SELECTION:
			index = mNextSelectedPosition - mFirstPosition;
			if (index >= 0 && index < childCount) {
				newSel = getChildAt(index);
			}
			break;
		case LAYOUT_FORCE_TOP:
		case LAYOUT_FORCE_BOTTOM:
		case LAYOUT_SPECIFIC:
		case LAYOUT_SYNC:
			break;
		case LAYOUT_MOVE_SELECTION:
		default:
			// Remember the previously selected view
			index = mSelectedPosition - mFirstPosition;
			if (index >= 0 && index < childCount) {
				oldSel = getChildAt(index);
			}

			// Remember the previous first child
			oldFirst = getChildAt(0);

			if (mNextSelectedPosition >= 0) {
				delta = mNextSelectedPosition - mSelectedPosition;
			}

			// Caution: newSel might be null
			newSel = getChildAt(index + delta);
		}

		// mDataChanged变量标记Adapter中数据是否发生变化
		boolean dataChanged = mDataChanged;
		if (dataChanged) {
			handleDataChanged();
		}

		// Handle the empty set by removing all views that are visible
		// and calling it a day
		if (mItemCount == 0) {
			resetList();
			invokeOnItemScrollListener();
			return;
		} else if (mItemCount != mAdapter.getCount()) {
			throw new IllegalStateException("The content of the adapter has changed but "
					+ "ListView did not receive a notification. Make sure the content of "
					+ "your adapter is not modified from a background thread, but only from "
					+ "the UI thread. Make sure your adapter calls notifyDataSetChanged() "
					+ "when its content changes. [in ListView(" + getId() + ", " + getClass()
					+ ") with Adapter(" + mAdapter.getClass() + ")]");
		}

		setSelectedPositionInt(mNextSelectedPosition);

		AccessibilityNodeInfo accessibilityFocusLayoutRestoreNode = null;
		View accessibilityFocusLayoutRestoreView = null;
		int accessibilityFocusPosition = INVALID_POSITION;

		// Remember which child, if any, had accessibility focus. This must
		// occur before recycling any views, since that will clear
		// accessibility focus.
		final ViewRootImpl viewRootImpl = getViewRootImpl();
		if (viewRootImpl != null) {
			final View focusHost = viewRootImpl.getAccessibilityFocusedHost();
			if (focusHost != null) {
				final View focusChild = getAccessibilityFocusedChild(focusHost);
				if (focusChild != null) {
					if (!dataChanged || isDirectChildHeaderOrFooter(focusChild)
							|| focusChild.hasTransientState() || mAdapterHasStableIds) {
						// The views won't be changing, so try to maintain
						// focus on the current host and virtual view.
						accessibilityFocusLayoutRestoreView = focusHost;
						accessibilityFocusLayoutRestoreNode = viewRootImpl
								.getAccessibilityFocusedVirtualView();
					}

					// If all else fails, maintain focus at the same
					// position.
					accessibilityFocusPosition = getPositionForView(focusChild);
				}
			}
		}

		View focusLayoutRestoreDirectChild = null;
		View focusLayoutRestoreView = null;

		// Take focus back to us temporarily to avoid the eventual call to
		// clear focus when removing the focused child below from messing
		// things up when ViewAncestor assigns focus back to someone else.
		final View focusedChild = getFocusedChild();
		if (focusedChild != null) {
			// TODO: in some cases focusedChild.getParent() == null

			// We can remember the focused view to restore after re-layout
			// if the data hasn't changed, or if the focused position is a
			// header or footer.
			if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)) {
				focusLayoutRestoreDirectChild = focusedChild;
				// Remember the specific view that had focus.
				focusLayoutRestoreView = findFocus();
				if (focusLayoutRestoreView != null) {
					// Tell it we are going to mess with it.
					focusLayoutRestoreView.onStartTemporaryDetach();
				}
			}
			requestFocus();
		}

		// Pull all children into the RecycleBin.
		// These views will be reused if possible
		final int firstPosition = mFirstPosition;
		final RecycleBin recycleBin = mRecycler;
		// RecycleBin控制着整个item的复用，等最后我们再仔细分析下RecycleBin类
		if (dataChanged) {
		    // 如果数据发生变化，就把现在的View都添加到一个废弃的View进行缓存，RecycleBin.addScrapView()方法就是缓存一些废弃的View
			for (int i = 0; i < childCount; i++) {
				recycleBin.addScrapView(getChildAt(i), firstPosition+i);
			}
		} else {
		    // 调用这个方法后就会根据传入的参数来将ListView中的指定元素存储到mActiveViews数组当中。
			recycleBin.fillActiveViews(childCount, firstPosition);
		}

		// Clear out old views
		detachAllViewsFromParent();
		recycleBin.removeSkippedScrap();
		// mLayoutMode默认是LAYOUT_NORMAL
		switch (mLayoutMode) {
		case LAYOUT_SET_SELECTION:
			if (newSel != null) {
				sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
			} else {
				sel = fillFromMiddle(childrenTop, childrenBottom);
			}
			break;
		case LAYOUT_SYNC:
			sel = fillSpecific(mSyncPosition, mSpecificTop);
			break;
		case LAYOUT_FORCE_BOTTOM:
			sel = fillUp(mItemCount - 1, childrenBottom);
			adjustViewsUpOrDown();
			break;
		case LAYOUT_FORCE_TOP:
			mFirstPosition = 0;
			sel = fillFromTop(childrenTop);
			adjustViewsUpOrDown();
			break;
		case LAYOUT_SPECIFIC:
			sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
			break;
		case LAYOUT_MOVE_SELECTION:
			sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
			break;
		default:
		    // 第一次调用layoutChildren方法的时候是还没有layout的，所以这时候childCount是0
			if (childCount == 0) {
			    // mStackFromBottom的注释Indicates whether the list is stacked from the bottom edge or the top edge.
				if (!mStackFromBottom) {
				    // 默认的布局方式是从上往下的。
					final int position = lookForSelectablePosition(0, true);
					setSelectedPositionInt(position);
					// 调用fillFromTop方法。
					sel = fillFromTop(childrenTop);
				} else {
					final int position = lookForSelectablePosition(mItemCount - 1, false);
					setSelectedPositionInt(position);
					sel = fillUp(mItemCount - 1, childrenBottom);
				}
			} else {
				if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
					sel = fillSpecific(mSelectedPosition,
							oldSel == null ? childrenTop : oldSel.getTop());
				} else if (mFirstPosition < mItemCount) {
					sel = fillSpecific(mFirstPosition,
							oldFirst == null ? childrenTop : oldFirst.getTop());
				} else {
					sel = fillSpecific(0, childrenTop);
				}
			}
			break;
		}

		// Flush any cached views that did not get reused above
		recycleBin.scrapActiveViews();

		if (sel != null) {
			// The current selected item should get focus if items are
			// focusable.
			if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
				final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
						focusLayoutRestoreView != null &&
						focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
				if (!focusWasTaken) {
					// Selected item didn't take focus, but we still want to
					// make sure something else outside of the selected view
					// has focus.
					final View focused = getFocusedChild();
					if (focused != null) {
						focused.clearFocus();
					}
					positionSelector(INVALID_POSITION, sel);
				} else {
					sel.setSelected(false);
					mSelectorRect.setEmpty();
				}
			} else {
				positionSelector(INVALID_POSITION, sel);
			}
			mSelectedTop = sel.getTop();
		} else {
			final boolean inTouchMode = mTouchMode == TOUCH_MODE_TAP
					|| mTouchMode == TOUCH_MODE_DONE_WAITING;
			if (inTouchMode) {
				// If the user's finger is down, select the motion position.
				final View child = getChildAt(mMotionPosition - mFirstPosition);
				if (child != null) {
					positionSelector(mMotionPosition, child);
				}
			} else if (mSelectorPosition != INVALID_POSITION) {
				// If we had previously positioned the selector somewhere,
				// put it back there. It might not match up with the data,
				// but it's transitioning out so it's not a big deal.
				final View child = getChildAt(mSelectorPosition - mFirstPosition);
				if (child != null) {
					positionSelector(mSelectorPosition, child);
				}
			} else {
				// Otherwise, clear selection.
				mSelectedTop = 0;
				mSelectorRect.setEmpty();
			}

			// Even if there is not selected position, we may need to
			// restore focus (i.e. something focusable in touch mode).
			if (hasFocus() && focusLayoutRestoreView != null) {
				focusLayoutRestoreView.requestFocus();
			}
		}

		// Attempt to restore accessibility focus, if necessary.
		if (viewRootImpl != null) {
			final View newAccessibilityFocusedView = viewRootImpl.getAccessibilityFocusedHost();
			if (newAccessibilityFocusedView == null) {
				if (accessibilityFocusLayoutRestoreView != null
						&& accessibilityFocusLayoutRestoreView.isAttachedToWindow()) {
					final AccessibilityNodeProvider provider =
							accessibilityFocusLayoutRestoreView.getAccessibilityNodeProvider();
					if (accessibilityFocusLayoutRestoreNode != null && provider != null) {
						final int virtualViewId = AccessibilityNodeInfo.getVirtualDescendantId(
								accessibilityFocusLayoutRestoreNode.getSourceNodeId());
						provider.performAction(virtualViewId,
								AccessibilityNodeInfo.ACTION_ACCESSIBILITY_FOCUS, null);
					} else {
						accessibilityFocusLayoutRestoreView.requestAccessibilityFocus();
					}
				} else if (accessibilityFocusPosition != INVALID_POSITION) {
					// Bound the position within the visible children.
					final int position = MathUtils.constrain(
							accessibilityFocusPosition - mFirstPosition, 0,
							getChildCount() - 1);
					final View restoreView = getChildAt(position);
					if (restoreView != null) {
						restoreView.requestAccessibilityFocus();
					}
				}
			}
		}

		// Tell focus view we are done mucking with it, if it is still in
		// our view hierarchy.
		if (focusLayoutRestoreView != null
				&& focusLayoutRestoreView.getWindowToken() != null) {
			focusLayoutRestoreView.onFinishTemporaryDetach();
		}
		
		mLayoutMode = LAYOUT_NORMAL;
		mDataChanged = false;
		if (mPositionScrollAfterLayout != null) {
			post(mPositionScrollAfterLayout);
			mPositionScrollAfterLayout = null;
		}
		mNeedSync = false;
		setNextSelectedPositionInt(mSelectedPosition);

		updateScrollIndicators();

		if (mItemCount > 0) {
			checkSelectionChanged();
		}

		invokeOnItemScrollListener();
	} finally {
		if (!blockLayoutRequests) {
			mBlockLayoutRequests = false;
		}
	}
}
```

上面看到会调用`fillFromTop()`方法，我们看一下该方法的实现:       
```java
/**
 * Fills the list from top to bottom, starting with mFirstPosition
 *
 * @param nextTop The location where the top of the first item should be
 *        drawn
 *
 * @return The view that is currently selected
 */
private View fillFromTop(int nextTop) {
	mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
	mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
	if (mFirstPosition < 0) {
		mFirstPosition = 0;
	}
	// 调用fillDown()方法。
	return fillDown(mFirstPosition, nextTop);
}
```
接下来我们看一下`fillDown()`方法的实现:      
```java
/**
 * Fills the list from pos down to the end of the list view.
 *
 * @param pos The first position to put in the list
 *
 * @param nextTop The location where the top of the item associated with pos
 *        should be drawn
 *
 * @return The view that is currently selected, if it happens to be in the
 *         range that we draw.
 */
private View fillDown(int pos, int nextTop) {
	View selectedView = null;

	int end = (mBottom - mTop);
	if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
		end -= mListPadding.bottom;
	}
	// 用一个循环一直去填充，nextTop与end的比较来判断是否超过当前屏幕了，这就是为什么ListView就算有一万条最开始载入也不卡，因为他只初始化一屏啊。
	// 以后都是边滑动边显示啊，说到这里一会我们还要看一下手势部分的处理。pos和mItemCount的比较来判断当前是否已经把所有条目填充完
	while (nextTop < end && pos < mItemCount) {
		// is this the selected item?
		boolean selected = pos == mSelectedPosition;
		// 调用makeAndAddView方法
		View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

		nextTop = child.getBottom() + mDividerHeight;
		if (selected) {
			selectedView = child;
		}
		pos++;
	}

	setVisibleRangeHint(mFirstPosition, mFirstPosition + getChildCount() - 1);
	return selectedView;
}
```

接下来看一下`makeAndAddView()`方法，文档里面说的非常清楚了:　　　　　　
```java
/**
 * Obtain the view and add it to our list of children. The view can be made
 * fresh, converted from an unused view, or used as is if it was in the
 * recycle bin.
 *
 * @param position Logical position in the list
 * @param y Top or bottom edge of the view to add
 * @param flow If flow is true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @return View that was added
 */
private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
		boolean selected) {
	View child;

	if (!mDataChanged) {
		// Try to use an existing view for this position
		child = mRecycler.getActiveView(position);
		if (child != null) {
			// Found it -- we're using an existing child
			// This just needs to be positioned
			// 最后一个参数为true说明是复用的
			setupChild(child, position, y, flow, childrenLeft, selected, true);

			return child;
		}
	}

	// Make a new view for this position, or convert an unused view if possible
	// 一会我们先看一下obtainView方法，这个方法其实就是创建一个childView
	// obtainView方法中会更改mIsScrap[0]的值，以便下面setupChild()中使用。
	child = obtainView(position, mIsScrap);

	// This needs to be positioned and measured
	// 把childView添加到listview中
	setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);

	return child;
}
```
在`makeAndAddView()`方法里面就是通过`RecycleBin`来复用`View`，如果不存在可以复用的`View`时就创建一个新的。     
那我们就先看一下`obtainView()方法，这个方法中的注释写的非常清楚，谷歌大神写代码就是规范，不想某些牛逼哄哄的人整天咋呼敏捷开发，一个注释也不写，那能叫敏捷？。      
```java
/**
 * Get a view and have it show the data associated with the specified
 * position. This is called when we have already discovered that the view is
 * not available for reuse in the recycle bin. The only choices left are
 * converting an old view or making a new one.
 *
 * @param position The position to display
 * @param isScrap Array of at least 1 boolean, the first entry will become true if
 *                the returned view was taken from the scrap heap, false if otherwise.
 *
 * @return A view displaying the data associated with the specified position
 */
View obtainView(int position, boolean[] isScrap) {
	Trace.traceBegin(Trace.TRACE_TAG_VIEW, "obtainView");

	isScrap[0] = false;

	// Check whether we have a transient state view. Attempt to re-bind the
	// data and discard the view if we fail.
	final View transientView = mRecycler.getTransientStateView(position);
	if (transientView != null) {
		final LayoutParams params = (LayoutParams) transientView.getLayoutParams();

		// If the view type hasn't changed, attempt to re-bind the data.
		if (params.viewType == mAdapter.getItemViewType(position)) {
			final View updatedView = mAdapter.getView(position, transientView, this);

			// If we failed to re-bind the data, scrap the obtained view.
			if (updatedView != transientView) {
				setItemViewLayoutParams(updatedView, position);
				mRecycler.addScrapView(updatedView, position);
			}
		}

		// Scrap view implies temporary detachment.
		isScrap[0] = true;
		return transientView;
	}

	final View scrapView = mRecycler.getScrapView(position);
	// 调用Adapter.getView()方法了。并且将scrapView作为converview的参数传入。这个方法都挺熟，就不说了。
	final View child = mAdapter.getView(position, scrapView, this);
	if (scrapView != null) {
		if (child != scrapView) {
			// Failed to re-bind the data, return scrap to the heap.
			mRecycler.addScrapView(scrapView, position);
		} else {
			isScrap[0] = true;

			child.dispatchFinishTemporaryDetach();
		}
	}

	if (mCacheColorHint != 0) {
		child.setDrawingCacheBackgroundColor(mCacheColorHint);
	}

	if (child.getImportantForAccessibility() == IMPORTANT_FOR_ACCESSIBILITY_AUTO) {
		child.setImportantForAccessibility(IMPORTANT_FOR_ACCESSIBILITY_YES);
	}
	// 设置该view的layoutparams
	setItemViewLayoutParams(child, position);

	if (AccessibilityManager.getInstance(mContext).isEnabled()) {
		if (mAccessibilityDelegate == null) {
			mAccessibilityDelegate = new ListItemAccessibilityDelegate();
		}
		if (child.getAccessibilityDelegate() == null) {
			child.setAccessibilityDelegate(mAccessibilityDelegate);
		}
	}

	Trace.traceEnd(Trace.TRACE_TAG_VIEW);

	return child;
}
```

到这里我们就把`obtainView()`方法都看完了，接下来我们再看一下`setupChild()`方法:     
```java
/**
 * Add a view as a child and make sure it is measured (if necessary) and
 * positioned properly.
 *
 * @param child The view to add
 * @param position The position of this child
 * @param y The y position relative to which this view will be positioned
 * @param flowDown If true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @param recycled Has this view been pulled from the recycle bin? If so it
 *        does not need to be remeasured.
 */
private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
		boolean selected, boolean recycled) {
	Trace.traceBegin(Trace.TRACE_TAG_VIEW, "setupListItem");

	final boolean isSelected = selected && shouldShowSelector();
	final boolean updateChildSelected = isSelected != child.isSelected();
	final int mode = mTouchMode;
	final boolean isPressed = mode > TOUCH_MODE_DOWN && mode < TOUCH_MODE_SCROLL &&
			mMotionPosition == position;
	final boolean updateChildPressed = isPressed != child.isPressed();
	final boolean needToMeasure = !recycled || updateChildSelected || child.isLayoutRequested();

	// Respect layout params that are already in the view. Otherwise make some up...
	// noinspection unchecked
	AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();
	if (p == null) {
		p = (AbsListView.LayoutParams) generateDefaultLayoutParams();
	}
	// 这里就是Adapter中getItemViewType的调用处
	p.viewType = mAdapter.getItemViewType(position);

	if ((recycled && !p.forceAdd) || (p.recycledHeaderFooter &&
			p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER)) {
		// 复用的或者headerView及footerView等调用attachViewToParent方法
		attachViewToParent(child, flowDown ? -1 : 0, p);
	} else {
		p.forceAdd = false;
		if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
			p.recycledHeaderFooter = true;
		}
		// 第一次都是通过该方法来添加到ListView中的
		addViewInLayout(child, flowDown ? -1 : 0, p, true);
	}

	if (updateChildSelected) {
		child.setSelected(isSelected);
	}

	if (updateChildPressed) {
		child.setPressed(isPressed);
	}

	if (mChoiceMode != CHOICE_MODE_NONE && mCheckStates != null) {
		if (child instanceof Checkable) {
			((Checkable) child).setChecked(mCheckStates.get(position));
		} else if (getContext().getApplicationInfo().targetSdkVersion
				>= android.os.Build.VERSION_CODES.HONEYCOMB) {
			child.setActivated(mCheckStates.get(position));
		}
	}

	if (needToMeasure) {
		int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
				mListPadding.left + mListPadding.right, p.width);
		int lpHeight = p.height;
		int childHeightSpec;
		if (lpHeight > 0) {
			childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
		} else {
			childHeightSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
		}
		child.measure(childWidthSpec, childHeightSpec);
	} else {
		cleanupLayoutState(child);
	}

	final int w = child.getMeasuredWidth();
	final int h = child.getMeasuredHeight();
	final int childTop = flowDown ? y : y - h;

	if (needToMeasure) {
		final int childRight = childrenLeft + w;
		final int childBottom = childTop + h;
		child.layout(childrenLeft, childTop, childRight, childBottom);
	} else {
		child.offsetLeftAndRight(childrenLeft - child.getLeft());
		child.offsetTopAndBottom(childTop - child.getTop());
	}

	if (mCachingStarted && !child.isDrawingCacheEnabled()) {
		child.setDrawingCacheEnabled(true);
	}

	if (recycled && (((AbsListView.LayoutParams)child.getLayoutParams()).scrappedFromPosition)
			!= position) {
		child.jumpDrawablesToCurrentState();
	}

	Trace.traceEnd(Trace.TRACE_TAG_VIEW);
}
```
通过分析我们看到核心的部分就是`attachViewToParent()`方法和`addViewInLayout()`方法，这两个方法都是父类`ViewGroup`中的方法，
我们分别来看一下:     
```java
/**
 * Attaches a view to this view group. Attaching a view assigns this group as the parent,
 * sets the layout parameters and puts the view in the list of children so that
 * it can be retrieved by calling {@link #getChildAt(int)}.
 * <p>
 * This method is intended to be lightweight and makes no assumptions about whether the
 * parent or child should be redrawn. Proper use of this method will include also making
 * any appropriate {@link #requestLayout()} or {@link #invalidate()} calls.
 * For example, callers can {@link #post(Runnable) post} a {@link Runnable}
 * which performs a {@link #requestLayout()} on the next frame, after all detach/attach
 * calls are finished, causing layout to be run prior to redrawing the view hierarchy.
 * <p>
 * This method should be called only for views which were detached from their parent.
 *
 * @param child the child to attach
 * @param index the index at which the child should be attached
 * @param params the layout parameters of the child
 *
 * @see #removeDetachedView(View, boolean)
 * @see #detachAllViewsFromParent()
 * @see #detachViewFromParent(View)
 * @see #detachViewFromParent(int)
 */
protected void attachViewToParent(View child, int index, LayoutParams params) {
	child.mLayoutParams = params;

	if (index < 0) {
		index = mChildrenCount;
	}
	
	// 把该view添加到数组中，就像注释所说的为了能够让`getChildAt()`方法能获取到。
	addInArray(child, index);

	child.mParent = this;
	child.mPrivateFlags = (child.mPrivateFlags & ~PFLAG_DIRTY_MASK
					& ~PFLAG_DRAWING_CACHE_VALID)
			| PFLAG_DRAWN | PFLAG_INVALIDATED;
	this.mPrivateFlags |= PFLAG_INVALIDATED;

	if (child.hasFocus()) {
		requestChildFocus(child, child.findFocus());
	}
}
```
复用的处理看完后，我们看一下新创建的childView如何被添加到ListView上，我们看一下`addViewInLayout()`方法:       
```java
/**
 * Adds a view during layout. This is useful if in your onLayout() method,
 * you need to add more views (as does the list view for example).
 *
 * If index is negative, it means put it at the end of the list.
 *
 * @param child the view to add to the group
 * @param index the index at which the child must be added
 * @param params the layout parameters to associate with the child
 * @param preventRequestLayout if true, calling this method will not trigger a
 *        layout request on child
 * @return true if the child was added, false otherwise
 */
protected boolean addViewInLayout(View child, int index, LayoutParams params,
		boolean preventRequestLayout) {
	if (child == null) {
		throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
	}
	child.mParent = null;
	// 这个方法最终会调用addView方法添加childView
	addViewInner(child, index, params, preventRequestLayout);
	child.mPrivateFlags = (child.mPrivateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;
	return true;
}
```

到这里我们就基本把第一部分分析完了，就是`setAdapter()`方法后初始化第一屏数据的的流程。


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
