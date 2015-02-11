ListView源码分析
===


一直都想写一篇文章分析下`ListView`的实现，总是忙，一直拖到现在，快到年底了，写出来希望能帮助一些面试跳槽的人。      
当然写这篇文章也是有原因的，当时一个同事在面试的时候，被对方问答`ListView`实现原理，同事简单的答了一些，被对方刁难，    
最后不欢而散，想想也是，一下说起来，还真只能说些简单的原理。

在`Android`开发过程中`ListView`比较常用，先看一下文档中对`ListView`的说明：
```java
/*
 * Implementation Notes:
 *
 * Some terminology:
 *
 *     index    - index of the items that are currently visible
 *     position - index of the items in the cursor
 */


/**
 * A view that shows items in a vertically scrolling list. The items
 * come from the {@link ListAdapter} associated with this view.
 */
```

然后再说一下`ListView`的继承关系`ListView extends AbsListView extends AdapterView<ListAdapter> extends ViewGroup extends View`









---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 