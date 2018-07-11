Android6.0权限系统
===

`Android`权限系统是一个非常重要的安全问题，因为它只有在安装时会询问一次。一旦软件本安装之后，应用程序可以在用户毫不知情的情况下使用这些权限来获取所有的内容。     

很多坏蛋会通过这个安全缺陷来收集用户的个人信息并使用它们来做坏事的情况就不足为奇了。    

`Android`团队也意识到了这个问题。在经过了7年后，权限系统终于被重新设置了。从`Anroid 6.0(API Level 23)`开始，应用程序在安装时不会被授予任何权限，取而代之的是在运行时应用回去请求用户授予对应的权限。这样可以让用户能更好的去控制应用功能。例如，一个用户可能会同一个拍照应用使用摄像头的权限，但是不同授权它获取设备位置的权限。用户可以在任何时候通过去应用的设置页面来撤销授权。    

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/runtimepermission.jpg?raw=true)  

注意:上面请求权限的对话框不会自动弹出。开发者需要手动的调用。如果开发者调用了一些需要权限的功能，但是用户又拒绝授权的话，应用就会`crash`。   

系统权限被分为两类，`normal`和`dangerous`:    

- `Normal Permissions`不需要用户直接授权，如果你的应用在清单文件中声明了`Normal Permissions`，系统会自动授权该权限。    
- `Dangerous Permissions`可以让应用获取用户的私人数据。如果你的应用在清单文件中申请了`Dangerous Permissions`，那就必须要用户来授权给应用。   

`Normal Permissions`:    
`Normal Permission`是当用户安装或更新应用时，系统将授予应用所请求的权限，又称为`PROTECTION_NORMAL`（安装时授权的一类基本权限）。该类权限只需要在`manifest`文件中声明即可，安装时就授权，不需要每次使用时进行检查，而且用户不能取消以上授权。   


- ACCESS_LOCATION_EXTRA_COMMANDS
- ACCESS_NETWORK_STATE
- ACCESS_NOTIFICATION_POLICY
- ACCESS_WIFI_STATE
- BLUETOOTH
- BLUETOOTH_ADMIN
- BROADCAST_STICKY
- CHANGE_NETWORK_STATE
- CHANGE_WIFI_MULTICAST_STATE
- CHANGE_WIFI_STATE
- DISABLE_KEYGUARD
- EXPAND_STATUS_BAR
- GET_PACKAGE_SIZE
- INSTALL_SHORTCUT
- INTERNET
- KILL_BACKGROUND_PROCESSES
- MODIFY_AUDIO_SETTINGS
- NFC
- READ_SYNC_SETTINGS
- READ_SYNC_STATS
- RECEIVE_BOOT_COMPLETED
- REORDER_TASKS
- REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
- REQUEST_INSTALL_PACKAGES
- SET_ALARM
- SET_TIME_ZONE
- SET_WALLPAPER
- SET_WALLPAPER_HINTS
- TRANSMIT_IR
- UNINSTALL_SHORTCUT
- USE_FINGERPRINT
- VIBRATE
- WAKE_LOCK
- WRITE_SYNC_SETTINGS



下图为`Dangerous permissions and permission groups`:  

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/permgroup.png?raw=true)     


在所有的`Android`版本中，你的应用都需要在`manifest`文件中声明`normal`和`dangerous`权限。然而声明所影响的效果会因系统版本和你应用的`target SDK lever`有关:     

- 如果设备运行的是`Android 5.1`或者之前的系统，或者应用的`targetSdkVersion`是22或者之前版本：如果你在`manifest`中声明了`dangerous permission`，用户需要在安装应用时授权这些权限。如果用户不授权这些权限，系统就不会安装该应用。(我试了下发现即使`targetSdkVersion`是22及以下，在6.0的手机上时，如果你安装时你不同意一些权限，也仍然可以安装的)  
- 如果设备运行的是`Android 6.0`或者更高的系统，并且你应用的`targetSdkVersion`是23或者更高:应用必须在`manifest`文件中申请这些权限，而且必须要在运行时对所需要的`dangerous permission`申请授权。用户可以统一或者拒绝授权，并且及时用户拒绝了授权，应用在无法使用一些功能的情况下也要保证能继续运行。     

