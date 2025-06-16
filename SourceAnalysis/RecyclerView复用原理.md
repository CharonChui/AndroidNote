RecyclerView复用原理
===

但是很多时候我们的使用方式并不是最完美的，将Adapter和数据进行了强绑定。尽管在很多实际项目中，将数据存储在 Adapter 内部，甚至将数据作为了Adapter的成员变量是一种常见的做法:

这种实现方式就是把数据给Adapter，然后Adapter进过处理，完成数据的展示，不过，从严格的单一职责原则角度出发， Adapter 只应该负责“映射”而不是“管理”数据。也就是说，数据的维护（如数据的获取、更新和业务逻辑）可以交给 Activity、Fragment 或 ViewModel，而 Adapter 仅仅负责根据外部传入的数据来创建和绑定视图





---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
