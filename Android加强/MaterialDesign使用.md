MaterialDesign使用
===

- `Material Design`是`Google`在`2014`年的`I/O`大会上推出的全新设计语言。                        
- `Material Design`是基于`Android 5.0``(API level 21)`的，兼容5.0以下的设备时需要使用版本号`v21.0.0`以上的                           
`support v7`包中的`appcpmpat`，不过遗憾的是`support`包只支持`Material Design`的部分特性。                       
使用`eclipse`或`Android Studio`进行开发时，直接在`Android SDK Manager`中将`Extras->Android Support Library`
升级至最新版即可。

下面我就简单讲解一下如何通过`support v7`包来使用`Material Design`进行开发。

Material Design Theme
---

`Material`主题:                         

- @android:style/Theme.Material (dark version)             --               Theme.AppCompat                           
- @android:style/Theme.Material.Light (light version)      --               Theme.AppCompat.Light                 
- @android:style/Theme.Material.Light.DarkActionBar        --               Theme.AppCompat.Light.DarkActionBar                 

对应的效果分别如下:    

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/Material_theme.png?raw=true)    

使用ToolBar
---

- 禁止Action Bar                  
    可以通过使用`Material theme`来让应用使用`Material Design`。想要使用`ToolBar`需要先禁用`ActionBar`。
    可以通过自定义`theme`继承`Theme.AppCompat.Light.NoActionBar`或者在`theme`中通过以下配置来进行。
    ```xml
    <item name="windowActionBar">false</item>
    <item name="android:windowNoTitle">true</item>
    ```
    
    下面我通过第二种方式来看一下具体的实现:   
    
    在`style.xml`中自定义`AppTheme`:    
    
    ```xml
    <!-- Base application theme. -->
    <style name="AppTheme" parent="AppTheme.Base"/>
    
    <style name="AppTheme.Base" parent="Theme.AppCompat.Light.DarkActionBar">
    	<!-- Tell Android System that we will use ToolBar instead of ActionBar -->
    	<item name="windowNoTitle">true</item>
    	<item name="windowActionBar">false</item>
    	<!-- colorPrimary is used for the default action bar background -->
    	<item name="colorPrimary">@color/colorPrimary</item>
    
    	<!-- colorPrimaryDark is used for the status bar -->
    	<item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    
    	<!-- colorAccent is used as the default value for colorControlActivated
    		 which is used to tint widgets -->
    	<item name="colorAccent">@color/colorAccent</item>
    </style>
    ```
    
    配置的这几种颜色分别如下图所示:    
    ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/material_color.png?raw=true)	       
    里面没有`colorAccent`的颜色，这个颜色是设置`Checkbox`等控件选中时的颜色。

    在`values-v21`中的`style.xml`中同样自定义`AppTheme`主题: 
    ```xml
    <style name="AppTheme" parent="AppTheme.Base">
        <!-- Customize your theme using Material Design here. -->
    	<item name="android:windowContentTransitions" >true</item>
    	<item name="android:windowAllowEnterTransitionOverlap" >true</item>
    	<item name="android:windowAllowReturnTransitionOverlap">true</item>
    	<item name="android:windowSharedElementEnterTransition">@android:transition/move</item>
    	<item name="android:windowSharedElementExitTransition">@android:transition/move</item>
    </style>
    ```
    
- 在`Manifest`文件中设置`AppTheme`主题:   

    ```xml
    <application
    	android:allowBackup="true"
    	android:icon="@mipmap/ic_launcher"
    	android:label="@string/app_name"
    	android:theme="@style/AppTheme" >
    	<activity
    		...
    	</activity>
    	<activity android:name="com.charon.materialsample.FriendsActivity"></activity>
    </application>
    ```
    这里说一下为什么要在`values-v21`中也自定义个主题，这是为了能让在`21`以上的版本能更好的使用`Material Design`，
在21以上的版本中会有更多的动画、特效等。

- 让Activity继承AppCompatActivity

    ```java
    public class MainActivity extends AppCompatActivity {
    	...
    }
    ```

