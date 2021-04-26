MaterialDesign使用
===

- [Material Design](https://material.io/components?platform=android)是`Google`在`2014`年的`I/O`大会上推出的全新设计语言。                        
为了使用Materia功能，首先需要集成相应的依赖: 
```
implementation 'com.google.android.material:material:<version>'
```
并保证Activity都使用AppCompatActivity，这样可以确保所有组件都能正常使用。
把主题改成集成自Material组件的主题: 
```
    Theme.MaterialComponents
    Theme.MaterialComponents.NoActionBar
    Theme.MaterialComponents.Light
    Theme.MaterialComponents.Light.NoActionBar
    Theme.MaterialComponents.Light.DarkActionBar
    Theme.MaterialComponents.DayNight
    Theme.MaterialComponents.DayNight.NoActionBar
    Theme.MaterialComponents.DayNight.DarkActionBar
```

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
    ToolBar相当于是ActionBar的替代版，因此需要制定一个不带ActionBar的主题，
    可以通过自定义`theme`继承`Theme.AppCompat.Light.NoActionBar`或者在`theme`中通过以下配置来进行禁用`ActionBar`。
    
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
里面没有`colorAccent`的颜色，唯独colorAccent这个属性比较难理解，它不是用来指定Checkbox`等某一个按钮的颜色，而是更多表达了一个强调的意思，比如一些控件的选中状态也会使用colorAccent的颜色。
    
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

ToolBar最左侧的这个按钮就叫做HomeAsUp按钮，它默认的图标是一个返回的箭头，含义是返回上一个活动，可以通过setDisplayHomeAsUpEnabled来让导航按钮显示出来。 可以在onOptionsItemSelected()方法中对HomeAsUp安阿牛的点击事件进行处理，HomeAsUp按钮的id永远都是android.R.id.home。

​	           

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

### NavigationView

NavigationView包含两个部分：menu和headerLayout。menu是用来在NavigationView中显示具体的菜单项的，headerLayout则是用来在NavigationView中显示头部布局的。

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/navigator_view.png?raw=true)        

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="@color/colorPrimary" />

    </RelativeLayout>

    <android.support.design.widget.NavigationView
        android:id="@+id/navigation_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:headerLayout="@layout/drawer_header"
        app:menu="@menu/drawer_view" />
</android.support.v4.widget.DrawerLayout>
```



drawer_header.xml 如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:scaleType="fitXY"
        android:src="@drawable/image"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:textStyle="bold"
        android:textColor="@android:color/white"
        android:textSize="20sp"
        android:text="Header View"
        android:layout_marginTop="50dp"
        android:layout_marginLeft="25dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</RelativeLayout>
```



drawer_view.xml 如下： 

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">

        <item
            android:id="@+id/menu_home"
            android:icon="@drawable/ic_home"
            android:title="Home" />

        <item
            android:id="@+id/menu_settings"
            android:icon="@drawable/ic_settings"
            android:title="Settings" />

        <item android:title="Other">
            <menu>
                <item
                    android:id="@+id/menu_share"
                    android:icon="@drawable/ic_share"
                    android:title="Share" />
                <item
                    android:id="@+id/menu_about"
                    android:icon="@drawable/ic_info_outline"
                    android:title="About" />
            </menu>
        </item>
    </group>
</menu>
```



我们给 Header 和 Menu 添加点击事件： 

```java
final NavigationView navigationView = (NavigationView) findViewById(R.id.navigation_view);
        navigationView.getHeaderView(0).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                drawerLayout.closeDrawer(navigationView);
                Toast.makeText(MainActivity.this, "Header View is clicked!", Toast.LENGTH_SHORT).show();
            }
        });
        navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                switch (item.getItemId()) {
                    case R.id.menu_home:
                        Toast.makeText(MainActivity.this, "Home is clicked!", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.menu_settings:
                        Toast.makeText(MainActivity.this, "Settings is clicked!", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.menu_share:
                        Toast.makeText(MainActivity.this, "Share is clicked!", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.menu_about:
                        Toast.makeText(MainActivity.this, "About is clicked!", Toast.LENGTH_SHORT).show();
                        break;
                }
                drawerLayout.closeDrawer(navigationView);
                return false;
            }
        });
```



### CoordinatorLayout

**一、CoordinatorLayout 的作用**

从名字可以看出，这个ViewGroup是用来协调它的子View的，CoordinatorLayout 作为一个 **“super-powered FrameLayout”**，主要有以下两个作用：

1. 作为顶层布局；
2. 作为协调子View之间交互的容器。

CoordinatorLayout也是在`com.android.support.design`包中的组件。 

通过为 CoordinatorLayout 的子View指定 Behaviors，你可以在单一父View下提供许多不同的交互，同时也可以让子View间各自进行交互。
#### CoordinatorLayout与FloadingActionButton

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout     
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/contentView"
    android:orientation="vertical"     
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:onClick="onClick"
        android:layout_marginRight="10dp"
        android:layout_marginBottom="10dp"/>
</RelativeLayout>
```



```java
public void onClick(View v) {
    switch (v.getId()) {
        case R.id.fab:
            Snackbar.make(findViewById(R.id.contentView), "Snackbar", Snackbar.LENGTH_SHORT).show();
            break;
    }
}
```

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/smacker_floatingbutton.webp?raw=true)        

可以看到FloatingActionButton会被SmackBar遮挡，为了解决遮挡的问题，就需要使用到CoordinatorLayout。CoordinatorLayout可以说是一个加强版的FrameLayout，这个布局也是由Design Support库提供的。它在普通情况下的作用和FrameLayout基本一致，不过既然是Design Support库中提供的布局，那么就必然有一些Material Design的魔力了。
事实上，CoordinatorLayout可以监听其所有子控件的各种事件，然后自动帮助我们做出最为合理的响应。举个简单的例子，刚才弹出的Snackbar提示将悬浮按钮遮挡住了，而如果我们能让CoordinatorLayout监听到Snackbar的弹出事件，那么它会自动将内部的FloatingActionButton向上偏移，从而确保不会被Snackbar遮挡到。

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/contentView"
    android:orientation="vertical" 
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:id="@+id/anchorView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"/>
    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_anchor="@id/anchorView"
        app:layout_anchorGravity="bottom|right"
        android:onClick="onClick"
        android:layout_marginRight="10dp"
        android:layout_marginBottom="10dp"/>
</android.support.design.widget.CoordinatorLayout>
```



悬浮按钮自动向上偏移了Snackbar的同等高度，从而确保不会被遮挡住，当Snackbar消失的时候，悬浮按钮会自动向下偏移回到原来位置。
另外悬浮按钮的向上和向下偏移也是伴随着动画效果的，且和Snackbar完全同步，整体效果看上去特别赏心悦目。



或者我们也可以不把不过FloatingActionButton放到布局中，只是make()方法时传入的view是在布局中即可，我们回过头来再思考一下，刚才说的是CoordinatorLayout可以监听其所有子控件的各种事件，但是Snackbar好像并不是CoordinatorLayout的子控件吧，为什么它却可以被监听到呢？

其实道理很简单，还记得我们在Snackbar的make()方法中传入的第一个参数吗？这个参数就是用来指定Snackbar是基于哪个View来触发的，刚才我们传入的是FloatingActionButton本身，而FloatingActionButton是CoordinatorLayout中的子控件，因此这个事件就理所应当能被监听到了。你可以自己再做个试验，如果给Snackbar的make()方法传入一个DrawerLayout，那么Snackbar就会再次遮挡住悬浮按钮，因为DrawerLayout不是CoordinatorLayout的子控件，CoordinatorLayout也就无法监听到Snackbar的弹出和隐藏事件了。



#### CollapsingToolbarLayout

可折叠式标题栏，CollapsingToolbarLayout是一个作用于Toolbar基础之上的布局，它也是由Design Support库提供的。CollapsingToolbarLayout可以让Toolbar的效果变得更加丰富，不仅仅是展示一个标题栏，而是能够实现非常华丽的效果。



![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/CollapsingToolbarLayout.gif?raw=true)        



可以看到，我们在CollapsingToolbarLayout中定义了一个ImageView和一个Toolbar，也就意味着，这个高级版的标题栏将是由普通的标题栏加上图片组合而成的。这里定义的大多数属性我们都是见过的，就不再解释了，只有一个app:layout_collapseMode比较陌生。它用于指定当前控件在CollapsingToolbarLayout折叠过程中的折叠模式，其中Toolbar指定成pin，表示在折叠的过程中位置始终保持不变，ImageView指定成parallax，表示会在折叠的过程中产生一定的错位偏移，这种模式的视觉效果会非常好。

### NestedScrollView

NestedScrollView 即 支持嵌套滑动的ScrollView。

因此，我们可以简单的把NestedScrollView类比为ScrollView，其作用就是作为控件父布局，从而具备（嵌套）滑动功能。

NestedScrollView与ScrollView的区别就在于NestedScrollView支持 *嵌套滑动*，无论是作为父控件还是子控件，嵌套滑动都支持，且默认开启。

因此，在一些需要支持嵌套滑动的情景中，比如一个ScrollView内部包裹一个RecyclerView，那么就会产生滑动冲突，这个问题就需要你自己去解决。而如果使用NestedScrollView包裹RecyclerView，嵌套滑动天然支持，你无需做什么就可以实现前面想要实现的功能了。

我们通常为RecyclerView增加一个Header和Footer的方法是通过定义不同的viewType来区分的，而如果使用NestedScrollView，我们完全可以把RecyclerView当成一个单独的控件，然后在其上面增加一个控件作为Header，在其下面增加一个控件作为Footer。

虽然NestedScrollView内嵌RecyclerView和其他控件可以实现Header和Footer，但还是不推荐上面这种做法（建议还是直接使用RecyclerView自己添加Header和Footer），因为虽然NestedScrollView支持嵌套滑动，但是在实际应用中，嵌套滑动可能会带来其他的一些奇奇怪怪的副作用，Google 也推荐我们能不使用嵌套滑动就尽量不要使用。



作者：Whyn
链接：https://www.jianshu.com/p/f55abc60a879
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。








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



[更多实例请看MaterialSample](https://github.com/CharonChui/MaterialSample.git)

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
