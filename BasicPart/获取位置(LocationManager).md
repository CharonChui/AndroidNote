获取位置(LocationManager)
===

1. 需要申请权限
	```xml
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
	```
	
2. 代码
	```java
	public class TestgpsActivity extends Activity {
		private LocationManager lm;
		private MyListener listener;

		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.main);
			lm = (LocationManager) getSystemService(LOCATION_SERVICE);

			Criteria criteria = new Criteria();
			criteria.setAccuracy(Criteria.ACCURACY_FINE);
			criteria.setCostAllowed(true);
			criteria.setPowerRequirement(Criteria.POWER_HIGH);
			criteria.setSpeedRequired(true);
			String provider = lm.getBestProvider(criteria, true);
			//第一个参数 位置提供者 第二个参数 最短更新时间 第三参数 最短的更新的距离
			listener = new MyListener();
			lm.requestLocationUpdates(provider, 0, 0, listener);

		}
		@Override
		protected void onDestroy() {
			lm.removeUpdates(listener);
			super.onDestroy();
		}

		private class MyListener implements LocationListener{

			/**
			 * 当位置改变的时候
			 */
			@Override
			public void onLocationChanged(Location location) {
				float accuracy = location.getAccuracy();
				double wlong = location.getLatitude(); //纬度
				double jlong = location.getLongitude(); //经度

				TextView tv = new TextView(getApplicationContext());
				tv.setText("经度:"+jlong+"\n"+"纬度:"+wlong+"\n"+ accuracy);
				setContentView(tv);

			}
			/**
			 * 某一个位置提供者的状态发生改变的时候调用的方法
			 */
			@Override
			public void onStatusChanged(String provider, int status, Bundle extras) {

			}

			@Override
			public void onProviderEnabled(String provider) {

			}

			@Override
			public void onProviderDisabled(String provider) {

			}
		}
	}
	```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 