- 在布局文件中进行声明

    声明`toolbar.xml`，我们把他单独放到一个文件中，方便多布局使用:    
    ```xml
    <android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:local="http://schemas.android.com/apk/res-auto"
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
        local:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        local:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar" />
    ```
    在`Activity`的布局中使用`ToolBar`:    
    
    ```xml
    <LinearLayout
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	android:orientation="vertical">
    
    	<include
    		android:id="@+id/toolbar"
    		layout="@layout/toolbar" />
    
    	<com.charon.materialsample.view.PagerSlidingTabStrip
    		android:id="@+id/psts_main"
    		android:layout_width="match_parent"
    		android:layout_height="48dip"
    		android:background="@color/colorPrimary" />
    
    	<android.support.v4.view.ViewPager
    		android:id="@+id/vp_main"
    		android:layout_width="match_parent"
    		android:layout_height="match_parent"></android.support.v4.view.ViewPager>
    
    </LinearLayout>
    ```

- 在Activity中设置ToolBar
    ```java
    public class MainActivity extends AppCompatActivity{
        private Context mContext;
        private Toolbar mToolbar;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            mContext = this;
    		mToolbar = (Toolbar) findViewById(R.id.toolbar);
    		mToolbar.setTitle(R.string.app_name);
    		// 将ToolBar设置为ActionBar，这样一设置后他就能像ActionBar一样直接显示menu目录中的菜单资源
    		// 如果不用该方法，那ToolBar就只是一个普通的View，对menu要用inflateMenu去加载布局。
    		setSupportActionBar(mToolbar);
    		getSupportActionBar().setDisplayShowHomeEnabled(true);
        }
    }
    ```

到这里运行项目就可以了，就可以看到一个简单的`ToolBar`实现。

接下来我们看一下`ToolBar`中具体有哪些内容:    

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/ToolBar_content.jpg?raw=true)	              

我们可以通过对应的方法来修改他们的属性:                 
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/toolbarCode.png?raw=true)	           

对于`ToolBar`中的`Menu`部分我们可以通过一下方法来设置:     
```java
toolbar.inflateMenu(R.menu.menu_main);
toolbar.setOnMenuItemClickListener();
```
或者也可以直接在`Activity`的`onCreateOptionsMenu`及`onOptionsItemSelected`来处理: 
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
	// Inflate the menu; this adds items to the action bar if it is present.
	getMenuInflater().inflate(R.menu.menu_main, menu);
	return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
	// Handle action bar item clicks here. The action bar will
	// automatically handle clicks on the Home/Up button, so long
	// as you specify a parent activity in AndroidManifest.xml.
	int id = item.getItemId();

	//noinspection SimplifiableIfStatement
	if (id == R.id.action_settings) {
		return true;
	}
	if (id == R.id.action_search) {
		Toast.makeText(getApplicationContext(), "Search action is selected!", Toast.LENGTH_SHORT).show();
		return true;
	}
	return super.onOptionsItemSelected(item);
}
```
`menu`的实现如下:   
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools" tools:context=".MainActivity">

        <item
            android:id="@+id/action_search"
            android:title="@string/action_search"
            android:orderInCategory="100"
            android:icon="@drawable/ic_action_search"
            app:showAsAction="ifRoom" />

        <item
            android:id="@+id/action_settings"
            android:title="@string/action_settings"
            android:orderInCategory="100"
            app:showAsAction="never" />

</menu>
```

如果想要对`NavigationIcon`添加点击实现:  
```java
toolbar.setNavigationOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		onBackPressed();
	}
});
```

运行后发现我们强大的`Activity`切换动画怎么在`5.0`一下系统上实现呢？`support v7`包也帮我们考虑到了。使用`ActivityOptionsCompat`
及`ActivityCompat.startActivity`，但是悲剧了，他对4.0一下基本都无效，而且就算在4.0上很多动画也不行，具体还是用其他
大神在`github`写的开源项目吧。


- 动态取色Palette 

    `Palette`这个类中可以提取一下集中颜色：　　
    
    - Vibrant （有活力）
    - Vibrant dark（有活力 暗色）
    - Vibrant light（有活力 亮色）
    - Muted （柔和）
    - Muted dark（柔和 暗色）
    - Muted light（柔和 亮色）
    
    ```java
    //目标bitmap，代码片段
    Bitmap bm = BitmapFactory.decodeResource(getResources(),
    		R.drawable.kale);
    Palette palette = Palette.generate(bm);
    if (palette.getLightVibrantSwatch() != null) {
    	//得到不同的样本，设置给imageview进行显示
    	iv.setBackgroundColor(palette.getLightVibrantSwatch().getRgb());
    	iv1.setBackgroundColor(palette.getDarkVibrantSwatch().getRgb());
    	iv2.setBackgroundColor(palette.getLightMutedSwatch().getRgb());
    	iv3.setBackgroundColor(palette.getDarkMutedSwatch().getRgb());
    }
    ```

