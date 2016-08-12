Android开发中的MVP模式详解
===

[MVC、MVP、MVVM介绍](https://github.com/CharonChui/AndroidNote/blob/master/Java%E5%9F%BA%E7%A1%80/MVC%E4%B8%8EMVP%E5%8F%8AMVVM.md)

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_mvp.jpg?raw=true)

在`Android`开发中，如果不注重架构的话，`Activity`类就会变得愈发庞大。这是因为在`Android`开发中`View`和其他的线程可以共存于`Activity`内。那最大的问题是什么呢？ 其实就是`Activity`中同时存在业务逻辑和`UI`逻辑。这导致增加了单元测试和维护的成本。

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


上面只是抛砖引玉。`MVP`的有点十分明显，就是代码解耦、可以让逻辑清晰，但是同样它也会有缺点，它的缺点就是项目的复杂程度会增加，项目中会多出很多类。
之前很多人都在讨论该如何去正确的设计使用`MVP`来避免它的缺点，众说纷纭，很多人讨论的你死我活。直到`Google`发布了`MVP架构蓝图`，大家才意识到这才是规范。     

项目地址:[android-architecture](https://github.com/googlesamples/android-architecture)
`Google`将该项目命名为`Android`的架构蓝图，我想从名字上已可以看穿一切。

在它的官方介绍中是这样说的:   

> The Android framework offers a lot of flexibility when it comes to defining how to organize and architect an Android app. This freedom, whilst very valuable, can also result in apps with large classes, inconsistent naming and architectures (or lack of) that can make testing, maintaining and extending difficult.

> Android Architecture Blueprints is meant to demonstrate possible ways to help with these common problems. In this project we offer the same application implemented using different architectural concepts and tools.

> You can use these samples as a reference or as a starting point for creating your own apps. The focus here is on code structure, architecture, testing and maintainability. However, bear in mind that there are many ways to build apps with these architectures and tools, depending on your priorities, so these shouldn't be considered canonical examples. The UI is deliberately kept simple.



已完成的示例:  

- todo-mvp/ - Basic Model-View-Presenter architecture.
- todo-mvp-loaders/ - Based on todo-mvp, fetches data using Loaders.
- todo-mvp-databinding/ - Based on todo-mvp, uses the Data Binding Library.
- todo-mvp-clean/ - Based on todo-mvp, uses concepts from Clean Architecture.
- todo-mvp-dagger/ - Based on todo-mvp, uses Dagger2 for Dependency Injection
- todo-mvp-contentproviders/ - Based on todo-mvp-loaders, fetches data using Loaders and uses Content Providers

正在进行中的示例:   

- dev-todo-mvp-rxjava/ - Based on todo-mvp, uses RxJava for concurrency and data layer abstraction.


我们接下来就用`todo-mvp`来进行分析，这个应用非常简单，主要有以下几个功能:     

- 列表页:展示所有的`todo`项   
- 添加页:添加`todo`项
- 详情页:查看`todo`项的详情
- 统计页:查看当前所有已完成`todo`及未完成项的统计数据     







---


- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
