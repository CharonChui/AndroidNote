RecyclerView专题
===

### 简介

`RecyclerView`是`Android 5.0`提供的新控件，已经用了很长时间了，但是一直没有时间去仔细的梳理一下。现在项目不太紧，决定来整理下。

官方文档中是这样介绍的:     
`A flexible view for providing a limited window into a large data set.`

`RecyclerView`比`listview`更先进更灵活，对于很多的视图它就是一个容器，可以有效的重用和滚动。当数据动态变化的时候请使用它。



##### 专业术语:

- `Adapter`: `A subclass of RecyclerView.Adapter responsible for providing views that represent items in a data set.`
- `Position`: `The position of a data item within an Adapter.`
- `Index`: `The index of an attached child view as used in a call to getChildAt(int). Contrast with Position.`
- `Binding`: `The process of preparing a child view to display data corresponding to a position within the adapter.`
- `Recycle (view)`: `A view previously used to display data for a specific adapter position may be placed in a cache for later reuse to display the same type of data again later. This can drastically improve performance by skipping initial layout inflation or construction.`
- `Scrap (view)`: `A child view that has entered into a temporarily detached state during layout. Scrap views may be reused without becoming fully detached from the parent RecyclerView, either unmodified if no rebinding is required or modified by the adapter if the view was considered dirty.`
- `Dirty (view)`: `A child view that must be rebound by the adapter before being displayed.`

##### `RecyclerView`中的位置:      

`RecyclerView`在`RecyclerView.Adapter`和`RecyclerView.LayoutManager`中引进了一个抽象的额外中间层来保证在布局计算的过程中能批量的监听到数据变化。这样介绍了`LayoutManager`追踪`adapter`数据变化来计算动画的时间。因为所有的`View`绑定都是在同一时间执行，所以这样也提高了性能和避免了一些非必要的绑定。       
因为这个原因，在`RecylcerView`中有两种`position`类型相关的方法:     
- `layout position`:  在最近一次`layout`计算是`item`的位置。这是`LayoutManager`角度中的位置。   
- `adapter position`: `item`在`adapter`中的位置。这是从`Adapter`的角度中的位置。
      
这两种`position`除了在分发`adapter.notify`事件与之后计算布局更新的这段时间之内外都是相同的。  
可以通过`getLayoutPosition(),findViewHolderForLayoutPosition(int)`方法来获取最近一次布局计算的`LayoutPosition`。这些`positions`包括从最近一次布局计算的所有改变。你可以根据这些位置来方便的得到用户当前从屏幕上所看到的。例如，如果在屏幕上有一个列表，用户请求的是第五个条目，你可以通过该方法来匹配当前用户正在看的内容。
            
另一种`AdapterPosition`相关的方法是`getAdapterPosition(),findViewHolderForAdapterPosition(int)`，当及时一些数据可能没有来得及被展现到布局上时便需要获取最新的`adapter`位置可以使用这些相关的方法。例如，如果你想获取一个条目的`ViewHOlder`的`click`事件时，你应该使用`getAdapterPosition()`。需要知道这些方法在`notifyDataSetChange()`方法被调用和新布局还没有被计算之前是不能使用的。鉴于这个原因，你应该小心的去处理这些方法有可能返回`NO_POSITION`或者`null`的情况。



### 结构

- `RecyclerView.Adapter`: 创建`View`并将数据集合绑定到`View`上
- `ViewHolder`: 持有所有的用于绑定数据或者需要操作的`View`
- `LayoutManager`: 布局管理器，负责摆放视图等相关操作
- `ItemDecoration`: 负责绘制`Item`附近的分割线，通过`RecyclerView.addItemDecoration()`使用
- `ItemAnimator`: 为`Item`的操作添加动画效果，如，增删条目等，通过`RecyclerView.setItemAnimator(new DefaultItemAnimator());`使用

下图能更直观的了解:   
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/RecyclerView.png?raw=true)

##### `RecyclerView`提供这些内置布局管理器:    

- `LinearLayoutManager`: 以垂直或水平滚动列表方式显示项目。
- `GridLayoutManager`: 在网格中显示项目。
- `StaggeredGridLayoutManager`: 在分散对齐网格中显示项目。