也就是说新的运行时权限仅当我们设置`targetSdkVersion`是23及以上时才会起作用。      
如果你的`targtSdkVersion`低于23，那将被认为该应用没有经过`android 6.0`的测试，当该应用被安装到了6.0的手机上时，仍然会使用之前的旧权限规则，在安装时会提示所有需要的权限(这样做是有道理的，不然对于之前开发的应用，我们都要立马去修改让它适应6.0，来不及的话就导致6.0的手机都无法使用了，显然Android开发团队不会考虑不到这种情况)，当然用户可以在安装的界面不允许一些权限，那当程序使用到了这些权限时，会崩溃吗？答案是在`Android 6.0`及以上的手机会直接`crash`，但是在`23`之前的手机上不会`crash`。     

所以如果你的应用没有支持运行时权限的功能，那千万不要讲`targetSdkVersion`设置为23，否则就麻烦了。  

> 注意:从`Android 6.0(API Level 23)`开始，即使应用的`targetSdkVersion`是比较低的版本，但是用户仍然可以在任何时候撤销对应用的授权。所以不管应用的`targetSdkVerison`是什么版本，你都要测试你的应用在不能获取权限时能不能正常运行。 

下面介绍下如何使用`Android Support Library`来检查和请求权限。`Android`框架在`6.0`开始也提供了相同的方法。然而使用`support`包会比较简单，因为这样你就不需要在请求方法时判断当前的系统版本。(后面说的这几个类都是`android.support.v4`中的)    

### 检查权限    

如果应用需要使用`dangerous permission`，在任何时候执行需要该权限的操作时你都需要检查是否已经授权。用户可能会经常取消授权，所以即使昨天应用已经使用了摄像头，这也不能保证今天仍然有使用摄像头的权限。    

为了检查是否可以使用该权限，调用`ContextCompat.checkSelfPermission()`。   
例如:    
```java
// Assume thisActivity is the current activity
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.permission.WRITE_CALENDAR);
```
如果应用有该权限，该方法将返回`PackageManager.PERMISSION_GRANTED`，应用可以进行相关的操作。如果应用不能使用该权限，该方法将返回`PERMISSION_DENIED`，这是应用将必须要向用户申请该权限。    


### 申请使用权限

如果应用需要使用清单文件中申明的`dangerous permission`，它必须要向用户申请来授权。`Android`提供了几种申请授权的方法。使用这些方法时将会弹出一个标准的系统对话框，该对话框不能自定义。     

##### 说明为什么应用需要使用这些权限      

在一些情况下，你可能需要帮助用力理解为什么需要该权限。例如，一个用户使用了一个照相的应用，用户不会奇怪为什么应用申请使用摄像头的权限，但是用户可能会不理解为什么应用需要获取位置或者联系人的权限。在请求一个权限之前，你需要该用户一个说明。一定要切记不要通过说明来压倒用户。如果你提供了太多的说明，用户可能会感觉沮丧并且会卸载它。   

一个你需要提供说明的合适时机就是在用户之前已经不同意授权该权限的情况下。如果一个用户继续尝试使用需要权限的功能时，但是之前确禁止了该权限的请求，这就可能是因为用户不理解为什么该功能需要使用该权限。在这种情况下，提供一个说明是非常合适的。  

为了能找到用户可能需要说明的情况，`android`提供了一个工具类方法`ActivityCompat.shouldShowRequestPermissionRationale().`。如果应用之前申请了该权限但是用户拒绝授权后该方法会返回`true`。(在Android 6.0之前调用的时候会直接返回false)     

> 注意:如果用户之前拒绝了权限申请并且选择了请求权限对话框中的`Don’t ask again`选项，该方法就会返回`false`。如果设备策略禁止了该应用使用该权限，该方法也会返回`false`。(我测试的时候发现请求权限的对话框中并没有`Don’t asdk again`这一项)

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/request_permission_dialog.png?raw=true" width="100%" height="100%">

