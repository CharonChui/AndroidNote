This tag and its children can be replaced by one TextView and a compound drawable
===

一直以来忙着赶项目，无脑的去写代码，没有时间停下来优化一下、仔细思考学习，最近突感不适，决定不能这样下去，我要走到正规的渠道上来。      
今天写项目的时候遇到一个warning，本着本屌不屈不挠、认真钻研的精神，决定消灭它。

华丽丽的分割线

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
`warning:This tag and its children can be replaced by one TextView and a compound drawable`
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

强迫症的孩子伤不起

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 