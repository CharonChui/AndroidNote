ListView源码分析
===


一直都想写一篇文章分析下`ListView`的实现，总是忙，一直拖到现在，快到年底了，写出来希望能帮助一些面试跳槽的人。      
当然写这篇文章也是有原因的，当时有同事在面试的时候，被对方要求当场实现一个`ListView`，同事简单的答了一些实现原理后，很显然对方不满意，经过几轮PK后，就有了不河蟹的结局。    
不欢而散，哈哈。听说后当时我想着要把`ListView`源码仔细分析后，我自己也去面试试试，可是拖到了现在也没机会了。

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
这里自然想到的就是`MVC`设计模式。     

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
			    // 如果已经有条目了就会走到这里
				if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
					sel = fillSpecific(mSelectedPosition,
							oldSel == null ? childrenTop : oldSel.getTop());
				} else if (mFirstPosition < mItemCount) {
				    // 没有选中的条目
					// TODO 一会先分析上面的fillFromTop部分，把其全部分析完后再回来分析这里。
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
	// 把childView添加到listview中，这里最后一个参数会是false
	setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);

	return child;
}
```
在`makeAndAddView()`方法里面就是通过`RecycleBin`来复用`View`，如果不存在可以复用的`View`时就创建一个新的。     
那我们就先看一下`obtainView()`方法，这个方法中的注释写的非常清楚，谷歌大神写代码就是规范，不想某些牛逼哄哄的人整天咋呼敏捷开发，一个注释也不写，那能叫敏捷？。      
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
    // getScrapView()方法会从废弃View的缓存中去取，一旦View移除了屏幕就会被加到该废弃缓存中，所以他就是当View移除屏幕了就加到该缓存中
	// 显示的时候再从该缓存中取，就这样进行了复用。
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
		// 复用的或者headerView及footerView等调用attachViewToParent方法，把之前detach的View重新attach到ViewGroup上
		attachViewToParent(child, flowDown ? -1 : 0, p);
	} else {
		p.forceAdd = false;
		if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
			p.recycledHeaderFooter = true;
		}
		// 第一次都是通过该方法来把View添加到ViewGroup中
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

到这里已经把上面`fillFromTop()`方法的部分都分析完了。  
不要忘了我们上面还留了一个TODO的部分，就是对于已经有数据的部分调用`fillSpecific()`方法，那我们就看一下他的实现:　　　　　
```java
/**
 * Put a specific item at a specific location on the screen and then build
 * up and down from there.
 *
 * @param position The reference view to use as the starting point
 * @param top Pixel offset from the top of this view to the top of the
 *        reference view.
 *
 * @return The selected view, or null if the selected view is outside the
 *         visible area.
 */
