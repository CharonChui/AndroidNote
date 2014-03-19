Fragment专题
===

Fragment简介
---

Fragment 必须总是被嵌入到一个activity 中, 它们的生命周期直接被其所属的宿主,activity 的生命周期影响.例如, 当activity 被暂停,那么在其中的所有fragment 也被暂停; 当activity 被销毁,所有隶属于它的fragment 也被销毁. 然而,当一个activity 正在运行时(处于resumed 状态),我们可以独立地操作每一个fragment, 比如添加或删除它们. 将一个fragment 作为activity 布局的一部分添加进来时, 它处在activity 的viewhierarchy中的ViewGroup 中,并且定义有它自己的view 布局.

1. Fragment生命周期
    onAttach()Fragment被绑定到Activity时调用 ---> onCreate()Fragment创建 --> onCreateView()创建和Fragment关联的View Hierarchy时调用 --> onActivityCreated()Activity的onCreate()方法返回时调用，这前面的四个状态都对应着Activity的Created --> onStart() --> onResume() --> onPause() --> onStop() --> onDestroyView()当和fragment 关联的view hierarchy 正在被移除时调用. 从这个状态开始到最后这三个状态都对应着Activity的Destroyed状态--> onDestroy()Activity的onDestroy执行后的回调, -- onDetach()当fragment 从activity 解除关联时被调用.;


2. Fragment添加到Activity的两种方式
    1. 布局添加
	    ```xml
		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			android:orientation="horizontal"
			android:layout_width="match_parent"
			android:layout_height="match_parent">
				<fragment android:name="com.example.news.ArticleListFragment"
					android:id="@+id/list"
					android:layout_weight="1"
					android:layout_width="0dp"
					android:layout_height="match_parent" />
				<fragment android:name="com.example.news.ArticleReaderFragment"
					android:id="@+id/viewer"
					android:layout_weight="2"
					android:layout_width="0dp"
					android:layout_height="match_parent" />
		</LinearLayout>
		<fragment> 中的android:name 属性指定了在layout 中实例化的Fragment 类.
		```
		**每一个fragment 都需要一个唯一的标识,如果activity重启,系统可以用来恢复fragment(并且你也可以用来捕获fragment 来处理事务,例如移除它.)**
		有3 种方法来为一个fragment 提供一个标识:
			1. 为android:id 属性提供一个唯一ID.
			2. 为android:tag 属性提供一个唯一字符串.
			3. 如果以上2 个你都没有提供, 系统使用容器view 的ID.
			
	2. 通过代码添加
		当activity 运行的任何时候, 都可以将fragment 添加到activitylayout.只需简单的指定一个需要放置fragment 的ViewGroup.为了在你的activity 中操作fragment 事务(例如添加,移除,或代替一个fragment),必须使用来自FragmentTransaction 的API.
		```java
		FragmentManager fragmentManager = getFragmentManager();
		FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
		ExampleFragment fragment = new ExampleFragment();
		//add()的第一个参数是fragment 要放入的ViewGroup, 由resource ID 指定,第二个参数是需要添加的fragment
		fragmentTransaction.add(R.id.fragment_container, fragment);
		fragmentTransaction.commit();
		```
	
	3. 添加没有UI的Fragment
		之前的例子展示了对UI 的支持, 如何将一个fragment 添加到activity. 然而,也可以使用fragment 来为activity 提供后台行为而不用展现额外的UI.要添加一个无UI 的fragment, 需要从activity 使用add(Fragment, String) 来添加fragment (为fragment 提供一个唯一的字符串"tag", 而不是一个view ID).这么做添加了fragment,但因为它没有关联到一个activity layout 中的一个view, 所以不会接收到onCreateView()调用.因此不必实现此方法.为fragment 提供一个字符串tag 并不是专门针对无UI 的fragment的–也可以提供字符串tag 给有UI 的fragment – 但是如果fragment 没有UI,那么这个tag 是仅有的标识它的途径.如果随后你想从activity 获取这个fragment, 需要使用findFragmentByTag().
		
3. 管理Fragment
	要在activity 中管理fragment,需要使用FragmentManager. 通过调用activity 的getFragmentManager()取得它的实例.
可以通过FragmentManager 做一些事情, 包括:
    1. 使用findFragmentById() (用于在activitylayout 中提供一个UI 的fragment)或findFragmentByTag()(适用于有或没有UI 的fragment)获取activity 中存在的fragment
    2. 将fragment 从后台堆栈中弹出, 使用popBackStack() (模拟用户按下BACK 命令).
    3. 使用addOnBackStackChangeListener()注册一个监听后台堆栈变化的listener.

4. 处理Fragment事务
    每一个事务都是同时要执行的一套变化.可以在一个给定的事务中设置你想执行的所有变化,使用诸如add(), remove(),和replace().然后, 要给activity 应用事务, 必须调用commit().在调用commit()之前, 你可能想调用addToBackStack(),将事务添加到一个fragment 事务的backstack. 这个	

	将一个fragment 替换为另一个, 并在后台堆栈中保留之前的状态:
	```java
	// Create new fragment and transaction
	Fragment newFragment = new ExampleFragment();
	FragmentTransaction transaction = getFragmentManager().beginTransaction();
	// Replace whatever is in the fragment_container view with this fragment,
	// and add the transaction to the back stack
	transaction.replace(R.id.fragment_container, newFragment);
	transaction.addToBackStack(null);
	// Commit the transaction
	transaction.commit();
	```
    在这个例子中, newFragment 替换了当前layout 容器中的由R.id.fragment_container 标识的
    fragment.通过调用addToBackStack(), replace 事务被保存到back stack,因此用户可以回退事务,并通过按下BACK 按键带回前一个fragment.
	
	
	
	
`FragmentStatePagerAdapter`，会自动保存和恢复`Fragment`。

Fragment真正的onPause以及onResume
---

`Fragment`虽然有`onResume()`和`onPause()`方法，但是这两个方法是`Activity`的方法调用时机也与`Activity`相同，和`ViewPager`搭配使用这个方法就很鸡肋了，根本不是你想要的效果，这里介绍一种方法。
```java
@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
	super.setUserVisibleHint(isVisibleToUser);
	if (isVisibleToUser) {
		//相当于Fragment的onResume
	} else {
		//相当于Fragment的onPause
	}
}
```

通过阅读`ViewPager`和`PageAdapter`相关的代码，切换`Fragment`实际上就是通过设置`setUserVisibleHint`和`setMenuVisibility`来实现的，调用这个方法时并不会释放掉`Fragment`（即不会执行onDestoryView）。

------------------------------------------

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 