使用DrawerLayout
---

- 布局中的使用

```xml
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--主页面-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <include
            android:id="@+id/toolbar"
            layout="@layout/toolbar" />

        <com.charon.materialsample.view.PagerSlidingTabStrip
            android:id="@+id/psts_main"
            android:layout_width="match_parent"
            android:layout_height="48dip"
            android:background="@color/colorPrimary" />

        <android.support.v4.view.ViewPager
            android:id="@+id/vp_main"
            android:layout_width="match_parent"
            android:layout_height="match_parent"></android.support.v4.view.ViewPager>

    </LinearLayout>

    <!--侧边栏部分-->
    <fragment
        android:id="@+id/fragment_navigation_drawer"
        android:name="com.charon.materialsample.fragment.FragmentDrawer"
        android:layout_width="@dimen/nav_drawer_width"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:layout="@layout/fragment_navigation_drawer"
        tools:layout="@layout/fragment_navigation_drawer" />

</android.support.v4.widget.DrawerLayout>
```

使用DrawerLayout后可以实现类似SlidingMenu的效果。但是怎么将DrawerLayout与ToolBar结合起来呢？ 还有再结合Navigation Tabs
以及ViewPager。下面我就直接上代码了。

先看布局：　activity_main.xml                
```xml
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--主页面-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <include
            android:id="@+id/toolbar"
            layout="@layout/toolbar" />

        <com.charon.materialsample.view.PagerSlidingTabStrip
            android:id="@+id/psts_main"
            android:layout_width="match_parent"
            android:layout_height="48dip"
            android:background="@color/colorPrimary" />

        <android.support.v4.view.ViewPager
            android:id="@+id/vp_main"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </LinearLayout>

    <!--侧边栏部分-->
    <fragment
        android:id="@+id/fragment_navigation_drawer"
        android:name="com.charon.materialsample.fragment.DrawerFragment"
        android:layout_width="wrap_content"
        android:layout_marginRight="56dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:layout="@layout/fragment_navigation_drawer"
        tools:layout="@layout/fragment_navigation_drawer" />

</android.support.v4.widget.DrawerLayout>
```

