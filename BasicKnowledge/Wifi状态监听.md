Wifi状态监听
===

```java
/**
  * 监控Wifi状态的广播接收器
  */
private final class WifiStateReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context c, Intent intent) {
        Bundle bundle = intent.getExtras();
        int statusInt = bundle.getInt("wifi_state");
        switch (statusInt) {
        case WifiManager.WIFI_STATE_UNKNOWN:
            break;
        case WifiManager.WIFI_STATE_ENABLING:
            break;
        case WifiManager.WIFI_STATE_ENABLED:
            LogUtil.e(tag, "wifi enable");
            if(!isWifiEnable) {
                isWifiEnable = true;
                //断网后又连上了
                isGoon = false;
                if (!Util.isServiceRun(MultiPointControlActivity.this,
                        DLNAServiceName)) {
                    LogUtil.e(tag, "start dlna service");
                }else {
                    LogUtil.e(tag, "runing .... stop dlna service");
                    stopDLNAService();
                }
                startDLNAService();
                firstPlay();
            }
            break;
        case WifiManager.WIFI_STATE_DISABLING:
            break;
        case WifiManager.WIFI_STATE_DISABLED:
            isWifiEnable = false;
            LogUtil.e(tag, "wifi disable");
            break;
        default:
            break;
        }
    }
}

private void registReceiver() {
    receiver = new WifiStateReceiver();
    IntentFilter filter = new IntentFilter(WifiManager.WIFI_STATE_CHANGED_ACTION);
    registerReceiver(receiver, filter);
}
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 