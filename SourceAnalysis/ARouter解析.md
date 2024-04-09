# ARouter解析

[ARouter](https://github.com/alibaba/ARouter)

> 一个用于帮助 Android App 进行组件化改造的框架 —— 支持模块间的路由、通信、解耦

简单的说: 它是一个路由系统 ——— 给无依赖的组件提供通信和路由的能力。

举个例子： 
你过节了你想写一个明信片递给远方的朋友，那你就需要通过邮局(ARoter)来把明信片派送给你的朋友(你和你的朋友相当于两个组件)。

使用ARouter在进行Activity跳转非常简单:    

- 初始化ARouter  `ARouter.init(mApplication); // 尽可能早，推荐在Application中初始化`
- 添加注解@Route  
	```java
	// 在支持路由的页面上添加注解(必选)
	// 这里的路径需要注意的是至少需要有两级，/xx/xx
	@Route(path = "/test/activity")
	public class YourActivity extend Activity {
	    ...
	}
	```
- 发起路由  `ARouter.getInstance().build("/test/activity").navigation();`



ARouter框架能将多个服务提供者隔离，减少相互之间的依赖。其实现的流程和我们平常的快递物流管理很类似，每一个具体的快递包裹就是一个独立的服务提供者（IProvider），每一个快递信息单就是一个RouteMeta对象，客户端就是快递的接收方，而使用@Route注解中的path就是快递单号。在初始化流程中，主要完成的工作就是将所有注册的快递信息表都在物流中心（LogisticsCenter）注册，并将数据存储到数据仓库中（Warehouse）。

作者：魔焰之
链接：https://www.jianshu.com/p/11006054f156
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


















---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
