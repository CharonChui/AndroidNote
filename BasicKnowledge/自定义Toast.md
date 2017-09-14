自定义Toast
===

系统`Toast`提示时不能够进行取消，如果有多个`Toast`时会很长时间才消失。自定义`Toast`通过`WindowManager`来进行手动的控制`Toast`的显示与隐藏。能有效的解决该问题。

`Toast`提示的布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="@android:drawable/toast_frame "
    android:gravity="center"
    android:orientation="horizontal" >
    <ImageView
        android:id="@+id/toast_img"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone" />
    <TextView
        android:id="@+id/toast_text"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_weight="1"
        android:textAppearance="@android:style/TextAppearance.Small"
        android:textColor="#ffffffff" />
</LinearLayout>
``` 

```java
/**
 * 吐司提示的工具类，能够控制吐司的显示和隐藏
 */
public class ToastUtil {
    public static final int LENGTH_SHORT = 0;
    public static final int LENGTH_LONG = 1;
    private static View toastView;
    private WindowManager mWindowManager;
    private static int mDuration;
    private final int WHAT = 100;
    private static View oldView;
    private static Toast toast;
    private static CharSequence oldText;
    private static CharSequence currentText;
    private static ToastUtil instance = null;
    private static TextView textView;
 
    private ToastUtil(Context context) {
        mWindowManager = (WindowManager) context.getApplicationContext()
                .getSystemService(Context.WINDOW_SERVICE);
        toastView = LayoutInflater.from(context).inflate(R.layout.toast_view,
                null);
        textView = (TextView) toastView.findViewById(R.id.toast_text);
        toast = Toast.makeText(context, "", Toast.LENGTH_SHORT);
    }
    private static ToastUtil getInstance(Context context) {
        if (instance == null) {
            synchronized (ToastUtil.class) {
                if (instance == null)
                    instance = new ToastUtil(context);
            }
        }
        return instance;
    }
    public static ToastUtil makeText(Context context, CharSequence text,
            int duration) {
        ToastUtil util = getInstance(context);
        mDuration = duration;
        toast.setText(text);                
        currentText = text;
        textView.setText(text);
        return util;
    }
    public static ToastUtil makeText(Context context, int resId, int duration{
        ToastUtil util = getInstance(context);
        mDuration = duration;
        toast.setText(resId); 
        currentText = context.getResources().getString(resId);
        textView.setText(context.getResources().getString(resId));
        return util;
    }
    /**
     * 进行Toast显示，在显示之前会取消当前已经存在的Toast
     */
    public void show() {
        long time = 0;
        switch (mDuration) {
        case LENGTH_SHORT:
            time = 2000;
            break;
        case LENGTH_LONG:
            time = 3500;
            break;
        default:
            time = 2000;
            break;
        }
        if (currentText.equals(oldText) && oldView.getParent() != null) {
            toastHandler.removeMessages(WHAT);
            toastView = oldView;
            oldText = currentText;
            toastHandler.sendEmptyMessageDelayed(WHAT, time);
            return;
        }
        cancelOldAlert();
        toastHandler.removeMessages(WHAT);
        WindowManager.LayoutParams params = new WindowManager.LayoutParams();
        params.height = WindowManager.LayoutParams.WRAP_CONTENT;
        params.width = WindowManager.LayoutParams.WRAP_CONTENT;
        params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE
                | WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;
        params.format = PixelFormat.TRANSLUCENT;
        params.windowAnimations = android.R.style.Animation_Toast;
        params.type = WindowManager.LayoutParams.TYPE_TOAST;
        params.setTitle("Toast");
        params.gravity = toast.getGravity();
        params.y = toast.getYOffset();
        if (toastView.getParent() == null) {
            mWindowManager.addView(toastView, params);
        }
        oldView = toastView;
        oldText = currentText;
        toastHandler.sendEmptyMessageDelayed(WHAT, time);
    }
    private Handler toastHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            cancelOldAlert();
            int id = msg.what;
            if (WHAT == id) {
                cancelCurrentAlert();
            }
        }
    };
    private void cancelOldAlert() {
        if (oldView != null && oldView.getParent() != null) {
            mWindowManager.removeView(oldView);
        }
    }
    public void cancelCurrentAlert() {
        if (toastView != null && toastView.getParent() != null) {
            mWindowManager.removeView(toastView);
        }
    }
}
```


在某些Pad上面Toast显示出来后就不会自动消失，在这些Pad上`toastView.getParent()会为nul`这样就导致无法移除。可以将`cancelOldAlert()`以及
`cancelCurrentAlert()`进行如下修改。

```java
private void cancelOldAlert() {
    if (oldView != null) { // 去掉 oldView.getParent() != null 这个参数，然后加上try catch代码块，解决在部分Pad上oldView.getParent()不准确的问题 
        try {
            mWindowManager.removeView(oldView);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

public void cancelCurrentAlert() {
    if (toastView != null) {
        try {
		    // 去掉 oldView.getParent() != null 这个参数，然后加上try catch代码块，解决在部分Pad上oldView.getParent()不准确的问题 
            mWindowManager.removeView(toastView);
        } catch (Exception e) {
            e.printStackTrace();
        }
    } else if (oldView != null) {
        try {
            mWindowManager.removeView(oldView);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 