##### `RecyclerView.ItemDecoration`是一个抽象类，可以通过重写以下三个方法，来实现Item之间的偏移量或者装饰效果:     

- `public void onDraw(Canvas c, RecyclerView parent)` 装饰的绘制在Item条目绘制之前调用，所以这有可能被Item的内容所遮挡
- `public void onDrawOver(Canvas c, RecyclerView parent)` 装饰的绘制在Item条目绘制之后调用，因此装饰将浮于Item之上
- `public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent)` 与padding或margin类似，LayoutManager在测量阶段会调用该方法，计算出每一个Item的正确尺寸并设置偏移量。


##### `ItemAnimator`触发于以下三种事件:    

- 某条数据被插入到数据集合中
- 从数据集合中移除某条数据
- 更改数据集合中的某条数据

在之前的版本中，当时据集合发生改变时通过调用`notifyDataSetChanged()`，来刷新列表，因为这样做会触发列表的重绘，所以并不会出现任何动画效果，因此需要调用一些以`notifyItem*()`作为前缀的特殊方法，比如:    

- `public final void notifyItemInserted(int position)` 向指定位置插入`Item`
- `public final void notifyItemRemoved(int position)` 移除指定位置`Item`
- `public final void notifyItemChanged(int position)` 更新指定位置`Item`



### 使用介绍:     

- 添加依赖库           
    ```
    dependencies {
        compile 'com.android.support:recyclerview-v7:23.4.0'
    }
    ```
