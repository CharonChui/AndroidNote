Android开发中的MVP模式详解
===

[MVC、MVP、MVVM介绍][1]

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_mvp.jpg?raw=true)

在`Android`开发中，如果不注重架构的话，`Activity`类就会变得愈发庞大。这是因为在`Android`开发中`View`和其他的线程可以共存于`Activity`内。那最大的问题是什么呢？ 其实就是`Activity`中同时存在业务逻辑和`UI`逻辑。这导致增加了单元测试和维护的成本。

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/activity_is_god.png?raw=true)
        
这就是为什么要清晰架构的原因之一。不仅是因为`Activity`类变得臃肿，也是其他的一些问题，例如`Activity`和`Fragment`相结合时的生命周期、数据绑定等等。   

### MVP简介

`MVP(Model,View,Presenter)`      

- `View`：负责处理用户时间和视图展现。在`Android`中就可能是`Activity`或者`Fragment`。
- `Model`： 负责数据访问。数据可以是从接口或者本地数据库中获取。
- `Presenter`: 负责连接`View`和`Model`。

用一句话来说:`MVP`其实就是面向接口编程，`V`实现接口，`P`使用接口。

清晰的架构:    

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/mvp_a.png?raw=true)


举个栗子:
在`Android Studio`中新建一个`Activity`，系统提供了`LoginActivity`，直接用它是极好的。    

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/loginactivity.png?raw=true" width="50%" height="50%" />

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


上面只是抛砖引玉。`MVP`的优点十分明显，就是代码解耦、可以让逻辑清晰，但是同样它也会有缺点，它的缺点就是项目的复杂程度会增加，项目中会多出很多类。
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
- todo-mvp-rxjava/ - Based on todo-mvp, uses RxJava for concurrency and data layer abstraction.


我们接下来就用`todo-mvp`来进行分析，这个应用非常简单，主要有以下几个功能:     

- 列表页:展示所有的`todo`项   
- 添加页:添加`todo`项
- 详情页:查看`todo`项的详情
- 统计页:查看当前所有已完成`todo`及未完成项的统计数据     

代码并不多:   


<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/google_mvp.png?raw=true" width="50%" height="50%" />

功能也比较简单:    

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/google_todo_main.png?raw=true" width="40%" height="40%" />    <img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/google_todp_add.png?raw=true" width="40%" height="40%" />
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/google_todo_list.png?raw=true" width="40%" height="40%" />          <img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/google_todo_delete.png?raw=true" width="40%" height="40%" />

我们先从两个`Base`类开始看，分别是`BaseView`以及`BasePresenter`类。 

`BaseView`类:   

```java
public interface BaseView<T> {

    void setPresenter(T presenter);

}
```
`BaseView`中的`setPresenter()`是将`Presenter`的实例传入到`View`中。    
`BasePresenter`类:   
```java
public interface BasePresenter {

    void start();

}
```
`BasePresenter`中只有一个`start()`方法，从名字上我们就能看出他的作用。 

接下来继续看一下项目的入口`Activity`，`TaskDetailActivity`的实现: 
```java
public class TaskDetailActivity extends AppCompatActivity {

    public static final String EXTRA_TASK_ID = "TASK_ID";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.taskdetail_act);

        // Set up the toolbar.
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        ActionBar ab = getSupportActionBar();
        ab.setDisplayHomeAsUpEnabled(true);
        ab.setDisplayShowHomeEnabled(true);

        // Get the requested task id
        String taskId = getIntent().getStringExtra(EXTRA_TASK_ID);
        
        TaskDetailFragment taskDetailFragment = (TaskDetailFragment) getSupportFragmentManager()
                .findFragmentById(R.id.contentFrame);

        if (taskDetailFragment == null) {
            // 创建对应的Fragment
            taskDetailFragment = TaskDetailFragment.newInstance(taskId);

            ActivityUtils.addFragmentToActivity(getSupportFragmentManager(),
                    taskDetailFragment, R.id.contentFrame);
        }

        // Create the presenter，因为Presenter是M和V的连接桥，所以在第二个参数中会传入TasksRepository
        new TaskDetailPresenter(
                taskId,
                Injection.provideTasksRepository(getApplicationContext()),
                taskDetailFragment);
    }

    @Override
    public boolean onSupportNavigateUp() {
        onBackPressed();
        return true;
    }
}
```