MainActivity的代码: 
```java
public class MainActivity extends AppCompatActivity {

    private Context mContext;

    private Toolbar mToolbar;
    private PagerSlidingTabStrip mScrollingTabs;
    private ViewPager mViewPager;
    private MainPagerAdapter mPagerAdapter;
    private ActionBarDrawerToggle mDrawerToggle;
    private DrawerLayout mDrawerLayout;

    private List<String> mTitles;
    private List<Fragment> mFragments;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mContext = this;

        findView();
        setToolBar();
        initView();
        initDrawerFragment();
    }

    private void findView() {
        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mScrollingTabs = (PagerSlidingTabStrip) findViewById(R.id.psts_main);
        mViewPager = (ViewPager) findViewById(R.id.vp_main);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
    }


    private void setToolBar() {
        mToolbar.setTitle(R.string.app_name);
        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayShowHomeEnabled(true);
    }

    private void initView() {
        mFragments = new ArrayList<>();
        for (int xxx = 0; xxx < 5; xxx++) {
            mFragments.add(new FriendsFragment());
        }

        mTitles = new ArrayList<>();
        for (int xxx = 0; xxx < 5; xxx++) {
            mTitles.add("Tab : " + xxx);
        }

        mPagerAdapter = new MainPagerAdapter(getSupportFragmentManager(), mFragments, mTitles);
        mViewPager.setAdapter(mPagerAdapter);
        mScrollingTabs.setDividerColor(Color.TRANSPARENT);
        mScrollingTabs.setIndicatorHeight(10);
        mScrollingTabs.setUnderlineHeight(0);
        mScrollingTabs.setTextSize(50);
        mScrollingTabs.setTextColor(Color.BLACK);
        mScrollingTabs.setSelectedTextColor(Color.WHITE);
        mScrollingTabs.setViewPager(mViewPager);

    }

    private void initDrawerFragment() {
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout, mToolbar, R.string.drawer_open, R.string.drawer_close) {
            @Override
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                MainActivity.this.invalidateOptionsMenu();
            }

            @Override
            public void onDrawerClosed(View drawerView) {
                super.onDrawerClosed(drawerView);
                MainActivity.this.invalidateOptionsMenu();
            }

            @Override
            public void onDrawerSlide(View drawerView, float slideOffset) {
                super.onDrawerSlide(drawerView, slideOffset);
                mToolbar.setAlpha(1 - slideOffset / 2);
            }
        };

        mDrawerLayout.setDrawerListener(mDrawerToggle);
        mDrawerToggle.syncState();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }
        if (id == R.id.action_search) {
            Toast.makeText(getApplicationContext(), "Search action is selected!", Toast.LENGTH_SHORT).show();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

}
```
最后再看一下`DrawerFragment`的代码: 
```java
public class DrawerFragment extends Fragment {
    private Context mContext;
    private RecyclerView mRecyclerView;
    private NavigationDrawerAdapter mAdapter;
    private static String[] titles = null;

    public DrawerFragment() {

    }

    public static List<NavDrawerItem> getData() {
        List<NavDrawerItem> data = new ArrayList<>();

        // preparing navigation drawer items
        for (int i = 0; i < titles.length; i++) {
            NavDrawerItem navItem = new NavDrawerItem();
            navItem.setTitle(titles[i]);
            data.add(navItem);
        }
        return data;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mContext = getActivity();
        // drawer labels
        titles = getActivity().getResources().getStringArray(R.array.nav_drawer_labels);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflating view layout
        View layout = inflater.inflate(R.layout.fragment_navigation_drawer, container, false);
        mRecyclerView = (RecyclerView) layout.findViewById(R.id.drawerList);
        mRecyclerView.setHasFixedSize(true);
        mAdapter = new NavigationDrawerAdapter(getActivity(), getData());
        mRecyclerView.setAdapter(mAdapter);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
        mAdapter.setOnRecyclerViewListener(new NavigationDrawerAdapter.OnRecyclerViewListener() {
            @Override
            public void onItemClick(int position) {
                Toast.makeText(mContext, getData().get(position).getTitle(), Toast.LENGTH_SHORT).show();
                startActivity(new Intent(getActivity(), FriendsActivity.class));
            }

            @Override
            public boolean onItemLongClick(int position) {
                return false;
            }
        });
        return layout;
    }

}
```
上面的`PagerSlidingTabStrip`是开源项目，我改了下，添加了一个选中时的文字颜色改变。

[Demo地址](https://github.com/CharonChui/MaterialLibrary)


Ripple效果
---

个人非常喜欢的效果。相当于给点击事件加上了动态的赶脚。。。


假设现在有一个`Button`的`selector`，我们想给这个`Button`加上`Ripple`效果，肿么办？ 
新建一个`xml`文件，用`ripple`包裹`selector`，然后在`Button`的`backgroud`直接引用这个`xml`就好了。
```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
android:color="@color/whatever">
<selector xmlns:android="http://schemas.android.com/apk/res/android"
<item android:state_pressed="false" android:state_focused="true"
android:drawable="@drawable/some_focused_blah"/>
<item android:state_pressed="true" android:drawable="@drawable/some_pressed_blah"/>
<item android:drawable="@android:color/whatever"/>
</selector>
</ripple>

```
但是很遗憾，`ripple`是5.0才有的，而且`support`包中没有实现该功能的扩展。
`5.0`的这些效果还是无法在低版本上实现，包括一些`TextView`等样式，现在可以用大神的开源项目        
[MaterialDesignLibrary](https://github.com/navasmdc/MaterialDesignLibrary)


RecyclerView
---

`ListView`的升级版，还有什么理由不去用呢？ 同样他也在`support v7`包中。
```
compile 'com.android.support:recyclerview-v7:21.+'  
```
通过`mRecyclerView.setLayoutManager(new LinearLayoutManager(this)); `设置为`LinearLayoutManager`来实现水平或者竖直
方向的`ListView`。


阴影
---

通过对`View`设置`backgroud`后再添加`android:elevation="2dp"`来实现背景大小。



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 