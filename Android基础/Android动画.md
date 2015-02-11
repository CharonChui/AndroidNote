Android动画
===

1. AlphaAnimation

    ```java
    RelativeLayout rl_splash = (RelativeLayout) findViewById(R.id.rl_splash);
    //播放动画效果
    AlphaAnimation animation = new AlphaAnimation(1.0f, 0.0f);
    //设置Alpha动画的持续时间
    animation.setDuration(2000);
    //播放Alpha动画
    rl_splash.setAnimation(animation);
    ```

2. RotateAnimation

    ```java
    //相对于自身的哪个位置旋转，这里是相对于自身的右下角
    RotateAnimation ra = new RotateAnimation(0, 360,  //从哪旋转，旋转多少度
            Animation.RELATIVE_TO_SELF, 1.0f, Animation.RELATIVE_TO_SELF,
            1.0f);
    ra.setDuration(800);
    ra.setRepeatCount(Animation.INFINITE);
    ra.setRepeatMode(Animation.RESTART);
    iv_scan.startAnimation(ra);
     ```
3. ScaleAnimation(缩放动画)    
     
    ```java 
    ScaleAnimation(float fromX, float toX, float fromY, float toY) 
    Constructor to use when building a ScaleAnimation from code
    ```

4. TranslateAnimation(位移动画)     
    
    ```java
    TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue) 
    Constructor to use when building a TranslateAnimation from code
    ```

5. AnimationSet (多组动画)
    
    ```java
    ScaleAnimation sa = new ScaleAnimation(0.2f, 1.0f, 0.4f,1.0f);//缩放的动画效果,1.0f就代表窗体的总宽或者高
    sa.setDuration(400);
    TranslateAnimation ta = new TranslateAnimation(//位移动的动画效果
              Animation.RELATIVE_TO_SELF, 0,//指定这个位置是相对于谁
              Animation.RELATIVE_TO_SELF, 0.1f,
              Animation.RELATIVE_TO_SELF, 0,
              Animation.RELATIVE_TO_SELF, 0);
    ta.setDuration(300);
    AnimationSet set = new AnimationSet(false);//如果想播放多种动画的组合，这里就要用到了AnimationSet
    set.addAnimation(sa);
    set.addAnimation(ta);
    contentView.startAnimation(set); // 播放一组动画. 
    ```

6. Frame动画    

    - 在`drawable`目录下新建一个`xml`文件，内容如下:
    
        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
            android:oneshot="true" > //onshot是指定是否循环播放
            <item
                android:drawable="@drawable/desktop_rocket_launch_1"  //Frame动画的图片
                android:duration="50"/> //播放这个图片持续的时间
            <item
                android:drawable="@drawable/desktop_rocket_launch_2"
                android:duration="100"/>
        </animation-list>
        ```
    - 播放Frame动画
    
        ```java
        AnimationDrawable rocketAnimation;
        public void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.main);
              ImageView rocketImage = (ImageView) findViewById(R.id.iv);
              rocketImage.setBackgroundResource(R.drawable.animlist); //将上边建的Frame动画的xml文件通过背景资源设置给图片
              rocketAnimation = (AnimationDrawable) rocketImage.getBackground();  //获取到图片的背景资源
        }
        public void start(View view) {
              if (!rocketAnimation.isRunning()) {
                   rocketAnimation.start();  //播放
              }
        }
        ```
        
7. 保持动画播放完成后的状态`animation.setFillAfter(true);`   

    ```java
    Interpolator //定义了动画的变化速度，可以实现匀速、正加速、负加速、无规则变加速度
    AccelerateDecelerateInterpolator//先加速后减速。
    AccelerateInterpolator//逐渐加速。    
    LinearInterpolator//平稳不变的   
    DecelerateInterpolator//逐渐减速
    CycleInterpolator//曲线运动特效，要传递float型的参数。     
    animation.setInterpolator(new LinearInterpolator());//指定动画的运行效果
    ```
    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