可以看到这里`Activity`相当于一个管理类，里面控制着`MVP`中的`V`和`P`,上面的代码主要做了两部分:  

- 创建对应的`Fragment`
- 创建对应的`Presenter`

这里分别看一下`TaskDetailFragment`和`TaskDetailPresenter`的源码:   
```java
public class TaskDetailFragment extends Fragment implements TaskDetailContract.View {
    private TaskDetailContract.Presenter mPresenter;

    public static TaskDetailFragment newInstance(@Nullable String taskId) {
        Bundle arguments = new Bundle();
        arguments.putString(ARGUMENT_TASK_ID, taskId);
        TaskDetailFragment fragment = new TaskDetailFragment();
        fragment.setArguments(arguments);
        return fragment;
    }

    @Override
    public void onResume() {
        super.onResume();
        // 调用Presenter.start()方法
        mPresenter.start();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        ....
    }

    @Override
    public void setPresenter(@NonNull TaskDetailContract.Presenter presenter) {
        // 将Prsenter传递给V中 
        mPresenter = checkNotNull(presenter);
    }

    ....
}
```
及
```java
public class TaskDetailPresenter implements TaskDetailContract.Presenter {
    private final TasksRepository mTasksRepository;

    private final TaskDetailContract.View mTaskDetailView;

    @Nullable
    private String mTaskId;

    // 把M和V传递进来
    public TaskDetailPresenter(@Nullable String taskId,
                               @NonNull TasksRepository tasksRepository,
                               @NonNull TaskDetailContract.View taskDetailView) {
        mTaskId = taskId;
        mTasksRepository = checkNotNull(tasksRepository, "tasksRepository cannot be null!");
        mTaskDetailView = checkNotNull(taskDetailView, "taskDetailView cannot be null!");
        // 将该Presenter设置给V
        mTaskDetailView.setPresenter(this);
    }

    @Override
    public void start() {
        // 调用获取数据的方法
        openTask();
    }
    ....
}
```

可以看到上面分别实现了`TaskDetailContract`类中的`Presenter`和`View`接口，而不是`BasePresenter`和`BaseView`中的接口，那这个`TaskDetailContract`类是什么呢？ 
```java
/**
 * This specifies the contract between the view and the presenter.
 */
public interface TaskDetailContract {

    interface View extends BaseView<Presenter> {

        void setLoadingIndicator(boolean active);

        void showMissingTask();

        void hideTitle();

        void showTitle(String title);

        void hideDescription();

        void showDescription(String description);

        void showCompletionStatus(boolean complete);

        void showEditTask(String taskId);

        void showTaskDeleted();

        void showTaskMarkedComplete();

        void showTaskMarkedActive();

        boolean isActive();
    }

    interface Presenter extends BasePresenter {

        void editTask();

        void deleteTask();

        void completeTask();

        void activateTask();
    }
}
```
按照上面描述的介绍可以看出通过这个连接类将`VP`联系到了一起，它统一管理`view`和`presenter`中的所有接口，这样比起分开写会更加清晰。

分析完`VP`之后我们继续看一下`TaskDetailPresenter`类，因为`Presenter`是`MV`的桥梁。
```java
public TaskDetailPresenter(@Nullable String taskId,
                               @NonNull TasksRepository tasksRepository,
                               @NonNull TaskDetailContract.View taskDetailView) {
        mTaskId = taskId;
        mTasksRepository = checkNotNull(tasksRepository, "tasksRepository cannot be null!");
        mTaskDetailView = checkNotNull(taskDetailView, "taskDetailView cannot be null!");

        mTaskDetailView.setPresenter(this);
    }

    @Override
    public void start() {
        openTask();
    }

    private void openTask() {
        if (Strings.isNullOrEmpty(mTaskId)) {
            mTaskDetailView.showMissingTask();
            return;
        }
        // 控制V显示UI
        mTaskDetailView.setLoadingIndicator(true);
    // 通过M去获取数据
        mTasksRepository.getTask(mTaskId, new TasksDataSource.GetTaskCallback() {
            @Override
            public void onTaskLoaded(Task task) {
                // The view may not be able to handle UI updates anymore
                if (!mTaskDetailView.isActive()) {
                    return;
                }
                mTaskDetailView.setLoadingIndicator(false);
                if (null == task) {
                    mTaskDetailView.showMissingTask();
                } else {
                    showTask(task);
                }
            }

            @Override
            public void onDataNotAvailable() {
                // The view may not be able to handle UI updates anymore
                if (!mTaskDetailView.isActive()) {
                    return;
                }
                mTaskDetailView.showMissingTask();
            }
        });
    }

```
在它的构造函数中传入了一个`TasksRepository`，这个就是`Model`层。而在该`Presenter`中的`start()`方法回去调用获取数据的方法。   