##### 申请需要的权限      

如果应用没有所需的权限时，应用必须调用`ActivityCompat.requestPermissions (Activity activity, 
                String[] permissions, 
                int requestCode)`方法来申请对用的权限。参数传递对应所需的权限以及一个整数型的`request code`来标记该权限申请。 该方法是异步的:该方法会立即返回，在用户响应了请求权限的对话框之后，系统会调用对用的回调方法来通知结果，并且会传递在`reqeustPermissions()`方法中的`request code`。(在Android 6.0之前调用的时候会直接去调用`onRequestPermissionsResult()`的回调方法)         
如图:    
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/requestpermission.jpg?raw=true)  


下面是检查是否读取联系人权限，并且在必要时申请权限的代码:    

```java
// Here, thisActivity is the current activity
if (ContextCompat.checkSelfPermission(thisActivity,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {

    // Should we show an explanation?
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.READ_CONTACTS)) {

        // Show an expanation to the user *asynchronously* -- don't block
        // this thread waiting for the user's response! After the user
        // sees the explanation, try again to request the permission.

    } else {

        // No explanation needed, we can request the permission.

        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);

        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
    }
}
```

> 注意:当调用`requestPermissions()`方法时，系统会显示一个标准的对话框。应用不能指定或者改变该对话框。如果你想提供一些信息或者说明给用户，你需要在调用`requestPermissions()`之前处理。    

##### 处理请求权限的的结果     

如果应用申请权限，系统会显示一个对话框。当用户相应后，系统会调用应用中的`onRequestPermissionsResult (int requestCode, 
                String[] permissions, 
                int[] grantResults)`方法并传递用户的操作结果。在应用中必须要重写该方法来查找授权了什么权限。该回调方法会传递你在`requestPermisssions()`方法中传递的`request code`。直接在`Activity`或者`Fragment`中重写`onRequestPermissionsResult()`方法即可。例如，申请`READ_CONTACTS`的权限可能会有下面的回到方法:     

```java
@Override
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted, yay! Do the
                // contacts-related task you need to do.

            } else {

                // permission denied, boo! Disable the
                // functionality that depends on this permission.
            }
            return;
        }

        // other 'case' lines to check for other
        // permissions this app might request
    }
}

```

系统提示的对话框会描述应用所需的`permission groud`。它不会列出特定的权限。例如，如果你申请了`READ_CONTACTS`权限，系统的对话框只会说你的应用需要获取设备的联系人信息。用户只需要授权每个`permission group`一次。如果你应用需要申请其他任何一个在该`permission group`中的权限时，系统会自动授权。在申请这些授权时，系统会像用户明确通过系统对话框统一授权时一样去调用`onRequestPermissionsResult()`方法并且传递`PERMISSION_GRANTED`参数。    

> 注意:虽然用户已经授权了同一`permission group`中其他的任何权限，但是应用仍然需要明确申请每个需要的权限。例外，`permission group`中的权限在以后可能会发生变化。    

例如，假设在应用的`manifest`文件中同时声明了`READ_CONTACTS`和`WRITE_CONTACTS`权限。如果你申请`READ_CONTACTS`权限而且用户同意了该权限，如果你想继续申请`WRITE_CONTACTS`权限，系统不会与用户有任何交互就会直接进行授权。     

如果用户拒绝了一个权限申请，你的应用进行合适的处理。例如，你的应用可能显示一个对话框来表明无法执行用户请求的需要该权限的操作。

如果系统向用户申请权限授权，用户选择了让系统以后不要再申请该权限。 在这种情况下，应用在任何时间调用`reqeustPermissions()`方法来再次申请权限时，系统都会直接拒绝该请求。系统会直接调用`onRequestPermissionResult()`回调方法并且传递`PERMISSION_DENIED`参数，和用户明确拒绝应用申请该权限时一样。 这就意味着在你调用`requestPermissions()`方法是，你无法确定是否会和用户有直接的交互操作。   