private View fillSpecific(int position, int top) {
	boolean tempIsSelected = position == mSelectedPosition;
	View temp = makeAndAddView(position, top, true, mListPadding.left, tempIsSelected);
	// Possibly changed again in fillUp if we add rows above this one.
	mFirstPosition = position;

	View above;
	View below;

	final int dividerHeight = mDividerHeight;
	if (!mStackFromBottom) {
		above = fillUp(position - 1, temp.getTop() - dividerHeight);
		// This will correct for the top of the first view not touching the top of the list
		adjustViewsUpOrDown();
		below = fillDown(position + 1, temp.getBottom() + dividerHeight);
		int childCount = getChildCount();
		if (childCount > 0) {
			correctTooHigh(childCount);
		}
	} else {
		below = fillDown(position + 1, temp.getBottom() + dividerHeight);
		// This will correct for the bottom of the last view not touching the bottom of the list
		adjustViewsUpOrDown();
		above = fillUp(position - 1, temp.getTop() - dividerHeight);
		int childCount = getChildCount();
		if (childCount > 0) {
			 correctTooLow(childCount);
		}
	}

	if (tempIsSelected) {
		return temp;
	} else if (above != null) {
		return above;
	} else {
		return below;
	}
}
```
就像注释所说的它是让指定位置的View先加载到屏幕上，然后在加载该view往上以及往下位置的其他View，他里面会调用`fillUp()`和`fillDown()`方法，又会走上面的流程，所以这里就不分析了。

好了，到这里我们就基本已经分析完了，但是感觉好像好少了点什么？　　　

这里少了不是一点，是两点:      

- 手势滑动过程中ListView的复用处理
- RecycleBin中复用的具体实现


一个个的来，先看一下手势滑动部分，这个当然是从`onTouchEvent()`方法看了，         
手势这一块不太清楚的可以看我之前的一篇文章
[Android Touch事件分发详解](https://github.com/CharonChui/AndroidNote/blob/master/SourceAnalysis/Android%20Touch%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E8%AF%A6%E8%A7%A3.md)


发现`ListView`没有重写`onTouchEvent()`方法，这也好理解，因为`GridView`也有类似的滑动功能，所以去父类`AbsListView`中看.      
我们看一下`AbsListView.onTouchEvent()`方法:　　　　　
```java
@Override
public boolean onTouchEvent(MotionEvent ev) {
	if (!isEnabled()) {
		// A disabled view that is clickable still consumes the touch
		// events, it just doesn't respond to them.
		return isClickable() || isLongClickable();
	}

	if (mPositionScroller != null) {
		mPositionScroller.stop();
	}

	if (mIsDetaching || !isAttachedToWindow()) {
		// Something isn't right.
		// Since we rely on being attached to get data set change notifications,
		// don't risk doing anything where we might try to resync and find things
		// in a bogus state.
		return false;
	}

	startNestedScroll(SCROLL_AXIS_VERTICAL);

	if (mFastScroll != null) {
		boolean intercepted = mFastScroll.onTouchEvent(ev);
		if (intercepted) {
			return true;
		}
	}

	initVelocityTrackerIfNotExists();
	final MotionEvent vtev = MotionEvent.obtain(ev);

	final int actionMasked = ev.getActionMasked();
	if (actionMasked == MotionEvent.ACTION_DOWN) {
		mNestedYOffset = 0;
	}
	vtev.offsetLocation(0, mNestedYOffset);
	switch (actionMasked) {
		case MotionEvent.ACTION_DOWN: {
			onTouchDown(ev);
			break;
		}

		case MotionEvent.ACTION_MOVE: {
		    // 具体移动的部分就在这里了
			onTouchMove(ev, vtev);
			break;
		}

		case MotionEvent.ACTION_UP: {
			onTouchUp(ev);
			break;
		}

		case MotionEvent.ACTION_CANCEL: {
			onTouchCancel();
			break;
		}

		case MotionEvent.ACTION_POINTER_UP: {
			onSecondaryPointerUp(ev);
			final int x = mMotionX;
			final int y = mMotionY;
			final int motionPosition = pointToPosition(x, y);
			if (motionPosition >= 0) {
				// Remember where the motion event started
				final View child = getChildAt(motionPosition - mFirstPosition);
				mMotionViewOriginalTop = child.getTop();
				mMotionPosition = motionPosition;
			}
			mLastY = y;
			break;
		}

		case MotionEvent.ACTION_POINTER_DOWN: {
			// New pointers take over dragging duties
			final int index = ev.getActionIndex();
			final int id = ev.getPointerId(index);
			final int x = (int) ev.getX(index);
			final int y = (int) ev.getY(index);
			mMotionCorrection = 0;
			mActivePointerId = id;
			mMotionX = x;
			mMotionY = y;
			final int motionPosition = pointToPosition(x, y);
			if (motionPosition >= 0) {
				// Remember where the motion event started
				final View child = getChildAt(motionPosition - mFirstPosition);
				mMotionViewOriginalTop = child.getTop();
				mMotionPosition = motionPosition;
			}
			mLastY = y;
			break;
		}
	}

	if (mVelocityTracker != null) {
		mVelocityTracker.addMovement(vtev);
	}
	vtev.recycle();
	return true;
}
```

接着看一下`onTouchMove()`方法:      
```java
private void onTouchMove(MotionEvent ev, MotionEvent vtev) {
	int pointerIndex = ev.findPointerIndex(mActivePointerId);
	if (pointerIndex == -1) {
		pointerIndex = 0;
		mActivePointerId = ev.getPointerId(pointerIndex);
	}

	if (mDataChanged) {
		// Re-sync everything if data has been changed
		// since the scroll operation can query the adapter.
		layoutChildren();
	}

	final int y = (int) ev.getY(pointerIndex);

	switch (mTouchMode) {
		case TOUCH_MODE_DOWN:
		case TOUCH_MODE_TAP:
		case TOUCH_MODE_DONE_WAITING:
			// Check if we have moved far enough that it looks more like a
			// scroll than a tap. If so, we'll enter scrolling mode.
			if (startScrollIfNeeded((int) ev.getX(pointerIndex), y, vtev)) {
				break;
			}
			// Otherwise, check containment within list bounds. If we're
			// outside bounds, cancel any active presses.
			final View motionView = getChildAt(mMotionPosition - mFirstPosition);
			final float x = ev.getX(pointerIndex);
			if (!pointInView(x, y, mTouchSlop)) {
				setPressed(false);
				if (motionView != null) {
					motionView.setPressed(false);
				}
				removeCallbacks(mTouchMode == TOUCH_MODE_DOWN ?
						mPendingCheckForTap : mPendingCheckForLongPress);
				mTouchMode = TOUCH_MODE_DONE_WAITING;
				updateSelectorState();
			} else if (motionView != null) {
				// Still within bounds, update the hotspot.
				final float[] point = mTmpPoint;
				point[0] = x;
				point[1] = y;
				transformPointToViewLocal(point, motionView);
				motionView.drawableHotspotChanged(point[0], point[1]);
			}
			break;

		case TOUCH_MODE_SCROLL:
		case TOUCH_MODE_OVERSCROLL:
			// 滑动部分的处理，传入当前位置的x,y坐标
			scrollIfNeeded((int) ev.getX(pointerIndex), y, vtev);
			break;
	}
}
```

再看一下`scrollIfNeeded()`方法的实现:　　　　
```java
private void scrollIfNeeded(int x, int y, MotionEvent vtev) {
	int rawDeltaY = y - mMotionY;
	int scrollOffsetCorrection = 0;
	int scrollConsumedCorrection = 0;
	if (mLastY == Integer.MIN_VALUE) {
		rawDeltaY -= mMotionCorrection;
	}
	if (dispatchNestedPreScroll(0, mLastY != Integer.MIN_VALUE ? mLastY - y : -rawDeltaY,
			mScrollConsumed, mScrollOffset)) {
		rawDeltaY += mScrollConsumed[1];
		scrollOffsetCorrection = -mScrollOffset[1];
		scrollConsumedCorrection = mScrollConsumed[1];
		if (vtev != null) {
			vtev.offsetLocation(0, mScrollOffset[1]);
			mNestedYOffset += mScrollOffset[1];
		}
	}
	final int deltaY = rawDeltaY;
	int incrementalDeltaY =
			mLastY != Integer.MIN_VALUE ? y - mLastY + scrollConsumedCorrection : deltaY;
	int lastYCorrection = 0;

	if (mTouchMode == TOUCH_MODE_SCROLL) {
		if (PROFILE_SCROLLING) {
			if (!mScrollProfilingStarted) {
				Debug.startMethodTracing("AbsListViewScroll");
				mScrollProfilingStarted = true;
			}
		}

		if (mScrollStrictSpan == null) {
			// If it's non-null, we're already in a scroll.
			mScrollStrictSpan = StrictMode.enterCriticalSpan("AbsListView-scroll");
		}

		if (y != mLastY) {
			// 移动了
			// We may be here after stopping a fling and continuing to scroll.
			// If so, we haven't disallowed intercepting touch events yet.
			// Make sure that we do so in case we're in a parent that can intercept.
			if ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) == 0 &&
					Math.abs(rawDeltaY) > mTouchSlop) {
				final ViewParent parent = getParent();
				if (parent != null) {
					parent.requestDisallowInterceptTouchEvent(true);
				}
			}

			final int motionIndex;
			if (mMotionPosition >= 0) {
				motionIndex = mMotionPosition - mFirstPosition;
			} else {
				// If we don't have a motion position that we can reliably track,
				// pick something in the middle to make a best guess at things below.
				motionIndex = getChildCount() / 2;
			}

			int motionViewPrevTop = 0;
			View motionView = this.getChildAt(motionIndex);
			if (motionView != null) {
				motionViewPrevTop = motionView.getTop();
			}

			// No need to do all this work if we're not going to move anyway
			boolean atEdge = false;
			if (incrementalDeltaY != 0) {
			    // 移动的过程中不断调用
				atEdge = trackMotionScroll(deltaY, incrementalDeltaY);
			}

			// Check to see if we have bumped into the scroll limit
			motionView = this.getChildAt(motionIndex);
			if (motionView != null) {
				// Check if the top of the motion view is where it is
				// supposed to be
				final int motionViewRealTop = motionView.getTop();
				if (atEdge) {
					// Apply overscroll

					int overscroll = -incrementalDeltaY -
							(motionViewRealTop - motionViewPrevTop);
					if (dispatchNestedScroll(0, overscroll - incrementalDeltaY, 0, overscroll,
							mScrollOffset)) {
						lastYCorrection -= mScrollOffset[1];
						if (vtev != null) {
							vtev.offsetLocation(0, mScrollOffset[1]);
							mNestedYOffset += mScrollOffset[1];
						}
					} else {
						final boolean atOverscrollEdge = overScrollBy(0, overscroll,
								0, mScrollY, 0, 0, 0, mOverscrollDistance, true);

						if (atOverscrollEdge && mVelocityTracker != null) {
							// Don't allow overfling if we're at the edge
							mVelocityTracker.clear();
						}

						final int overscrollMode = getOverScrollMode();
						if (overscrollMode == OVER_SCROLL_ALWAYS ||
								(overscrollMode == OVER_SCROLL_IF_CONTENT_SCROLLS &&
										!contentFits())) {
							if (!atOverscrollEdge) {
								mDirection = 0; // Reset when entering overscroll.
								mTouchMode = TOUCH_MODE_OVERSCROLL;
							}
							if (incrementalDeltaY > 0) {
								mEdgeGlowTop.onPull((float) -overscroll / getHeight(),
										(float) x / getWidth());
								if (!mEdgeGlowBottom.isFinished()) {
									mEdgeGlowBottom.onRelease();
								}
								invalidate(0, 0, getWidth(),
										mEdgeGlowTop.getMaxHeight() + getPaddingTop());
							} else if (incrementalDeltaY < 0) {
								mEdgeGlowBottom.onPull((float) overscroll / getHeight(),
										1.f - (float) x / getWidth());
								if (!mEdgeGlowTop.isFinished()) {
									mEdgeGlowTop.onRelease();
								}
								invalidate(0, getHeight() - getPaddingBottom() -
										mEdgeGlowBottom.getMaxHeight(), getWidth(),
										getHeight());
							}
						}
					}
				}
				mMotionY = y + lastYCorrection + scrollOffsetCorrection;
			}
			mLastY = y + lastYCorrection + scrollOffsetCorrection;
		}
	} else if (mTouchMode == TOUCH_MODE_OVERSCROLL) {
	    // 灰起了.
		if (y != mLastY) {
			final int oldScroll = mScrollY;
			final int newScroll = oldScroll - incrementalDeltaY;
			int newDirection = y > mLastY ? 1 : -1;

			if (mDirection == 0) {
				mDirection = newDirection;
			}

			int overScrollDistance = -incrementalDeltaY;
			if ((newScroll < 0 && oldScroll >= 0) || (newScroll > 0 && oldScroll <= 0)) {
				overScrollDistance = -oldScroll;
				incrementalDeltaY += overScrollDistance;
			} else {
				incrementalDeltaY = 0;
			}

			if (overScrollDistance != 0) {
				overScrollBy(0, overScrollDistance, 0, mScrollY, 0, 0,
						0, mOverscrollDistance, true);
				final int overscrollMode = getOverScrollMode();
				if (overscrollMode == OVER_SCROLL_ALWAYS ||
						(overscrollMode == OVER_SCROLL_IF_CONTENT_SCROLLS &&
								!contentFits())) {
					if (rawDeltaY > 0) {
						mEdgeGlowTop.onPull((float) overScrollDistance / getHeight(),
								(float) x / getWidth());
						if (!mEdgeGlowBottom.isFinished()) {
							mEdgeGlowBottom.onRelease();
						}
						invalidate(0, 0, getWidth(),
								mEdgeGlowTop.getMaxHeight() + getPaddingTop());
					} else if (rawDeltaY < 0) {
						mEdgeGlowBottom.onPull((float) overScrollDistance / getHeight(),
								1.f - (float) x / getWidth());
						if (!mEdgeGlowTop.isFinished()) {
							mEdgeGlowTop.onRelease();
						}
						invalidate(0, getHeight() - getPaddingBottom() -
								mEdgeGlowBottom.getMaxHeight(), getWidth(),
								getHeight());
					}
				}
			}

			if (incrementalDeltaY != 0) {
				// Coming back to 'real' list scrolling
				if (mScrollY != 0) {
					mScrollY = 0;
					invalidateParentIfNeeded();
				}
				// 继续处理着
				trackMotionScroll(incrementalDeltaY, incrementalDeltaY);

				mTouchMode = TOUCH_MODE_SCROLL;

				// We did not scroll the full amount. Treat this essentially like the
				// start of a new touch scroll
				final int motionPosition = findClosestMotionRow(y);

				mMotionCorrection = 0;
				View motionView = getChildAt(motionPosition - mFirstPosition);
				mMotionViewOriginalTop = motionView != null ? motionView.getTop() : 0;
				mMotionY =  y + scrollOffsetCorrection;
				mMotionPosition = motionPosition;
			}
			mLastY = y + lastYCorrection + scrollOffsetCorrection;
			mDirection = newDirection;
		}
	}
}
```

这部分代码逻辑牵扯太多，有点晕呼呼的，记着看一下`trackMotionScroll()`方法:      
```java
/**
 * Track a motion scroll
 *
 * @param deltaY Amount to offset mMotionView. This is the accumulated delta since the motion
 *        began. Positive numbers mean the user's finger is moving down the screen.
 * @param incrementalDeltaY Change in deltaY from the previous event. 通过它的正负来判断是向上还是向下。
 * @return true if we're already at the beginning/end of the list and have nothing to do.
 */
