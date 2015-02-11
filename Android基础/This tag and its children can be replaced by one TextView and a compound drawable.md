This tag and its children can be replaced by one TextView and a compound drawable
===

写项目的时候经常会遇到在`TextView`附近显示图片的问题，一般都是嵌套一层布局。
---
```xml
<LinearLayout
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:orientation="vertical" >

	<ImageView
		android:id="@+id/mediacontroller_battery"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:background="@drawable/battery_100"
		android:contentDescription="@string/iv_des" />

	<TextView
		android:id="@+id/mediacontroller_battery_level"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_gravity="center_horizontal"
		android:paddingRight="12dp"
		android:textColor="@android:color/white" />
</LinearLayout>
```

这样写的时候会有个`warning`提示，`warning:This tag and its children can be replaced by one TextView and a compound drawable`
意思就是这样写不合理，可以直接用`TextView`达到这种效果
```xml
<TextView
	android:id="@+id/mediacontroller_battery_level"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:drawableTop="@drawable/battery_100"
	android:layout_gravity="center_horizontal"
	android:paddingRight="12dp"
	android:drawablePadding="0dp" // 有些时候会用到
	android:textColor="@android:color/white" />
```
代码的方式： `mBatteryLevel.setCompoundDrawables(left, top, right, bottom);`


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 