示例代码:     
```java

final private int REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS = 124;
  
private void insertDummyContactWrapper() {
    List<String> permissionsNeeded = new ArrayList<String>();
  
    final List<String> permissionsList = new ArrayList<String>();
    if (!addPermission(permissionsList, Manifest.permission.ACCESS_FINE_LOCATION))
        permissionsNeeded.add("GPS");
    if (!addPermission(permissionsList, Manifest.permission.READ_CONTACTS))
        permissionsNeeded.add("Read Contacts");
    if (!addPermission(permissionsList, Manifest.permission.WRITE_CONTACTS))
        permissionsNeeded.add("Write Contacts");
  
    if (permissionsList.size() > 0) {
        if (permissionsNeeded.size() > 0) {
            // Need Rationale
            String message = "You need to grant access to " + permissionsNeeded.get(0);
            for (int i = 1; i < permissionsNeeded.size(); i++)
                message = message + ", " + permissionsNeeded.get(i);
            showMessageOKCancel(message,
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                                    REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
                        }
                    });
            return;
        }
        requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
        return;
    }
  
    insertDummyContact();
}
  
private boolean addPermission(List<String> permissionsList, String permission) {
    if (checkSelfPermission(permission) != PackageManager.PERMISSION_GRANTED) {
        permissionsList.add(permission);
        // Check for Rationale Option
        if (!shouldShowRequestPermissionRationale(permission))
            return false;
    }
    return true;
}
```


上面讲到的都是`Activity`中的使用方法，那`Fragment`中怎么授权呢？
如果在`Fragment`中使用，用`v13`包中的`FragmentCompat.requestPermissions()`和`FragmentCompat.shouldShowRequestPermissionRationale()`。 