- 示例代码        

    ```java
    public class MainActivity extends AppCompatActivity {
        private RecyclerView mRecyclerView;
        private LinearLayoutManager mLayoutManager;
        private RecyclerView.Adapter mAdapter;
    
        private String [] mDatas = {"Android","ios","jack","tony","window","mac","1234","hehe","495948", "89757", "66666"};
    
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            findView();
            initView();
        }
    
        private void findView() {
            mRecyclerView = (RecyclerView) findViewById(R.id.rv);
        }
    
        private void initView() {
            // use this setting to improve performance if you know that changes
            // in content do not change the layout size of the RecyclerView
            mRecyclerView.setHasFixedSize(true);
    
            // use a linear layout manager
            mLayoutManager = new LinearLayoutManager(this);
            mLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
            mRecyclerView.setLayoutManager(mLayoutManager);
            mAdapter = new MyAdapter(mDatas);
            mRecyclerView.setAdapter(mAdapter);
        }
    
        private class MyAdapter extends RecyclerView.Adapter<MyHolder> {
            private String[] mData;
    
            public MyAdapter(String[] data) {
                mData = data;
            }
    
            @Override
            public MyHolder onCreateViewHolder(ViewGroup parent, int viewType) {
                // create a new view
                View v = LayoutInflater.from(parent.getContext()).inflate(
                        R.layout.item, parent, false);
                MyHolder holder = new MyHolder(v);
                return holder;
            }
    
            @Override
            public void onBindViewHolder(MyHolder holder, int position) {
                holder.mTitleTv.setText(mData[position]);
            }
    
            @Override
            public int getItemCount() {
                return mData == null ? 0 : mData.length;
            }
        }
    
        static class MyHolder extends RecyclerView.ViewHolder {
            public TextView mTitleTv;
    
            public MyHolder(View itemView) {
                super(itemView);
                mTitleTv = (TextView) itemView;
            }
        }
    }
    ```
    
    `activity_main的内容如下: `     
    ```xml
    <android.support.v7.widget.RecyclerView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    ```
    
    `item的内容如下：`     
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="20dp"
        android:gravity="center_horizontal"
        android:textSize="30dp">
    
    </TextView>
    ```



### 点击事件

之前在使用`ListView`的时候，设置点击事件是非常方便的。    
```java
mListView.setOnItemClickListener();
mListView.setOnItemLongClickListener();
```
但是`RecylcerView`确没有提供类似的方法。那我们只能是自己去处理。处理的方式也有两种:       


- 通过`itemView.onClickListener()`以及`onLongClickListener()`     

    ```java
    public class MainActivity extends AppCompatActivity {
        private RecyclerView mRecyclerView;
        private LinearLayoutManager mLayoutManager;
        private MyAdapter mAdapter;
    
        private String[] mDatas = {"Android", "ios", "jack", "tony", "window", "mac", "1234", "hehe", "495948", "89757", "66666"};
    
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            findView();
            initView();
        }
    
        private void findView() {
            mRecyclerView = (RecyclerView) findViewById(R.id.rv);
        }
    
        private void initView() {
            // use this setting to improve performance if you know that changes
            // in content do not change the layout size of the RecyclerView
            mRecyclerView.setHasFixedSize(true);
    
            // use a linear layout manager
            mLayoutManager = new LinearLayoutManager(this);
            mLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
            mRecyclerView.setLayoutManager(mLayoutManager);
            mAdapter = new MyAdapter(mDatas);
            mAdapter.setOnItemClickListener(new OnItemClickListener() {
                @Override
                public void onItemClick(View view, int position) {
                    Toast.makeText(MainActivity.this, "click " + mDatas[position], Toast.LENGTH_SHORT).show();
                }
            });
            mAdapter.setOnItemLongClickListener(new OnItemLongClickListener() {
                @Override
                public void onItemLongClick(View view, int position) {
                    Toast.makeText(MainActivity.this, "long click " + mDatas[position], Toast.LENGTH_SHORT).show();
                }
            });
            mRecyclerView.setAdapter(mAdapter);
        }
    
        class MyAdapter extends RecyclerView.Adapter<MyHolder> {
            private String[] mData;
    
            public MyAdapter(String[] data) {
                mData = data;
            }
    
            @Override
            public MyHolder onCreateViewHolder(ViewGroup parent, int viewType) {
                // create a new view
                View v = LayoutInflater.from(parent.getContext()).inflate(
                        R.layout.item, parent, false);
                MyHolder holder = new MyHolder(v);
                return holder;
            }
    
            @Override
            public void onBindViewHolder(final MyHolder holder, final int position) {
                holder.mTitleTv.setText(mData[position]);
                if (mOnItemClickListener != null) {
                    holder.itemView.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            mOnItemClickListener.onItemClick(holder.itemView, position);
                        }
                    });
                }
                if (mOnItemLongClickListener != null) {
                    holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                        @Override
                        public boolean onLongClick(View v) {
                            mOnItemLongClickListener.onItemLongClick(holder.itemView, position);
                            return true;
                        }
                    });
                }
            }
    
            @Override
            public int getItemCount() {
                return mData == null ? 0 : mData.length;
            }
    
            private OnItemClickListener mOnItemClickListener;
            private OnItemLongClickListener mOnItemLongClickListener;
    
            public void setOnItemClickListener(OnItemClickListener mOnItemClickListener) {
                this.mOnItemClickListener = mOnItemClickListener;
            }
    
            public void setOnItemLongClickListener(OnItemLongClickListener mOnItemLongClickListener) {
                this.mOnItemLongClickListener = mOnItemLongClickListener;
            }
    
        }
    
        static class MyHolder extends RecyclerView.ViewHolder {
            public TextView mTitleTv;
    
            public MyHolder(View itemView) {
                super(itemView);
                mTitleTv = (TextView) itemView;
            }
        }
    
        public interface OnItemClickListener {
            void onItemClick(View view, int position);
        }
    
        public interface OnItemLongClickListener {
            void onItemLongClick(View view, int position);
        }
    
    }
    ```
- 使用`RecyclerView.OnItemTouchListener`           

    虽然没有提供现成的监听器，但是提供了一个内部接口`OnItemTouchListener`。    
    先来看看它的介绍:    
    ```
    /**
     * An OnItemTouchListener allows the application to intercept touch events in progress at the
     * view hierarchy level of the RecyclerView before those touch events are considered for
     * RecyclerView's own scrolling behavior.
     *
     * <p>This can be useful for applications that wish to implement various forms of gestural
     * manipulation of item views within the RecyclerView. OnItemTouchListeners may intercept
     * a touch interaction already in progress even if the RecyclerView is already handling that
     * gesture stream itself for the purposes of scrolling.</p>
     *
     * @see SimpleOnItemTouchListener
     */
    public static interface OnItemTouchListener {
        ...
    }
    ```

    说的和明白了，而且还让你看`SimpleOnItemTouchListener`，猜也能猜出来是一个默认的实现类。
    好了直接上代码:      
    ```java
    public class MainActivity extends AppCompatActivity {
    private RecyclerView mRecyclerView;
    private LinearLayoutManager mLayoutManager;
    private MyAdapter mAdapter;
    private String[] mDatas = {"Android", "ios", "jack", "tony", "window", "mac", "1234", "hehe", "495948", "89757", "66666"};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findView();
        initView();
    }

    private void findView() {
        mRecyclerView = (RecyclerView) findViewById(R.id.rv);
    }

    private void initView() {
        // use this setting to improve performance if you know that changes
        // in content do not change the layout size of the RecyclerView
        mRecyclerView.setHasFixedSize(true);

        // use a linear layout manager
        mLayoutManager = new LinearLayoutManager(this);
        mLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
        mRecyclerView.setLayoutManager(mLayoutManager);
        mAdapter = new MyAdapter(mDatas);
        mRecyclerView.setAdapter(mAdapter);
        mRecyclerView.addOnItemTouchListener(new RecyclerViewClickListener(this, mRecyclerView, new OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                Toast.makeText(MainActivity.this, mDatas[position], Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onItemLongClick(View view, int position) {
                Toast.makeText(MainActivity.this, "Long Click " + mDatas[position], Toast.LENGTH_SHORT).show();
            }
        }));
    }

    class RecyclerViewClickListener extends RecyclerView.SimpleOnItemTouchListener {
        private GestureDetector mGestureDetector;
        private OnItemClickListener mListener;

        public RecyclerViewClickListener(Context context, final RecyclerView recyclerView, OnItemClickListener listener) {
            mListener = listener;
            mGestureDetector = new GestureDetector(context,
                    new GestureDetector.SimpleOnGestureListener() {
                        @Override
                        public boolean onSingleTapUp(MotionEvent e) {
                            View childView = recyclerView.findChildViewUnder(e.getX(), e.getY());
                            if (childView != null && mListener != null) {
                                mListener.onItemClick(childView, recyclerView.getChildLayoutPosition(childView));
                                return true;
                            }
                            return false;
                        }

                        @Override
                        public void onLongPress(MotionEvent e) {
                            View childView = recyclerView.findChildViewUnder(e.getX(), e.getY());
                            if (childView != null && mListener != null) {
                                mListener.onItemLongClick(childView, recyclerView.getChildLayoutPosition(childView));
                            }
                        }
                    });
        }

        @Override
        public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
            if (mGestureDetector.onTouchEvent(e)) {
                return true;
            } else
                return super.onInterceptTouchEvent(rv, e);
        }

        @Override
        public void onTouchEvent(RecyclerView rv, MotionEvent e) {
            super.onTouchEvent(rv, e);
        }

        @Override
        public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {
            super.onRequestDisallowInterceptTouchEvent(disallowIntercept);
        }
    }

    interface OnItemClickListener {
        void onItemClick(View view, int position);

        void onItemLongClick(View view, int position);
    }
    ```
    上面的实现稍微有些缺陷，就是如果我手指按住某个条目一直不抬起，他也会执行`Long click`事件，这显然是不合理的，至于怎么解决，就是可以不用`GestureDetector`，自己在`DOWN`和`UP`事件中去判断处理。

### Headerview FooterView

之前在`ListView`中提供了`addHeaderView()`和`addFooterView()`等方法，但是在`RecyclerView`中并没有提供类似的方法，那我们该如何添加呢？ 也很简单，就是通过`Adapter`中去添加，利用不同的`itemViewType`，然后根据不同的类型去在`onCreateViewHOlder`中创建不同的视图，通过这种方式来达到`headerview`和`FooterView`的效果。

上一段简单的示例代码:     
```java
    public class HeaderAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private static final int TYPE_HEADER = 0;
    private static final int TYPE_ITEM = 1;
    String[] data;

    public HeaderAdapter(String[] data) {
        this.data = data;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (viewType == TYPE_ITEM) {
            //inflate your layout and pass it to view holder
            return new VHItem(null);
        } else if (viewType == TYPE_HEADER) {
            //inflate your layout and pass it to view holder
            return new VHHeader(null);
        }

        throw new RuntimeException("there is no type that matches the type " + viewType + " + make sure your using types correctly");
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        if (holder instanceof VHItem) {
            String dataItem = getItem(position);
            //cast holder to VHItem and set data
        } else if (holder instanceof VHHeader) {
            //cast holder to VHHeader and set data for header.
        }
    }

    @Override
    public int getItemCount() {
        return data.length + 1;
    }

    @Override
    public int getItemViewType(int position) {
        if (isPositionHeader(position))
            return TYPE_HEADER;

        return TYPE_ITEM;
    }

    private boolean isPositionHeader(int position) {
        return position == 0;
    }

    private String getItem(int position) {
        return data[position - 1];
    }

    class VHItem extends RecyclerView.ViewHolder {
        TextView title;

        public VHItem(View itemView) {
            super(itemView);
        }
    }

    class VHHeader extends RecyclerView.ViewHolder {
        Button button;

        public VHHeader(View itemView) {
            super(itemView);
        }
    }
}
```
上面的代码对`LinearLayoutManger`是没问题的，但是使用`GridLayoutManager`呢？ 如果是两列，那添加的`HeaderView`并不是占据上第一行，而是`HeaderView`与第二个`ItemView`一起占据第一行。那该怎么处理呢？ 
那就是使用`setSpanSizeLookup()`方法。
比如:      
```java
recyclerView.setLayoutManager(new GridLayoutManager(this, 2));
```
在上面的基本设置中，我们的`spanCount`为2，每个`item`的`span size`为1，因此一个`header`需要的`span size`则为2。在我尝试着添加`header`之前，我想先看看如何设置`span size`。其实很简单。
```java
final GridLayoutManager manager = new GridLayoutManager(this, 2);
recyclerView.setLayoutManager(manager);
manager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
  @Override
  public int getSpanSize(int position) {
    return adapter.isHeader(position) ? manager.getSpanCount() : 1;
  }
});
```

### 下拉刷新、自动加载

##### 实现下拉刷新      
实现下拉刷新也很简单了，可以使用`SwipeRefrshLayout`,`SwipeRefrshLayout`是`Google`官方提供的组件，可以实现下拉刷新的功能。已包含到`support.v4`包中。

主要方法有:      

- `setOnRefreshListener(OnRefreshListener)`:添加下拉刷新监听器
- `setRefreshing(boolean)`:显示或者隐藏刷新进度条
- `isRefreshing()`:检查是否处于刷新状态
- `setColorSchemeResources()`:设置进度条的颜色主题，最多设置四种。

```xml
<android.support.v4.widget.SwipeRefreshLayout
   android:layout_width="fill_parent"
   android:layout_height="fill_parent"
    android:scrollbars="vertical"
    >
   <android.support.v7.widget.RecyclerView
       android:layout_width="fill_parent"
       android:layout_height="fill_parent"
       ></android.support.v7.widget.RecyclerView>
</android.support.v4.widget.SwipeRefreshLayout>
```
具体实现就不写了。

##### 实现滑动自动加载更多功能

实现方式和`ListView`的实现方式类似，就是通过监听`scroll`时间，然后判断当前显示的`item`。

```java
//RecyclerView滑动监听
mRecylcerView.setOnScrollListener(new RecyclerView.OnScrollListener() {
    @Override
    public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
       super.onScrollStateChanged(recyclerView, newState);
        if (newState ==RecyclerView.SCROLL_STATE_IDLE && lastVisibleItem + 1 ==adapter.getItemCount()) {
            new Handler().postDelayed(new Runnable() {
                @Override
                public void run() {
                    List<String> newDatas = new ArrayList<String>();
                    for (int i = 0; i< 5; i++) {
                        int index = i +1;
                       newDatas.add("more item" + index);
                    }
                   adapter.addMoreItem(newDatas);
                }
            },1000);
        }
    }
    @Override
    public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
        super.onScrolled(recyclerView,dx, dy);
        lastVisibleItem =linearLayoutManager.findLastVisibleItemPosition();
    }
});
```

然后再通过结合`FooterView`以及增加几种状态就可以实现自动加载更多了。

    	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 