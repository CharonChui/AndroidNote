MVC与MVP及MVVM
===

MVC
---

`MVC`是一种使用Model View Controller模型-视图-控制器设计创建`Web`应用程序的模式:   

- `Model`（模型）表示应用程序核心（比如数据库记录列表）是应用程序中用于处理应用程序数据逻辑的部分。
- `View`（视图）显示数据（数据库记录）是应用程序中处理数据显示的部分。
- `Controller`（控制器）处理输入（写入数据库记录）应用程序中处理用户交互的部分。

MVC 分层有助于管理复杂的应用程序，因为您可以在一个时间内专门关注一个方面。例如，您可以在不依赖业务逻辑的情况下专注于视图设计。同时也让应用程序的测试更加容易。
MVC 分层同时也简化了分组开发。不同的开发人员可同时开发视图、控制器逻辑和业务逻辑。

- 优点
    - 耦合性低
	- 重用性高
	- 可维护性高
	- 有利软件工程化管理
	
- 缺点
    - 没有明确的定义
    - 视图与控制器间的过于紧密的连接
    - 增加系统结构和实现的复杂性

![image](https://github.com/CharonChui/Pictures/blob/master/mvc_model.png?raw=true)

MVP
---

`MVP`是从经典的模式`MVC`演变而来，它们的基本思想有相通的地方：`Controller/Presenter`负责逻辑的处理，`Model`提供数据，`View`负责显示。
作为一种新的模式，`MVP`与`MVC`有着一个重大的区别：在`MVP`中`View`并不直接使用`Model`，它们之间的通信是通过`Presenter`(`MVC`中的`Controller`)来进行的，
所有的交互都发生在`Presenter`内部，而在`MVC`中`View`会直接从`Model`中读取数据而不是通过`Controller`。
在`MVC`里,`View`是可以直接访问`Model`的！从而，`View`里会包含`Model`信息，不可避免的还要包括一些业务逻辑。 
在`MVC`模型里，更关注的`Model`的不变，而同时有多个对`Model`的不同显示及`View`。所以，在`MVC`模型里，`Model`不依赖于`View`，但是`View`是依赖于`Model`的。
不仅如此，因为有一些业务逻辑在`View`里实现了，导致要更改`View`也是比较困难的，至少那些业务逻辑是无法重用的。	

![image](https://github.com/CharonChui/Pictures/blob/master/is-activity-god-the-mvp-architecture-10-638.jpg?raw=true)

在`MVP`里，`Presenter`完全把`Model`和`View`进行了分离，主要的程序逻辑在`Presenter`里实现。而且`Presenter`与具体的`View`是没有直接关联的，
而是通过定义好的接口进行交互，从而使得在变更`View`时候可以保持`Presenter`的不变，即重用！ 
在MVP里，应用程序的逻辑主要在`Presenter`来实现，其中的`View`是很薄的一层。因此就有人提出了`Presenter First`的设计模式，
就是根据`User Story`来首先设计和开发`Presenter`。在这个过程中，`View`是很简单的，能够把信息显示清楚就可以了。
在后面，根据需要再随便更改`View`，而对`Presenter`没有任何的影响了。 如果要实现的`UI`比较复杂，而且相关的显示逻辑还跟`Model`有关系，
就可以在`View`和`Presenter`之间放置一个`Adapter`。由这个`Adapter`来访问`Model`和`View`，避免两者之间的关联。
而同时，因为`Adapter`实现了`View`的接口，从而可以保证与`Presenter`之间接口的不变。这样就可以保证`View`和`Presenter`之间接口的简洁，
又不失去`UI`的灵活性。在`MVP`模式里，`View`只应该有简单的`Set/Get`的方法，用户输入和设置界面显示的内容，除此就不应该有更多的内容，
绝不容许直接访问`Model`这就是与`MVC`很大的不同之处。

- 优点    
    - 模型与视图完全分离，我们可以修改视图而不影响模型
    - 可以更高效地使用模型，因为所有的交互都发生在一个地方——`Presenter`内部
    - 我们可以将一个`Presenter`用于多个视图，而不需要改变`Presenter`的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁。
    - 如果我们把逻辑放在`Presenter`中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）

- 缺点
    由于对视图的渲染放在了`Presenter`中，所以视图和`Presenter`的交互会过于频繁。还有一点需要明白，如果`Presenter`过多地渲染了视图，
	往往会使得它与特定的视图的联系过于紧密。一旦视图需要变更，那么`Presenter`也需要变更了。
	比如说，原本用来呈现`Html`的`Presenter`现在也需要用于呈现Pdf了，那么视图很有可能也需要变更。

MVVM
---

MVVM是Model-View-ViewModel的简写。

- 优点
    `MVVM`模式和`MVC`模式一样，主要目的是分离视图`View`和模型`Model`
- 低耦合。
- 可重用性。
- 独立开发。
- 可测试。

---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	