在`Fragment`中申请权限，不要使用`ActivityCompat.requestPermissions`, 直接使用`Fragment.requestPermissions`方法，
否则会回调到`Activity`的`onRequestPermissionsResult`。但是虽然你使用`Fragment.requestPermissions`方法，也照样回调不到`Fragment.onRequestPermissionsResult`中。这是`Android`的`Bug`,[详见](https://code.google.com/p/android/issues/detail?can=2&start=0&num=100&q=&colspec=ID%20Status%20Priority%20Owner%20Summary%20Stars%20Reporter%20Opened&groupby=&sort=&id=189121)，`Google`已经在`23.3.0`修复了该问题，所以要尽快升级。

所以升级到`23.3.0`及以上就没问题了。如果不升级该怎么处理呢？就是在`Activity.onRequestPermissionsResult`方法中去手动调用每个`Fragment`的方法(当然你要判断下权限个数，不然申请一个权限的情况下会重复调用).    
```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    List<Fragment> fragments = getSupportFragmentManager().getFragments();
    if (fragments != null) {
        for (Fragment fragment : fragments) {
            fragment.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }
    }
}
```

我简单的写了一个工具类:    

```java
public class PermissionUtil {
    /**
     * 在调用需要权限的功能时使用该方法进行检查。
     *
     * @param activity
     * @param requestCode
     * @param iPermission
     * @param permissions
     */
    public static void checkPermissions(Activity activity, int requestCode, IPermission iPermission, String... permissions) {
        handleRequestPermissions(activity, requestCode, iPermission, permissions);
    }

    public static void checkPermissions(Fragment fragment, int requestCode, IPermission iPermission, String... permissions) {
        handleRequestPermissions(fragment, requestCode, iPermission, permissions);
    }

    public static void checkPermissions(android.app.Fragment fragment, int requestCode, IPermission iPermission, String... permissions) {
        handleRequestPermissions(fragment, requestCode, iPermission, permissions);
    }

    /**
     * 在Actvitiy或者Fragment中重写onRequestPermissionsResult方法后调用该方法。
     *
     * @param activity
     * @param requestCode
     * @param permissions
     * @param grantResults
     * @param iPermission
     */
    public static void onRequestPermissionsResult(Activity activity, int requestCode, String[] permissions,
                                                  int[] grantResults, IPermission iPermission) {
        requestResult(activity, requestCode, permissions, grantResults, iPermission);

    }

    public static void onRequestPermissionsResult(Fragment fragment, int requestCode, String[] permissions,
                                                  int[] grantResults, IPermission iPermission) {
        requestResult(fragment, requestCode, permissions, grantResults, iPermission);
    }

    public static void onRequestPermissionsResult(android.app.Fragment fragment, int requestCode, String[] permissions,
                                                  int[] grantResults, IPermission iPermission) {
        requestResult(fragment, requestCode, permissions, grantResults, iPermission);
    }

    public static <T> void requestPermission(T t, int requestCode, String... permission) {
        List<String> permissions = new ArrayList<>();
        for (String s : permission) {
            permissions.add(s);
        }
        requestPermissions(t, requestCode, permissions);
    }

    /**
     * 在检查权限后自己处理权限说明的逻辑后调用该方法，直接申请权限。
     *
     * @param t
     * @param requestCode
     * @param permissions
     * @param <T>
     */
    public static <T> void requestPermission(T t, int requestCode, List<String> permissions) {
        if (permissions == null || permissions.size() == 0) {
            return;
        }
        requestPermissions(t, requestCode, permissions);
    }

    public static boolean checkSelfPermission(Context context, String permission) {
        if (context == null || TextUtils.isEmpty(permission)) {
            throw new IllegalArgumentException("invalidate params: the params is null !");
        }

        context = context.getApplicationContext();

        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
            int result = ContextCompat.checkSelfPermission(context, permission);
            if (PackageManager.PERMISSION_DENIED == result) {
                return false;
            }
        }

        return true;
    }

    private static <T> void handleRequestPermissions(T t, int requestCode, IPermission iPermission, String... permissions) {
        if (t == null || permissions == null || permissions.length == 0) {
            throw new IllegalArgumentException("invalidate params");
        }
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
            Activity activity = getActivity(t);
            List<String> deniedPermissions = getDeniedPermissions(activity, permissions);
            if (deniedPermissions != null && deniedPermissions.size() > 0) {

                List<String> rationalPermissions = new ArrayList<>();
                for (String deniedPermission : deniedPermissions) {
                    if (ActivityCompat.shouldShowRequestPermissionRationale(activity,
                            deniedPermission)) {
                        rationalPermissions.add(deniedPermission);
                    }
                }

                boolean showRational = false;
                if (iPermission != null) {
                    showRational = iPermission.showRational(requestCode);
                }

                if (rationalPermissions.size() > 0 && showRational) {
                    if (iPermission != null) {
                        iPermission.onRational(requestCode, deniedPermissions);
                    }
                } else {
                    requestPermissions(t, requestCode, deniedPermissions);
                }
            } else {
                if (iPermission != null) {
                    iPermission.onGranted(requestCode);
                }
            }
        } else {
            if (iPermission != null) {
                iPermission.onGranted(requestCode);
            }
        }
    }

    @Nullable
    private static <T> Activity getActivity(T t) {
        Activity activity = null;
        if (t instanceof Activity) {
            activity = (Activity) t;
        } else if (t instanceof Fragment) {
            activity = ((Fragment) t).getActivity();
        } else if (t instanceof android.app.Fragment) {
            activity = ((android.app.Fragment) t).getActivity();
        }
        return activity;
    }

    @TargetApi(Build.VERSION_CODES.M)
    private static <T> void requestPermissions(T t, int requestCode, List<String> deniedPermissions) {
        if (deniedPermissions == null || deniedPermissions.size() == 0) {
            return;
        }
        // has denied permissions
        if (t instanceof Activity) {
            ((Activity) t).requestPermissions(deniedPermissions.toArray(new String[deniedPermissions.size()]), requestCode);
        } else if (t instanceof Fragment) {
            ((Fragment) t).requestPermissions(deniedPermissions.toArray(new String[deniedPermissions.size()]), requestCode);
        } else if (t instanceof android.app.Fragment) {
            ((android.app.Fragment) t).requestPermissions(deniedPermissions.toArray(new String[deniedPermissions.size()]), requestCode);
        }
    }

    private static List<String> getDeniedPermissions(Context context, String... permissions) {
        if (context == null || permissions == null || permissions.length == 0) {
            return null;
        }
        List<String> denyPermissions = new ArrayList<>();
        for (String permission : permissions) {
            if (!checkSelfPermission(context, permission)) {
                denyPermissions.add(permission);
            }
        }
        return denyPermissions;
    }


    private static <T> void requestResult(T t, int requestCode, String[] permissions,
                                          int[] grantResults, IPermission iPermission) {
        List<String> deniedPermissions = new ArrayList<>();
        for (int i = 0; i < grantResults.length; i++) {
            if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                deniedPermissions.add(permissions[i]);
            }
        }
        if (deniedPermissions.size() > 0) {
            if (iPermission != null) {
                iPermission.onDenied(requestCode);
            }
        } else {
            if (iPermission != null) {
                iPermission.onGranted(requestCode);
            }
        }
    }

}

interface IPermission {
    void onGranted(int requestCode);

    void onDenied(int requestCode);

    void onRational(int requestCode, List<String> permissions);

    /**
     * 是否需要提示用户该权限的作用，提示后需要再调用requestPermission()方法来申请。
     *
     * @return true 为提示，false为不提示
     */
    boolean showRational(int requestCode);
}

```

使用方法:    
```java
public class MainFragment extends Fragment implements View.OnClickListener {
    private Button mReqCameraBt;
    private Button mReqContactsBt;
    private Button mReqMoreBt;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_main, container, false);
        findView(view);
        initView();
        return view;
    }

    private void findView(View view) {
        mReqCameraBt = (Button) view.findViewById(R.id.bt_requestCamera);
        mReqContactsBt = (Button) view.findViewById(R.id.bt_requestContacts);
        mReqMoreBt = (Button) view.findViewById(R.id.bt_requestMore);
    }

    private void initView() {
        mReqCameraBt.setOnClickListener(this);
        mReqContactsBt.setOnClickListener(this);
        mReqMoreBt.setOnClickListener(this);
    }


    public static final int REQUEST_CODE_CAMERA = 0;
    public static final int REQUEST_CODE_CONTACTS = 1;
    public static final int REQUEST_CODE_MORE = 2;

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionUtil.onRequestPermissionsResult(this, requestCode, permissions, grantResults, mPermission);
    }

    public void requestCamera() {
        PermissionUtil.checkPermissions(this, REQUEST_CODE_CAMERA, mPermission, Manifest.permission.CAMERA);
    }

    public void requestReadContacts() {
        PermissionUtil.checkPermissions(this, REQUEST_CODE_CONTACTS, mPermission, Manifest.permission.READ_CONTACTS);
    }

    public void requestMore() {
        PermissionUtil.checkPermissions(this, REQUEST_CODE_MORE, mPermission, Manifest.permission.READ_CONTACTS, Manifest.permission.READ_CALENDAR, Manifest.permission.CALL_PHONE);
    }

    private void showPermissionTipDialog(final int requestCode, final List<String> permissions) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder.setMessage("I want you permissions");
        builder.setTitle("Hello Permission");
        builder.setPositiveButton("确认", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
                PermissionUtil.requestPermission(MainFragment.this, requestCode, permissions);
            }
        });
        builder.setNegativeButton("取消", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });
        builder.create().show();
    }

    public IPermission mPermission = new IPermission() {
        @Override
        public void onGranted(int requestCode) {
            Toast.makeText(getActivity(), "onGranted :" + requestCode, Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onDenied(int requestCode) {
            Toast.makeText(getActivity(), "onDenied :" + requestCode, Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onRational(int requestCode, List<String> permission) {
            showPermissionTipDialog(requestCode, permission);
        }

        @Override
        public boolean showRational(int requestCode) {
            switch (requestCode) {
                case REQUEST_CODE_MORE:
                    return true;

                default:
                    break;
            }
            return false;
        }
    };

    @Override
    public void onClick(View v) {
        int id = v.getId();
        switch (id) {
            case R.id.bt_requestCamera:
                requestCamera();
                break;
            case R.id.bt_requestContacts:
                requestReadContacts();
                break;
            case R.id.bt_requestMore:
                requestMore();
                break;
        }
    }
}

```

		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 