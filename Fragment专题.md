Fragment专题
===

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