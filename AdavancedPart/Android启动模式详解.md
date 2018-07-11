Android启动模式详解
===

- `standard`    
    默认模式。在该模式下，`Activity`可以拥有多个实例，并且这些实例既可以位于同一个`task`，也可以位于不同的`task`。每次都会新创建。
- `singleTop`        
    该模式下，在同一个`task`中，如果存在该`Activity`的实例，并且该`Activity`实例位于栈顶则不会创建该`Activity`的示例,而仅仅只是调用`Activity`的`onNewIntent()`。否则的话，则新建该`Activity`的实例，并将其置于栈顶。
- `singleTask`     
    顾名思义，只容许有一个包含该`Activity`实例的`task`存在！
    在`android`浏览器`browser`中，`BrowserActivity`的`launcherMode="singleTask"`，因为`browser`不断地启动自己，所以要求这个栈中保持只能有一个自己的实例，`browser`上网的时候，
	遇到播放视频的链接，就会通过隐式`intent`方式跳转找`Gallery3D`中的`MovieView`这个类来播放视频，这时候如果你点击`home`键，再点击`browser`，你会发现`MovieView`这个类已经销毁不存在了，
	而不会像保存这个`MovieView`的类对象，给客户带来的用户体验特别的不好。就像别人总结的`singleTask`模式的`Activity`不管是位于栈顶还是栈底，再次运行这个`Activity`时，都会`destory`掉它上面的`Activity`来保证整个栈中只有一个自己。                   
    下面是官方文档中的介绍:      
    `The system creates a new task and instantiates the activity at the root of the new task. However, if an instance of the activity already exists in a separate task, the system routes the intent to the existing 
	instance through a call to its onNewIntent() method, rather than creating a new instance. Only one instance of the activity can exist at a time.`
    以`singleTask`方式启动的`Activity`，全局只有唯一个实例存在，因此，当我们第一次启动这个`Activity`时，系统便会创建一个新的任务栈，并且初始化一个`Activity`实例，放在新任务栈的底部，如果下次再启动这个`Activity`时，
	系统发现已经存在这样的`Activity`实例，就会调用这个`Activity`实例的`onNewIntent`方法，从而把它激活起来。从这句话就可以推断出，以`singleTask`方式启动的`Activity`总是属于一个任务栈的根`Activity`。
    下面我们看一下示例图:　
    ![image](https://github.com/CharonChui/Pictures/blob/master/singletask.gif?raw=true)          
     坑爹啊！有木有！前面刚说`singleTask`会在新的任务中运行，并且位于任务堆栈的底部，这里在`Task B`中，一个赤裸裸的带着`singleTask`标签的箭头无情地指向`Task B`堆栈顶端的`Activity Y`，什么鬼？               
这其实是和`taskAffinity`有关，在将要启动时，系统会根据要启动的`Activity`的`taskAffinity`属性值在系统中查找这样的一个`Task`：`Task`的`affinity`属性值与即将要启动的`Activity`的`taskAffinity`属性值一致。如果存在，
就返回这个`Task`堆栈顶端的`Activity`回去，不重新创建任务栈了，再去启动另外一个`singletask`的`activity`时就会在跟它有相同`taskAffinity`的任务中启动，并且位于这个任务的堆栈顶端，于是，前面那个图中，
就会出现一个带着`singleTask`标签的箭头指向一个任务堆栈顶端的`Activity Y`了。在上面的`AndroidManifest.xml`文件中，没有配置`MainActivity`和`SubActivity`的`taskAffinity`属性，
于是它们的`taskAffinity`属性值就默认为父标签`application`的`taskAffinity`属性值，这里，标签`application`的`taskAffinity`也没有配置，于是它们就默认为包名。                               
总的来说：`singleTask`的结论与`android:taskAffinity`相关:    　　           
    - 设置了`singleTask`启动模式的`Activity`，它在启动的时候，会先在系统中查找属性值`affinity`等于它的属性值`taskAffinity`的任务栈的存在；如果存在这样的任务栈，它就会在这个任务栈中启动，否则就会在新任务栈中启动。
	因此，如果我们想要设置了`singleTask`启动模式的`Activity`在新的任务栈中启动，就要为它设置一个独立的`taskAffinity`属性值。以`A`启动`B`来说当`A`和`B`的`taskAffinity`相同时：第一次创建`B`的实例时，并不会启动新的`task`，
	而是直接将`B`添加到`A`所在的`task`；否则，将`B`所在`task`中位于`B`之上的全部`Activity`都删除，然后跳转到`B`中。
    - 如果设置了`singleTask`启动模式的`Activity`不是在新的任务中启动时，它会在已有的任务中查看是否已经存在相应的`Activity`实例，如果存在，就会把位于这个`Activity`实例上面的`Activity`全部结束掉，
	即最终这个Activity实例会位于任务的堆栈顶端中。以`A`启动`B`来说,当`A`和`B`的`taskAffinity`不同时：第一次创建`B`的实例时，会启动新的`task`，然后将`B`添加到新建的`task`中；否则，将`B`所在`task`中位于`B`之上的全部`Activity`都删除，然后跳转到`B`中。
- `singleInstance`
顾名思义，是单一实例的意思，即任意时刻只允许存在唯一的`Activity`实例，而且该`Activity`所在的`task`不能容纳除该`Activity`之外的其他`Activity`实例！               
它与`singleTask`有相同之处，也有不同之处。          
相同之处：任意时刻，最多只允许存在一个实例。            
不同之处：
    - `singleTask`受`android:taskAffinity`属性的影响，而`singleInstance`不受`android:taskAffinity`的影响。 
    - `singleTask`所在的`task`中能有其它的`Activity`，而`singleInstance`的`task`中不能有其他`Activity`。     
    - 当跳转到`singleTask`类型的`Activity`，并且该`Activity`实例已经存在时，会删除该`Activity`所在`task`中位于该`Activity`之上的全部`Activity`实例；而跳转到`singleInstance`类型的`Activity`，并且该`Activity`已经存在时，
	不需要删除其他`Activity`，因为它所在的`task`只有该`Activity`唯一一个`Activity`实例。

    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 