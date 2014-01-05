PopupWindow
===
`
A popup window that can be used to display an arbitrary view. The popup windows is a floating container that appears on top of the current activity.`
 
1. 显示
	```java
	View contentView = View.inflate(getApplicationContext(),R.layout.popup_appmanger, null); 
	//这里最后一个参数是指定是否能够获取焦点，如果为false那么点击弹出来之后就不消失了，但是设置为true之后点击一个条目它弹出来了，再点击别的条目的时候这个popupWindow窗口没有焦点了就自己消失了
	PopupWindow popwindow = new PopupWindow(contentView ,LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT,true);
	popwindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
	int[] location = new int[2];
	//得到当前组件的位置
	view.getLocationInWindow(location);
	//显示popupWindow
	popwindow.showAtLocation(parent,Gravity.LEFT | Gravity.TOP, DensityUtil.dip2px(getApplicationContext(), location[0] + 70),location[1]);
	```

2. 取消
	```java
	if (popwindow != null && popwindow.isShowing()) {
	   popwindow.dismiss();
	   popwindow = null;
	} 
	```

3. 细节
	`PopupWindow`是存在到`Activity`上的，如果`PopupWindow`在显示的时候按退出键的时候该`Activity`已经销毁，但是`PopupWindow`没有销毁，所以就报错了(`Logcat`有报错信息但是程序不会崩溃)，所以我们在`Activity`的`onDestroy`方法中要判断一下`PopupWindow`是否在显示，如果在显示就取消显示。     
	`PopupWindow`默认是没有背景的，如果想让它播放动画就没有效果了，因为没有背景就什么也播放不了，所以我们在用这个`PopupWindow`的时候必须要给它设置一个背景，通常可以给它设置为透明色,这样再播放动画就有效果了
	`popwindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));`
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 