我们继续看一下`M`层`TasksRepository`的实现:    
```java
/**
 * Concrete implementation to load tasks from the data sources into a cache.
 * <p>
 * For simplicity, this implements a dumb synchronisation between locally persisted data and data
 * obtained from the server, by using the remote data source only if the local database doesn't
 * exist or is empty.
 */
public class TasksRepository implements TasksDataSource {
    ....
}
```

先看一下`TasksDataSource`接口:   

```java
/**
 * Main entry point for accessing tasks data.
 * <p>
 * For simplicity, only getTasks() and getTask() have callbacks. Consider adding callbacks to other
 * methods to inform the user of network/database errors or successful operations.
 * For example, when a new task is created, it's synchronously stored in cache but usually every
 * operation on database or network should be executed in a different thread.
 */
public interface TasksDataSource {

    interface LoadTasksCallback {

        void onTasksLoaded(List<Task> tasks);

        void onDataNotAvailable();
    }

    interface GetTaskCallback {

        void onTaskLoaded(Task task);

        void onDataNotAvailable();
    }

    void getTasks(@NonNull LoadTasksCallback callback);

    void getTask(@NonNull String taskId, @NonNull GetTaskCallback callback);

    void saveTask(@NonNull Task task);

    void completeTask(@NonNull Task task);

    void completeTask(@NonNull String taskId);

    void activateTask(@NonNull Task task);

    void activateTask(@NonNull String taskId);

    void clearCompletedTasks();

    void refreshTasks();

    void deleteAllTasks();

    void deleteTask(@NonNull String taskId);
}

```

可以看出`M`层被赋予了数据获取的功能，与之前我们写的`M`层只定义实体对象截然不同，数据的增删改查都是`M`层实现的。`Presenter`只需要调用`M`层的方法即可。这样`model、presenter、view`都只处理各自的任务，此种实现确实是单一职责最好的诠释。

这里我们就以上面用到的`getTasks()`方法为例来介绍一下:   
```java
public void getTask(@NonNull final String taskId, @NonNull final GetTaskCallback callback) {
        checkNotNull(taskId);
        checkNotNull(callback);
        // 从缓存中获取数据
        Task cachedTask = getTaskWithId(taskId);

        // Respond immediately with cache if available
        if (cachedTask != null) {
            callback.onTaskLoaded(cachedTask);
            return;
        }

        // Load from server/persisted if needed.

        // Is the task in the local data source? If not, query the network.
        mTasksLocalDataSource.getTask(taskId, new GetTaskCallback() {
            @Override
            public void onTaskLoaded(Task task) {
                // Do in memory cache update to keep the app UI up to date
                if (mCachedTasks == null) {
                    mCachedTasks = new LinkedHashMap<>();
                }
                mCachedTasks.put(task.getId(), task);
                callback.onTaskLoaded(task);
            }

            @Override
            public void onDataNotAvailable() {
                // 服务器的数据源
                mTasksRemoteDataSource.getTask(taskId, new GetTaskCallback() {
                    @Override
                    public void onTaskLoaded(Task task) {
                        // Do in memory cache update to keep the app UI up to date
                        if (mCachedTasks == null) {
                            mCachedTasks = new LinkedHashMap<>();
                        }
                        mCachedTasks.put(task.getId(), task);
                        callback.onTaskLoaded(task);
                    }

                    @Override
                    public void onDataNotAvailable() {
                        callback.onDataNotAvailable();
                    }
                });
            }
        });
    }
```

上面的注释说的非常明白了，这里就不去仔细看了。


下面用一张图来总结一下:   

![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/google_todo_mvp.png?raw=true)


由于架构的引入，虽然代码量有了一定的上升，但是功能分离的非常清晰明确，而且每个部分都可以进行单独的测试，对于后期的扩展维护会更加简单容易。



[1]: https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/MVC%E4%B8%8EMVP%E5%8F%8AMVVM.md  "MVC、MVP、MVVM介绍"

---


- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