boolean trackMotionScroll(int deltaY, int incrementalDeltaY) {
	final int childCount = getChildCount();
	if (childCount == 0) {
		return true;
	}

	final int firstTop = getChildAt(0).getTop();
	final int lastBottom = getChildAt(childCount - 1).getBottom();

	final Rect listPadding = mListPadding;

	// "effective padding" In this case is the amount of padding that affects
	// how much space should not be filled by items. If we don't clip to padding
	// there is no effective padding.
	int effectivePaddingTop = 0;
	int effectivePaddingBottom = 0;
	if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
		effectivePaddingTop = listPadding.top;
		effectivePaddingBottom = listPadding.bottom;
	}

	 // FIXME account for grid vertical spacing too?
	final int spaceAbove = effectivePaddingTop - firstTop;
	final int end = getHeight() - effectivePaddingBottom;
	final int spaceBelow = lastBottom - end;

	final int height = getHeight() - mPaddingBottom - mPaddingTop;
	if (deltaY < 0) {
		deltaY = Math.max(-(height - 1), deltaY);
	} else {
		deltaY = Math.min(height - 1, deltaY);
	}

	if (incrementalDeltaY < 0) {
		incrementalDeltaY = Math.max(-(height - 1), incrementalDeltaY);
	} else {
		incrementalDeltaY = Math.min(height - 1, incrementalDeltaY);
	}

	final int firstPosition = mFirstPosition;

	// Update our guesses for where the first and last views are
	if (firstPosition == 0) {
		mFirstPositionDistanceGuess = firstTop - listPadding.top;
	} else {
		mFirstPositionDistanceGuess += incrementalDeltaY;
	}
	if (firstPosition + childCount == mItemCount) {
		mLastPositionDistanceGuess = lastBottom + listPadding.bottom;
	} else {
		mLastPositionDistanceGuess += incrementalDeltaY;
	}

	final boolean cannotScrollDown = (firstPosition == 0 &&
			firstTop >= listPadding.top && incrementalDeltaY >= 0);
	final boolean cannotScrollUp = (firstPosition + childCount == mItemCount &&
			lastBottom <= getHeight() - listPadding.bottom && incrementalDeltaY <= 0);

	if (cannotScrollDown || cannotScrollUp) {
		return incrementalDeltaY != 0;
	}
	// 判断向上还是向下
	final boolean down = incrementalDeltaY < 0;

	final boolean inTouchMode = isInTouchMode();
	if (inTouchMode) {
		hideSelector();
	}

	final int headerViewsCount = getHeaderViewsCount();
	final int footerViewsStart = mItemCount - getFooterViewsCount();

	int start = 0;
	int count = 0;

	if (down) {
		int top = -incrementalDeltaY;
		if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
			top += listPadding.top;
		}
		for (int i = 0; i < childCount; i++) {
			final View child = getChildAt(i);
			if (child.getBottom() >= top) {
				break;
			} else {
			    // 如果这个View的底部位置已经小于top值了，说明这个view已经移除屏幕了，不可见了
				count++;
				int position = firstPosition + i;
				if (position >= headerViewsCount && position < footerViewsStart) {
					// The view will be rebound to new data, clear any
					// system-managed transient state.
					child.clearAccessibilityFocus();
					// 只要该view移除屏幕就把该view添加到废弃的缓存中
					mRecycler.addScrapView(child, position);
				}
			}
		}
	} else {
		int bottom = getHeight() - incrementalDeltaY;
		if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
			bottom -= listPadding.bottom;
		}
		for (int i = childCount - 1; i >= 0; i--) {
			final View child = getChildAt(i);
			if (child.getTop() <= bottom) {
				break;
			} else {
				start = i;
				count++;
				int position = firstPosition + i;
				if (position >= headerViewsCount && position < footerViewsStart) {
					// The view will be rebound to new data, clear any
					// system-managed transient state.
					child.clearAccessibilityFocus();
					mRecycler.addScrapView(child, position);
				}
			}
		}
	}

	mMotionViewNewTop = mMotionViewOriginalTop + deltaY;

	mBlockLayoutRequests = true;

	// count记录了当前已经移除屏幕的view个数
	if (count > 0) {
	    // 把已经移除的View从ViewGroup总detach掉
		detachViewsFromParent(start, count);
		mRecycler.removeSkippedScrap();
	}

	// invalidate before moving the children to avoid unnecessary invalidate
	// calls to bubble up from the children all the way to the top
	if (!awakenScrollBars()) {
	   invalidate();
	}

	// 这个方法让所有的子view都按照这个参数的距离大小进行改变，这样就实现了移动也就是滑动的功能。
	offsetChildrenTopAndBottom(incrementalDeltaY);

	if (down) {
		mFirstPosition += count;
	}

	final int absIncrementalDeltaY = Math.abs(incrementalDeltaY);
	if (spaceAbove < absIncrementalDeltaY || spaceBelow < absIncrementalDeltaY) {
	    // 第一个View的顶部移入了屏幕或者最后一个View的底部移入了屏幕
		fillGap(down);
	}

	if (!inTouchMode && mSelectedPosition != INVALID_POSITION) {
		final int childIndex = mSelectedPosition - mFirstPosition;
		if (childIndex >= 0 && childIndex < getChildCount()) {
			positionSelector(mSelectedPosition, getChildAt(childIndex));
		}
	} else if (mSelectorPosition != INVALID_POSITION) {
		final int childIndex = mSelectorPosition - mFirstPosition;
		if (childIndex >= 0 && childIndex < getChildCount()) {
			positionSelector(INVALID_POSITION, getChildAt(childIndex));
		}
	} else {
		mSelectorRect.setEmpty();
	}

	mBlockLayoutRequests = false;

	invokeOnItemScrollListener();

	return false;
}
```

继续看一下`offsetChildrenTopAndBottom()`方法的，在`AbsListView`中没有重写该方法，具体要看其父类`ViewGroup`中的实现:       `
```java
/**
 * Offset the vertical location of all children of this view by the specified number of pixels.
 *
 * @param offset the number of pixels to offset
 *
 * @hide
 */
public void offsetChildrenTopAndBottom(int offset) {
	final int count = mChildrenCount;
	final View[] children = mChildren;
	boolean invalidate = false;

	for (int i = 0; i < count; i++) {
	    // 把所有的view都移动指定的距离
		final View v = children[i];
		v.mTop += offset;
		v.mBottom += offset;
		if (v.mRenderNode != null) {
			invalidate = true;
			v.mRenderNode.offsetTopAndBottom(offset);
		}
	}

	if (invalidate) {
		invalidateViewProperty(false, false);
	}
	notifySubtreeAccessibilityStateChangedIfNeeded();
}
```

