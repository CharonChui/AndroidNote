Android开发中的MVP模式详解
===

[MVC、MVP、MVVM介绍](https://github.com/CharonChui/AndroidNote/blob/master/Java%E5%9F%BA%E7%A1%80/MVC%E4%B8%8EMVP%E5%8F%8AMVVM.md)


在`Android`开发中，如果不注重架构的话，`Activity`类就会变得愈发庞大。这是因为在`Android`开发中`View`和其他的线程可以共存于`Activity`内。那最大的问题是什么呢？ 其实就是**`Activity`中同事存在业务逻辑和`UI`逻辑。这导致增加了单元测试和维护的成本。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/activity_is_god.png?raw=true)
    	
这就是为什么要清晰架构的原因之一。不仅是因为`Activity`类变得臃肿，也是其他的一些问题，例如`Activity`和`Fragment`相结合时的生命周期、数据绑定等等。   

###MVP简介

`MVP(Model,View,Presenter)`      

- `View`：负责处理用户时间和视图展现。在`Android`中就可能是`Activity`或者`Fragment`。
- `Model`： 负责数据访问。数据可以是从接口或者本地数据库中获取。
- `Presenter`: 负责连接`View`和`Model`。

用一句话来说:`MVP`其实就是面向接口编程，`V`实现接口，`P`使用接口。

清晰的架构:    

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mvp_a.png?raw=true)


举个栗子:
在`Android Studio`中新建一个`Activity`，系统提供了`LoginActivity`，直接用它是极好的。    

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/loginactivity.png?raw=true)

不得不说，`Material Design`的效果真是美美哒！

好，那我们就用用户登录页来按照`MVP`的模式实现一下:     

- M: 很显然Model应该是`User`类。
- V: `View`就是`LoginActivity`。
- P: P那我们一会就创建一个`LoginPresenter`类。

齐了，那接下来就详细分析下他们这三部分:   

- `User`: 应该有`email`, `password`, `boolean login(email, password)`。
- `LoginActivity`:点击登录应该要出`loading`页。登录成功后要进入下一个页面。如果登录失败应该弹`toast`提示。那就需要`void showLoading()`，`void hideLoading()`，`void showErrorTip()`,`void doLoginSuccess()`这四个方法。
- `LoginPresenter`:这是`Model`和`View`的桥梁。他需要做的处理业务逻辑，直接与`Model`打交道，然后将`UI`的逻辑交给`LoginActivity`处理。
那怎么做呢？ 按照我上面总结的那一句话，、`MVP`其实就是面向接口编程，`V`实现接口，`P`使用接口。很显然我们需要提供一个接口。那就新建一个`ILoginView`的接口。这里面有哪些方法呢？ 当然是上面我们在分析`LoginActiity`时提出的那四个方法。这样`LoginActivity`直接实现`ILoginView`接口就好。


开始做:   

- 先把`Model`做好吧，创建`User`类。  
    ```java
    public class User {
        private String email;
        private String password;
    
        public User(String email, String password) {
            this.email = email;
            this.password = password;
        }
    
        public boolean login() {
            // do login request..
            return true;
        }
    }
    ```
- 创建`ILoginView`接口，定义登录所需要的`ui`逻辑。    
    ```java
    public interface ILoginView {
        void showLoading();
        void hideLoading();
        void showErrorTip();
        void doLoginSuccess();
    }
    ```

- 创建`LoginPresenter`类，使用`ILoginView`接口，那该类主要有什么功能呢？ 它主要是处理业务逻辑的，
    对于登录的话，当然是用户在`UI`页面输入邮箱和密码，然后`Presenter`去开线程、请求接口。然后得到登录结果再去让`UI`显示对应的视图。那自然就是有一个`void login(String email, String passowrd)`的方法了  
    ```java
    public class LoginPresenter {
        private ILoginView mLoginView;
    
        public LoginPresenter(ILoginView loginView) {
            mLoginView = loginView;
        }
    
        public void login(String email, String password) {
            if (TextUtils.isEmpty(email) || TextUtils.isEmpty(password)) {
                //
                mLoginView.showErrorTip();
                return;
            }
            mLoginView.showLoading();
            User user = new User(email, password);
    
            // do network request....
            // ....
            onSuccess() {
                boolean login = user.login();
                if (login) {
                    mLoginView.doLoginSuccess();
                } else {
                    mLoginView.showErrorTip();
                }
                mLoginView.hideLoading();
            }
    
            onFailde() {
                mLoginView.showErrorTip();
                mLoginView.hideLoading();
            }
        }
    }		
    ```
- 创建`LoginActivity`，实现`ILoginView`的接口，然后内部调用`LoginPresenter`来处理业务逻辑。
    ```java
    public class LoginActivity extends AppCompatActivity implements ILoginView {
        private LoginPresenter mLoginPresenter;
    
        private AutoCompleteTextView mEmailView;
        private EditText mPasswordView;
        private View mProgressView;
        private View mLoginButton;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_login);
            mEmailView = (AutoCompleteTextView) findViewById(R.id.email);
            mPasswordView = (EditText) findViewById(R.id.password);
            mLoginButton = findViewById(R.id.email_sign_in_button);
            mProgressView = findViewById(R.id.login_progress);
    
            mLoginPresenter = new LoginPresenter(this);
    
            mLoginButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mLoginPresenter.login(mEmailView.getText().toString().trim(), mPasswordView.getText().toString().trim());
                }
            });
        }
    
        @Override
        public void showLoading() {
            mProgressView.setVisibility(View.VISIBLE);
        }
    
        @Override
        public void hideLoading() {
            mProgressView.setVisibility(View.GONE);
        }
    
        @Override
        public void showErrorTip() {
            Toast.makeText(this, "login faled", Toast.LENGTH_SHORT).show();
        }
    
        @Override
        public void doLoginSuccess() {
            Toast.makeText(this, "login success", Toast.LENGTH_SHORT).show();
        }
    }
    ```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 