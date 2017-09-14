hashCode与equals
===

`HashSet`和`HashMap`一直都是`JDK`中最常用的两个类，`HashSet`要求不能存储相同的对象，`HashMap`要求不能存储相同的键。 那么`Java`运行时环境是如何判断`HashSet`
中相同对象、`HashMap`中相同键的呢？当存储了相同的东西之后`Java`运行时环境又将如何来维护呢？             
在研究这个问题之前，首先说明一下`JDK`对`equals(Object obj)`和`hashcode()`这两个方法的定义和规范:在`Java`中任何一个对象都具备`equals(Object obj)`
和`hashcode()`这两个方法，因为他们是在`Object`类中定义的。`equals(Object obj)`方法用来判断两个对象是否“相同”，如果“相同”则返回`true`，否则返回`false`。 
`hashcode()`方法返回一个`int`数，在`Object`类中的默认实现是“将该对象的内部地址转换成一个整数返回”。           

接下来有两个个关于这两个方法的重要规范:    
- 若重写`equals(Object obj)`方法，有必要重写`hashcode()`方法，确保通过`equals(Object obj)`方法判断结果为`true`的两个对象具备相等的`hashcode()`返回值。
    说得简单点就是:  如果两个对象相同，那么他们的hashcode应该相等。不过请注意：这个只是规范，如果你非要写一个类让`equals(Object obj)`返回`true`
	而`hashcode()`返回两个不相等的值，编译和运行都是不会报错的。不过这样违反了`Java`规范，程序也就埋下了`BUG`。 
- `如果equals(Object obj)`返回`false`，即两个对象“不相同”，并不要求对这两个对象调用`hashcode()`方法得到两个不相同的数。
    说的简单点就是：“如果两个对象不相同，他们的`hashcode`可能相同”。 
- 如果两个对象相同，那么它们的`hashCode`值一定要相同；
- 如果两个对象的`hashCode`相同，它们并不一定相同
上面说的对象相同指的是用`eqauls`方法比较。    
你当然可以不按要求去做了，但你会发现，相同的对象可以出现在`Set`集合中。同时，增加新元素的效率会大大下降。

---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	
