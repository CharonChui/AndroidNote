Kotlin学习教程(二)
===

上一篇文章介绍了`Kotlin`的基本语法，我感觉在继续学习更多知识之前有必要单独介绍以下编码规范。    

不管学什么东西，开始形成的习惯以后想改都比较困难。所以开始就用规范的方式学习是最好的。 


### 命名风格

如果拿不准的时候，默认使用`Java`的编码规范，比如:   

- 使用驼峰法命名（并避免命名含有下划线）
- 类型名以大写字母开头
- 方法和属性以小写字母开头
- 使用4个空格缩进
- 公有函数应撰写函数文档，这样这些文档才会出现在`Kotlin Doc`中


### 冒号

类型和超类型之间的冒号前要有一个空格，而实例和类型之间的冒号前不要有空格:     

```kotlin
interface Foo<out T : Any> : Bar {
    fun foo(a: Int): T
}
```

### `Lambda`表达式

在`lambda`表达式中, 大括号左右要加空格，分隔参数与代码体的箭头左右也要加空格。`lambda`表达应尽可能不要写在圆括号中:    

```kotlin
list.filter { it > 10 }.map { element -> element * 2 }
```

### 类头格式化

有少数几个参数的类可以写成一行：

```kotlin
class Person(id: Int, name: String)
```

具有较长类头的类应该格式化，以使每个主构造函数参数位于带有缩进的单独一行中。 此外，右括号应该另起一行。如果我们使用继承，
那么超类构造函数调用或者实现接口列表应位于与括号相同的行上：

```kotlin
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name) {
    // ……
}
```
对于多个接口，应首先放置超类构造函数调用，然后每个接口应位于不同的行中：
```kotlin
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    // ……
}
```


### `Unit`  

如果函数返回`Unit`类型，该返回类型应该省略:   
```kotlin
fun foo() { // 省略了 ": Unit"

}
```



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