再看一下`fillGap()`方法的实现:     
```java
/**
 * Fills the gap left open by a touch-scroll. During a touch scroll, children that
 * remain on screen are shifted and the other ones are discarded. The role of this
 * method is to fill the gap thus created by performing a partial layout in the
 * empty space.
 *
 * @param down true if the scroll is going down, false if it is going up
 */
abstract void fillGap(boolean down);
```
发现在`AbsListView`中该方法是抽象的，所以我们要再`ListView`中找一下他的具体实现类:       
```java
    /**
     * {@inheritDoc}
     */
    @Override
    void fillGap(boolean down) {
        final int count = getChildCount();
        if (down) {
            int paddingTop = 0;
            if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
                paddingTop = getListPaddingTop();
            }
            final int startOffset = count > 0 ? getChildAt(count - 1).getBottom() + mDividerHeight :
                    paddingTop;
            fillDown(mFirstPosition + count, startOffset);
            correctTooHigh(getChildCount());
        } else {
            int paddingBottom = 0;
            if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
                paddingBottom = getListPaddingBottom();
            }
            final int startOffset = count > 0 ? getChildAt(0).getTop() - mDividerHeight :
                    getHeight() - paddingBottom;
            fillUp(mFirstPosition - 1, startOffset);
            correctTooLow(getChildCount());
        }
    }
```
可以看到他会分别根据向下滑动还是向上滑动去调用`fillDown`和`fillUp`两个方法，这两个方法之前我们都分析过了，就不再继续看了。

