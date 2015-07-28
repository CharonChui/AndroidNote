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

        resetList();
        mRecycler.clear();

        if (mHeaderViewInfos.size() > 0|| mFooterViewInfos.size() > 0) {
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

            mRecycler.setViewTypeCount(mAdapter.getViewTypeCount());

            int position;
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










---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 