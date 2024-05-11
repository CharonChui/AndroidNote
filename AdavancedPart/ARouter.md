ARouter
---


[ARouter](https://github.com/alibaba/ARouter)
一个用于帮助 Android App 进行组件化改造的框架 —— 支持模块间的路由、通信、解耦




```java
Intent intent = new Intent(mContext, XxxActivity.class);
intent.putExtra("key","value");
startActivity(intent);
        
Intent intent = new Intent(mContext, XxxActivity.class);
intent.putExtra("key","value");
startActivityForResult(intent, 666);
```
在未使用ARouter路由框架之前的原生页面跳转方式。 


## 原生路由方案的缺点 

1. 显式: 直接的类依赖，耦合严重
2. 隐式: 会在Manifest文件中进行集中管理，写作困难


## ARouter的优势

1. 使用注解，实现了映射关系自动注册与分布式路由管理
2. 编译期间处理注解，并生成映射文件，没有使用反射，不影响运行时性能
3. 映射关系按组分类，分级管理，按需初始化。
4. 灵活的降级策略，每次跳转都会回调跳转结果，避免startActivity()一旦失败会抛出异常
5. 自定义拦截器，自定义拦截顺序，可以对路由进行拦截，比如登录判断和埋点处理
6. 支持依赖注入，可单独作为依赖注入框架使用，从而实现跨模块API调用
7. 支持直接解析标准url进行跳转，并自动注入参数到目标页面中
8. 支持多模块使用，支持组件化开发



## 基本使用

添加依赖并初始化后的基本使用:     

```java
// 在支持路由的页面上添加注解(必选)
// 这里的路径需要注意的是至少需要有两级，/xx/xx
@Route(path = "/test/activity")
public class YourActivity extend Activity {
    ...
}

// 1. 应用内简单的跳转(通过URL跳转在'进阶用法'中)
ARouter.getInstance().build("/test/activity").navigation();

// 2. 跳转并携带参数
ARouter.getInstance().build("/test/1")
    .withLong("key1", 666L)
    .withString("key3", "888")
    .withObject("key4", new Test("Jack", "Rose"))
    .navigation();
```