到这里基本就把滑动部分的处理都看完了。
还剩最后一个问题就是`RecycleBin`的实现,他是`AbsListView`中的一个内部类。
                 
直接上代码了，他的注释也说的非常清楚，我就把不好理解的地方简单说一下:    
```java
/**
 * The RecycleBin facilitates reuse of views across layouts. The RecycleBin has two levels of
 * storage: ActiveViews and ScrapViews. ActiveViews are those views which were onscreen at the
 * start of a layout. By construction, they are displaying current information. At the end of
 * layout, all views in ActiveViews are demoted to ScrapViews. ScrapViews are old views that
 * could potentially be used by the adapter to avoid allocating views unnecessarily.
 *
 * @see android.widget.AbsListView#setRecyclerListener(android.widget.AbsListView.RecyclerListener)
 * @see android.widget.AbsListView.RecyclerListener
 */
class RecycleBin {
	private RecyclerListener mRecyclerListener;

	/**
	 * The position of the first view stored in mActiveViews.
	 */
	private int mFirstActivePosition;

	/**
	 * Views that were on screen at the start of layout. This array is populated at the start of
	 * layout, and at the end of layout all view in mActiveViews are moved to mScrapViews.
	 * Views in mActiveViews represent a contiguous range of Views, with position of the first
	 * view store in mFirstActivePosition.
	 */
	private View[] mActiveViews = new View[0];

	/**
	 * 为什么是个数组呢？因为ListView会有多种不同的ViewType啊。
	 * Unsorted views that can be used by the adapter as a convert view.
	 */
	private ArrayList<View>[] mScrapViews;

	private int mViewTypeCount;
	// mScrapViews数组中的第一个元素，方便在ViewType是1的时候使用
	private ArrayList<View> mCurrentScrap;

	private ArrayList<View> mSkippedScrap;

	private SparseArray<View> mTransientStateViews;
	private LongSparseArray<View> mTransientStateViewsById;

	public void setViewTypeCount(int viewTypeCount) {
		if (viewTypeCount < 1) {
			throw new IllegalArgumentException("Can't have a viewTypeCount < 1");
		}
		// 根据ViewType的数量来创建，因为ViewType可以有多中类型，每种不同类型的View肯定要分开单独进行缓存和复用的。
		//noinspection unchecked
		ArrayList<View>[] scrapViews = new ArrayList[viewTypeCount];
		for (int i = 0; i < viewTypeCount; i++) {
			scrapViews[i] = new ArrayList<View>();
		}
		mViewTypeCount = viewTypeCount;
		mCurrentScrap = scrapViews[0];
		mScrapViews = scrapViews;
	}

	// 方法名说明了一切
	public void markChildrenDirty() {
		if (mViewTypeCount == 1) {
			final ArrayList<View> scrap = mCurrentScrap;
			final int scrapCount = scrap.size();
			for (int i = 0; i < scrapCount; i++) {
				scrap.get(i).forceLayout();
			}
		} else {
			final int typeCount = mViewTypeCount;
			for (int i = 0; i < typeCount; i++) {
				final ArrayList<View> scrap = mScrapViews[i];
				final int scrapCount = scrap.size();
				for (int j = 0; j < scrapCount; j++) {
					scrap.get(j).forceLayout();
				}
			}
		}
		if (mTransientStateViews != null) {
			final int count = mTransientStateViews.size();
			for (int i = 0; i < count; i++) {
				mTransientStateViews.valueAt(i).forceLayout();
			}
		}
		if (mTransientStateViewsById != null) {
			final int count = mTransientStateViewsById.size();
			for (int i = 0; i < count; i++) {
				mTransientStateViewsById.valueAt(i).forceLayout();
			}
		}
	}

	public boolean shouldRecycleViewType(int viewType) {
		return viewType >= 0;
	}

	/**
	 * Clears the scrap heap.
	 */
	void clear() {
		if (mViewTypeCount == 1) {
			final ArrayList<View> scrap = mCurrentScrap;
			clearScrap(scrap);
		} else {
			final int typeCount = mViewTypeCount;
			for (int i = 0; i < typeCount; i++) {
				final ArrayList<View> scrap = mScrapViews[i];
				clearScrap(scrap);
			}
		}

		clearTransientStateViews();
	}

	/**
	 * Fill ActiveViews with all of the children of the AbsListView.
	 * 将View存储到mActiveViews数组中
	 * @param childCount The minimum number of views mActiveViews should hold
	 * @param firstActivePosition The position of the first view that will be stored in
	 *        mActiveViews
	 */
	void fillActiveViews(int childCount, int firstActivePosition) {
		if (mActiveViews.length < childCount) {
			mActiveViews = new View[childCount];
		}
		mFirstActivePosition = firstActivePosition;

		//noinspection MismatchedReadAndWriteOfArray
		final View[] activeViews = mActiveViews;
		for (int i = 0; i < childCount; i++) {
			View child = getChildAt(i);
			AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
			// Don't put header or footer views into the scrap heap
			if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				// Note:  We do place AdapterView.ITEM_VIEW_TYPE_IGNORE in active views.
				//        However, we will NOT place them into scrap views.
				activeViews[i] = child;
				// Remember the position so that setupChild() doesn't reset state.
				lp.scrappedFromPosition = firstActivePosition + i;
			}
		}
	}

	/**
	 * Get the view corresponding to the specified position. The view will be removed from
	 * mActiveViews if it is found.注释说了获取完之后就会从mActiveViews中移除，这是很好理解的
	 * 因为获取了就是说这个View已经显示到当前的ListView中了，肯定不能再复用他了。
	 *
	 * @param position The position to look up in mActiveViews
	 * @return The view if it is found, null otherwise
	 */
	View getActiveView(int position) {
		int index = position - mFirstActivePosition;
		final View[] activeViews = mActiveViews;
		if (index >=0 && index < activeViews.length) {
			final View match = activeViews[index];
			activeViews[index] = null;
			return match;
		}
		return null;
	}

	View getTransientStateView(int position) {
		if (mAdapter != null && mAdapterHasStableIds && mTransientStateViewsById != null) {
			long id = mAdapter.getItemId(position);
			View result = mTransientStateViewsById.get(id);
			mTransientStateViewsById.remove(id);
			return result;
		}
		if (mTransientStateViews != null) {
			final int index = mTransientStateViews.indexOfKey(position);
			if (index >= 0) {
				View result = mTransientStateViews.valueAt(index);
				mTransientStateViews.removeAt(index);
				return result;
			}
		}
		return null;
	}

	/**
	 * Dumps and fully detaches any currently saved views with transient
	 * state.
	 */
	void clearTransientStateViews() {
		final SparseArray<View> viewsByPos = mTransientStateViews;
		if (viewsByPos != null) {
			final int N = viewsByPos.size();
			for (int i = 0; i < N; i++) {
				removeDetachedView(viewsByPos.valueAt(i), false);
			}
			viewsByPos.clear();
		}

		final LongSparseArray<View> viewsById = mTransientStateViewsById;
		if (viewsById != null) {
			final int N = viewsById.size();
			for (int i = 0; i < N; i++) {
				removeDetachedView(viewsById.valueAt(i), false);
			}
			viewsById.clear();
		}
	}

	/**
	 * 从废弃View的缓存中获取，只要移动出屏幕之后就都会被加到废弃View的缓存中 
	 * @return A view from the ScrapViews collection. These are unordered.
	 */
	View getScrapView(int position) {
		if (mViewTypeCount == 1) {
			return retrieveFromScrap(mCurrentScrap, position);
		} else {
			final int whichScrap = mAdapter.getItemViewType(position);
			if (whichScrap >= 0 && whichScrap < mScrapViews.length) {
				return retrieveFromScrap(mScrapViews[whichScrap], position);
			}
		}
		return null;
	}

	/**
	 * Puts a view into the list of scrap views.
	 * <p>
	 * If the list data hasn't changed or the adapter has stable IDs, views
	 * with transient state will be preserved for later retrieval.
	 *
	 * @param scrap The view to add
	 * @param position The view's position within its parent
	 */
	void addScrapView(View scrap, int position) {
		final AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
		if (lp == null) {
			return;
		}

		lp.scrappedFromPosition = position;

		// Remove but don't scrap header or footer views, or views that
		// should otherwise not be recycled.
		final int viewType = lp.viewType;
		if (!shouldRecycleViewType(viewType)) {
			return;
		}

		scrap.dispatchStartTemporaryDetach();

		// The the accessibility state of the view may change while temporary
		// detached and we do not allow detached views to fire accessibility
		// events. So we are announcing that the subtree changed giving a chance
		// to clients holding on to a view in this subtree to refresh it.
		notifyViewAccessibilityStateChangedIfNeeded(
				AccessibilityEvent.CONTENT_CHANGE_TYPE_SUBTREE);
		
		// 对一些瞬态View的处理
		// Don't scrap views that have transient state.
		final boolean scrapHasTransientState = scrap.hasTransientState();
		if (scrapHasTransientState) {
			if (mAdapter != null && mAdapterHasStableIds) {
				// If the adapter has stable IDs, we can reuse the view for
				// the same data.
				if (mTransientStateViewsById == null) {
					mTransientStateViewsById = new LongSparseArray<View>();
				}
				mTransientStateViewsById.put(lp.itemId, scrap);
			} else if (!mDataChanged) {
				// If the data hasn't changed, we can reuse the views at
				// their old positions.
				if (mTransientStateViews == null) {
					mTransientStateViews = new SparseArray<View>();
				}
				mTransientStateViews.put(position, scrap);
			} else {
				// Otherwise, we'll have to remove the view and start over.
				if (mSkippedScrap == null) {
					mSkippedScrap = new ArrayList<View>();
				}
				mSkippedScrap.add(scrap);
			}
		} else {
			if (mViewTypeCount == 1) {
				mCurrentScrap.add(scrap);
			} else {
				mScrapViews[viewType].add(scrap);
			}

			if (mRecyclerListener != null) {
				mRecyclerListener.onMovedToScrapHeap(scrap);
			}
		}
	}

	/**
	 * Finish the removal of any views that skipped the scrap heap.
	 */
	void removeSkippedScrap() {
		if (mSkippedScrap == null) {
			return;
		}
		final int count = mSkippedScrap.size();
		for (int i = 0; i < count; i++) {
			removeDetachedView(mSkippedScrap.get(i), false);
		}
		mSkippedScrap.clear();
	}

	/**
	 * Move all views remaining in mActiveViews to mScrapViews.
	 */
	void scrapActiveViews() {
		final View[] activeViews = mActiveViews;
		final boolean hasListener = mRecyclerListener != null;
		final boolean multipleScraps = mViewTypeCount > 1;

		ArrayList<View> scrapViews = mCurrentScrap;
		final int count = activeViews.length;
		for (int i = count - 1; i >= 0; i--) {
			final View victim = activeViews[i];
			if (victim != null) {
				final AbsListView.LayoutParams lp
						= (AbsListView.LayoutParams) victim.getLayoutParams();
				final int whichScrap = lp.viewType;

				activeViews[i] = null;

				if (victim.hasTransientState()) {
					// Store views with transient state for later use.
					victim.dispatchStartTemporaryDetach();

					if (mAdapter != null && mAdapterHasStableIds) {
						if (mTransientStateViewsById == null) {
							mTransientStateViewsById = new LongSparseArray<View>();
						}
						long id = mAdapter.getItemId(mFirstActivePosition + i);
						mTransientStateViewsById.put(id, victim);
					} else if (!mDataChanged) {
						if (mTransientStateViews == null) {
							mTransientStateViews = new SparseArray<View>();
						}
						mTransientStateViews.put(mFirstActivePosition + i, victim);
					} else if (whichScrap != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
						// The data has changed, we can't keep this view.
						removeDetachedView(victim, false);
					}
				} else if (!shouldRecycleViewType(whichScrap)) {
					// Discard non-recyclable views except headers/footers.
					if (whichScrap != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
						removeDetachedView(victim, false);
					}
				} else {
					// Store everything else on the appropriate scrap heap.
					if (multipleScraps) {
						scrapViews = mScrapViews[whichScrap];
					}

					victim.dispatchStartTemporaryDetach();
					lp.scrappedFromPosition = mFirstActivePosition + i;
					scrapViews.add(victim);

					if (hasListener) {
						mRecyclerListener.onMovedToScrapHeap(victim);
					}
				}
			}
		}

		pruneScrapViews();
	}

	/**
	 * Makes sure that the size of mScrapViews does not exceed the size of
	 * mActiveViews, which can happen if an adapter does not recycle its
	 * views. Removes cached transient state views that no longer have
	 * transient state.
	 */
	private void pruneScrapViews() {
		final int maxViews = mActiveViews.length;
		final int viewTypeCount = mViewTypeCount;
		final ArrayList<View>[] scrapViews = mScrapViews;
		for (int i = 0; i < viewTypeCount; ++i) {
			final ArrayList<View> scrapPile = scrapViews[i];
			int size = scrapPile.size();
			final int extras = size - maxViews;
			size--;
			for (int j = 0; j < extras; j++) {
				removeDetachedView(scrapPile.remove(size--), false);
			}
		}

		final SparseArray<View> transViewsByPos = mTransientStateViews;
		if (transViewsByPos != null) {
			for (int i = 0; i < transViewsByPos.size(); i++) {
				final View v = transViewsByPos.valueAt(i);
				if (!v.hasTransientState()) {
					removeDetachedView(v, false);
					transViewsByPos.removeAt(i);
					i--;
				}
			}
		}

		final LongSparseArray<View> transViewsById = mTransientStateViewsById;
		if (transViewsById != null) {
			for (int i = 0; i < transViewsById.size(); i++) {
				final View v = transViewsById.valueAt(i);
				if (!v.hasTransientState()) {
					removeDetachedView(v, false);
					transViewsById.removeAt(i);
					i--;
				}
			}
		}
	}

	/**
	 * Puts all views in the scrap heap into the supplied list.
	 */
	void reclaimScrapViews(List<View> views) {
		if (mViewTypeCount == 1) {
			views.addAll(mCurrentScrap);
		} else {
			final int viewTypeCount = mViewTypeCount;
			final ArrayList<View>[] scrapViews = mScrapViews;
			for (int i = 0; i < viewTypeCount; ++i) {
				final ArrayList<View> scrapPile = scrapViews[i];
				views.addAll(scrapPile);
			}
		}
	}

	/**
	 * Updates the cache color hint of all known views.
	 *
	 * @param color The new cache color hint.
	 */
	void setCacheColorHint(int color) {
		if (mViewTypeCount == 1) {
			final ArrayList<View> scrap = mCurrentScrap;
			final int scrapCount = scrap.size();
			for (int i = 0; i < scrapCount; i++) {
				scrap.get(i).setDrawingCacheBackgroundColor(color);
			}
		} else {
			final int typeCount = mViewTypeCount;
			for (int i = 0; i < typeCount; i++) {
				final ArrayList<View> scrap = mScrapViews[i];
				final int scrapCount = scrap.size();
				for (int j = 0; j < scrapCount; j++) {
					scrap.get(j).setDrawingCacheBackgroundColor(color);
				}
			}
		}
		// Just in case this is called during a layout pass
		final View[] activeViews = mActiveViews;
		final int count = activeViews.length;
		for (int i = 0; i < count; ++i) {
			final View victim = activeViews[i];
			if (victim != null) {
				victim.setDrawingCacheBackgroundColor(color);
			}
		}
	}

	private View retrieveFromScrap(ArrayList<View> scrapViews, int position) {
		final int size = scrapViews.size();
		if (size > 0) {
			// See if we still have a view for this position or ID.
			for (int i = 0; i < size; i++) {
				final View view = scrapViews.get(i);
				final AbsListView.LayoutParams params =
						(AbsListView.LayoutParams) view.getLayoutParams();

				if (mAdapterHasStableIds) {
					final long id = mAdapter.getItemId(position);
					if (id == params.itemId) {
						return scrapViews.remove(i);
					}
				} else if (params.scrappedFromPosition == position) {
					final View scrap = scrapViews.remove(i);
					clearAccessibilityFromScrap(scrap);
					return scrap;
				}
			}
			final View scrap = scrapViews.remove(size - 1);
			clearAccessibilityFromScrap(scrap);
			return scrap;
		} else {
			return null;
		}
	}

	private void clearScrap(final ArrayList<View> scrap) {
		final int scrapCount = scrap.size();
		for (int j = 0; j < scrapCount; j++) {
			removeDetachedView(scrap.remove(scrapCount - 1 - j), false);
		}
	}

	private void clearAccessibilityFromScrap(View view) {
		view.clearAccessibilityFocus();
		view.setAccessibilityDelegate(null);
	}

	private void removeDetachedView(View child, boolean animate) {
		child.setAccessibilityDelegate(null);
		AbsListView.this.removeDetachedView(child, animate);
	}
}
```

到此为止。


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
