滑动切换Activity(GestureDetector)
===

1. 实现手势滑动切换`Activity`        
	- 创建一个手势识别器(`GestureDetector`)       
	- 在`Activity`的`onTouchEvent`中去使用该手势识别器      
		```java
		public abstract class SetupBaseActivity extends Activity {
			protected SharedPreferences sp;
			protected GestureDetector mGestureDetector;

			@Override
			protected void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				sp =getSharedPreferences("config", MODE_PRIVATE);
				initView();
				setupView();
				//1.创建一个手势识别器 new 对象，并给这个手势识别器设置监听器
				mGestureDetector = new GestureDetector(new GestureDetector.SimpleOnGestureListener(){
					//当手指在屏幕上滑动的时候 调用的方法.
					@Override
					//e1代表的是手指刚开始滑动的事件，e2代表手指滑动完了的事件
					public boolean onFling(MotionEvent e1, MotionEvent e2,float velocityX, float velocityY) {
						if(e1.getRawX() - e2.getRawX() > 200){
							showNext();//向右滑动，显示下一个界面
							return true;
						}

						if(e2.getRawX() - e1.getRawX() > 200){
							showPre();//向左滑动，显示上一个界面
							return true;
						}
						return super.onFling(e1, e2, velocityX, velocityY);
					}
				});
			}

			//2.让手势识别器生效，重写Activity的触摸事件，并且将Activity的触摸事件传入到手势识别器中
			@Override
			public boolean onTouchEvent(MotionEvent event) {
				mGestureDetector.onTouchEvent(event);
				return super.onTouchEvent(event);
			}
		}
		```

2. 实现切换效果
	经过上一步已经实现了滑动界面的切换，但是切换界面时的效果不好看，我们需要自定义切换的效果
	
	- 在res目录下面新建一个anim文件夹在这个文件夹中新建动画效果
		tran_next_in.xml//下一个界面进入的样式         
		tran_next_out.xml//下一个界面进入时当前页面出去的样式          
		tran_pre_in.xml//上一个界面进入的样式          
		tran_pre_out.xml//上一个界面进入时当前页面出去的样式        
 
		- tran_next_in.xml里面的内容
			```xml
			<?xml version="1.0" encoding="utf-8"?>
			<translate xmlns:android="http://schemas.android.com/apk/res/android"    //translate是指定整个图片是位移动
				android:fromXDelta="100%p" //开始时的X轴位置，100%p当表是当前窗体的宽度
				android:toXDelta="0"//结束时候的X轴位置
				android:fromYDelta="0"//开始时Y轴的位置
				android:toYDelta="0"//结束时Y轴的位置
				android:duration="300"//整个动画持续的时间
				>
			</translate>
			```
	 
		- tran_next_out.xml里面的内容
			```xml
			<?xml version="1.0" encoding="utf-8"?>
			<translate xmlns:android="http://schemas.android.com/apk/res/android"
				android:fromXDelta="0"
				android:toXDelta="-100%p"
				android:fromYDelta="0"
				android:toYDelta="0"
				android:duration="300"
				>
			</translate>
			```
	 
		- tran_pre_in.xml里面的内容
			```xml
			<?xml version="1.0" encoding="utf-8"?>
			<translate xmlns:android="http://schemas.android.com/apk/res/android"
				android:fromXDelta="-100%p"
				android:toXDelta="0"
				android:fromYDelta="0"
				android:toYDelta="0"
				android:duration="300"
				>
			</translate>
			```
	 
		- tran_pre_out.xml里面的内容
			```xml
			<?xml version="1.0" encoding="utf-8"?>
			<translate xmlns:android="http://schemas.android.com/apk/res/android"
				android:fromXDelta="0"
				android:toXDelta="100%p"
				android:fromYDelta="0"
				android:toYDelta="0"
				android:duration="300"
				>
			</translate>
			```
  
	- 让`Activity`在创建和销毁时使用上面自定义的动画
		
		`public void overridePendingTransition(int enterAnim, int exitAnim);`
		`Call immediately after one of the flavors of startActivity(Intent) or finish() to specify an explicit transition animation to perform next.`
		`Parameters:`
		`enterAnim - A resource ID of the animation resource to use for the incoming activity. Use 0 for no animation.`
		`exitAnim - A resource ID of the animation resource to use for the outgoing activity. Use 0 for no animation.`
	
		```java
		public void showNext() {      
			Intent intent = new Intent(this, Setup3Activity.class);
			startActivity(intent);
			finish();
			//调用此方法让动画效果生效
			overridePendingTransition(R.anim.tran_next_in, R.anim.tran_next_out);
		}
    
		public void showPre() {
			Intent intent = new Intent(this, Setup1Activity.class);
			startActivity(intent);
			finish();
			overridePendingTransition(R.anim.tran_pre_in, R.anim.tran_pre_out);
		}
		```
		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
 
 