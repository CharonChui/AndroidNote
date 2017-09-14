Home键监听
===

1. `Home`键是一个系统的按钮，我们无法通过`onKeyDown`进行拦截，它是拦截不到的，我们只能得到他在什么时候被按下了。就是通过广播接收者
    ```java
    public class HomeKeyEventBroadCastReceiver extends BroadcastReceiver {
        static final String SYSTEM_REASON = "reason";
        static final String SYSTEM_HOME_KEY = "homekey";
        static final String SYSTEM_RECENT_APPS = "recentapps";
    
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (action.equals(Intent.ACTION_CLOSE_SYSTEM_DIALOGS)) {
                String reason = intent.getStringExtra(SYSTEM_REASON);
                if (reason != null) {
                    if (reason.equals(SYSTEM_HOME_KEY)) {
                        // home key处理点
                    } else if (reason.equals(SYSTEM_RECENT_APPS)) {
                        // long home key处理点
                    }
                }
            }
        }
    }
     ```

2. 在`Activity`中去注册这个广播接收者
    ```java
    receiver = new HomeKeyEventBroadCastReceiver();
    registerReceiver(receiver, new IntentFilter(Intent.ACTION_CLOSE_SYSTEM_DIALOGS));
    ```

3. 在`Activity`销毁的方法中去取消注册
    ```java
    unRegisterReceiver(receiver);
    ```

----

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 