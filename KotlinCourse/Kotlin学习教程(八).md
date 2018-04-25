Kotlin学习教程(八)
===

`Kotlin`协程
---

一些`API`启动长时间运行的操作(例如网络`IO`、文件`IO`、`CPU`或`GPU`密集型任务等)，并要求调用者阻塞直到它们完成。协程提供了一种避免阻塞线程
并用更廉价、更可控的操作替代线程阻塞的方法:协程挂起。     
协程通过将复杂性放入库来简化异步编程。程序的逻辑可以在协程中顺序地表达，而底层库会为我们解决其异步性。该库可以将用户代码的相关部分包装为回调、
订阅相关事件、在不同线程(甚至不同机器！)上调度执行，而代码则保持如同顺序执行一样简单。     


### 阻塞 vs 挂起

基本上，协程计算可以被挂起而无需阻塞线程。线程阻塞的代价通常是昂贵的，尤其在高负载时，因为只有相对少量线程实际可用，因此阻塞其中一个会导致一些
重要的任务被延迟。

另一方面，协程挂起几乎是无代价的。不需要上下文切换或者`OS`的任何其他干预。最重要的是，挂起可以在很大程度上由用户库控制:   
作为库的作者，我们可以决定挂起时发生什么并根据需求优化/记日志/截获。

另一个区别是，协程不能在随机的指令中挂起，而只能在所谓的挂起点挂起，这会调用特别标记的函数。

#### 挂起函数    

当我们调用标记有特殊修饰符`suspend`的函数时，会发生挂起:    

```kotlin
suspend fun doSomething(foo: Foo): Bar {
    ……
}
```

这样的函数称为挂起函数，因为调用它们可能挂起协程(如果相关调用的结果已经可用，库可以决定继续进行而不挂起)。挂起函数能够以与普通函数相同的方式
获取参数和返回值，但它们只能从协程和其他挂起函数中调用。事实上，要启动协程，
必须至少有一个挂起函数，它通常是匿名的(即它是一个挂起`lambda`表达式)。让我们来看一个例子，一个简化的`async()`函数
(源自`kotlinx.coroutines`库):    

```kotlin
fun <T> async(block: suspend () -> T)
```

这里的`async()`是一个普通函数(不是挂起函数)，但是它的`block`参数具有一个带`suspend`修饰符的函数类型:`suspend() -> T`。
所以，当我们将一个`lambda`表达式传给`async()`时，它会是挂起`lambda`表达式，于是我们可以从中调用挂起函数:    

```kotlin
async {
    doSomething(foo)
    ……
}
```

继续该类比,`await()`可以是一个挂起函数(因此也可以在一个`async {}`块中调用)，该函数挂起一个协程，直到一些计算完成并返回其结果:   
```kotlin
async {
    ……
    val result = computation.await()
    ……
}

```

更多关于`async/await`函数实际在`kotlinx.coroutines`中如何工作的信息可以在这里找到。

请注意，挂起函数`await()`和`doSomething()`不能在像`main()`这样的普通函数中调用:    
```kotlin
fun main(args: Array<String>) {
    doSomething() // 错误：挂起函数从非协程上下文调用
}
```

还要注意的是，挂起函数可以是虚拟的，当覆盖它们时，必须指定`suspend`修饰符:    
```kotlin
interface Base {
    suspend fun foo()
}

class Derived: Base {
    override suspend fun foo() { …… }
}
```

`Kotlin`解构声明    
---


有时把一个对象解构成很多变量会很方便:   

```kotlin
val (name, age) = person
```

这种语法称为解构声明。 一个解构声明可以同时创建多个变量。    

我们已经声明了两个新变量:`name`和`age`，并且可以独立使用它们:    
```kotlin
println(name)
println(age)
```

一个解构声明会被编译成以下代码:    
```kotlin
val name = person.component1()
val age = person.component2()
```

其中的`component1()`和`component2()`函数是在`Kotlin`中广泛使用的约定原则的另一个例子。

任何表达式都可以出现在解构声明的右侧，只要可以对它调用所需数量的`component`函数即可。
当然，可以有`component3()`和`component4()`等等。

请注意，`componentN()`函数需要用`operator`关键字标记，以允许在解构声明中使用它们。

解构声明也可以用在`for{: .keyword }`循环中:    
```kotlin
for ((a, b) in collection) { …… }
```
变量`a`和`b`的值取自对集合中的元素上调用`component1()`和`component2()`的返回值。

例:从函数中返回两个变量
让我们假设我们需要从一个函数返回两个东西。例如，一个结果对象和一个某种状态。
在`Kotlin`中一个简洁的实现方式是声明一个数据类并返回其实例:    
```kotlin
data class Result(val result: Int, val status: Status)
fun function(……): Result {
    // 各种计算
    return Result(result, status)
}
// 现在，使用该函数：
val (result, status) = function(……)
```
因为数据类自动声明`componentN()`函数，所以这里可以用解构声明。

注意：我们也可以使用标准类`Pair`并且让`function()`返回`Pair<Int, Status>`，
但是让数据合理命名通常更好。

例：解构声明和映射
可能遍历一个映射`(map)`最好的方式就是这样：
```kotlin
for ((key, value) in map) {
   // 使用该 key、value 做些事情
}
```
为使其能用，我们应该
通过提供一个`iterator()`函数将映射表示为一个值的序列，
通过提供函数`component1()`和`component2()`来将每个元素呈现为一对。
当然事实上，标准库提供了这样的扩展:   
```kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```
因此你可以在`for{: .keyword }`-循环中对映射(以及数据类实例的集合等)自由使用解构声明。


`Kotlin`反射
---


最基本的反射功能是获取`Kotlin`类的运行时引用。要获取对
静态已知的`Kotlin`类的引用，可以使用类字面值语法：
```kotlin
val c = KClass::class
```

该引用是`KClass`类型的值。

请注意，`Kotlin`类引用与`Java`类引用不同。要获得`Java`类引用，
请在`KClass`实例上使用`.java`属性，也就是`KClass::class.java`


#### 函数引用

当我们有一个命名函数声明如下:   
```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

我们可以很容易地直接调用它`(isOdd(5))`，但是我们也可以把它作为一个值传递。例如传给另一个函数。
为此，我们使用`::`操作符:   
```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 输出 [1, 3]
```
这里`::isOdd`是函数类型`(Int) -> Boolean`的一个值。

当上下文中已知函数期望的类型时`::`可以用于重载函数。

例如:      
```kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 引用到 isOdd(x: Int)
```

或者，你可以通过将方法引用存储在具有显式指定类型的变量中来提供必要的上下文:   
```kotlin
val predicate: (String) -> Boolean = ::isOdd   // 引用到 isOdd(x: String)
```
如果我们需要使用类的成员函数或扩展函数，它需要是限定的。
例如`String::toCharArray`为类型`String`提供了一个扩展函数:`String.() -> CharArray`。


#### 属性引用

要把属性作为`Kotlin`中的一等对象来访问，我们也可以使用`::`运算符:    

```kotlin
var x = 1

fun main(args: Array<String>) {
    println(::x.get()) // 输出 "1"
    ::x.set(2)
    println(x)         // 输出 "2"
}
```
表达式`::x`求值为`KProperty<Int>`类型的属性对象，它允许我们使用
`get()`读取它的值，或者使用`name`属性来获取属性名。更多信息请参见
关于`KProperty`类的文档。

对于可变属性，例如`var y = 1`，`::y`返回`KMutableProperty<Int>`类型的一个值，
该类型有一个`set()`方法。

属性引用可以用在不需要参数的函数处:   
```kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // 输出 [1, 2, 3]
```
要访问属于类的成员的属性，我们这样限定它:   
```kotlin
class A(val p: Int)

fun main(args: Array<String>) {
    val prop = A::p
    println(prop.get(A(1))) // 输出 "1"
}
```


Kotlin类型别名
---

类型别名为现有类型提供替代名称。
如果类型名称太长，你可以另外引入较短的名称，并使用新的名称替代原类型名。

它有助于缩短较长的泛型类型。
例如，通常缩减集合类型是很有吸引力的:    
```kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

你可以为函数类型提供另外的别名:   
```kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```
你可以为内部类和嵌套类创建新名称：

```kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```


类型别名不会引入新类型。
它们等效于相应的底层类型。
当你在代码中添加`typealias Predicate<T>`并使用`Predicate<Int>`时，`Kotlin`编译器总是把它扩展为`(Int) -> Boolean`。
因此，当你需要泛型函数类型时，你可以传递该类型的变量，反之亦然:    

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // 输出 "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // 输出 "[1]"
}
```



文档
---


用来编写`Kotlin`代码文档的语言(相当于`Java`的`JavaDoc`)称为`KDoc`。本质上`KDoc`是将`JavaDoc`的块标签`(block tags)`语法(
扩展为支持`Kotlin`的特定构造)和`Markdown`的内联标记`(inline markup)`结合在一起。


#### 生成文档    

`Kotlin`的文档生成工具称为[Dokka](https://github.com/Kotlin/dokka)。

`Dokka`有`Gradle`、`Maven`和`Ant`的插件，因此你可以将文档生成集成到你的构建过程中。


像`JavaDoc`一样，`KDoc`注释也以`/**`开头、以`*/`结尾。注释的每一行可以以
星号开头，该星号不会当作注释内容的一部分。

按惯例来说，文档文本的第一段(到第一行空白行结束)是该元素的
总体描述，接下来的注释是详细描述。

每个块标签都以一个新行开始且以`@`字符开头。

以下是使用`KDoc`编写类文档的一个示例:    
```kotlin
/**
 * 一组*成员*。
 *
 * 这个类没有有用的逻辑; 它只是一个文档示例。
 *
 * @param T 这个组中的成员的类型。
 * @property name 这个组的名称。
 * @constructor 创建一个空组。
 */
class Group<T>(val name: String) {
    /**
     * 将 [member] 添加到这个组。
     * @return 这个组的新大小。
     */
    fun add(member: T): Int { …… }
}
```

`KDoc`目前支持以下块标签`(block tags)`:     

- `@param` <名称>

    用于函数的值参数或者类、属性或函数的类型参数。
    为了更好地将参数名称与描述分开，如果你愿意，可以将参数的名称括在
    方括号中。因此，以下两种语法是等效的:   
	```kotlin
	@param name 描述。
	@param[name] 描述。
	```

- `@return`

    用于函数的返回值。

- `@constructor`

    用于类的主构造函数。

- `@receiver`

    用于扩展函数的接收者。

- `@property` <名称>

    用于类中具有指定名称的属性。这个标签可用于在
    主构造函数中声明的属性，当然直接在属性定义的前面放置`doc`注释会很别扭。

- `@throws` <类>,`@exception` <类>     

    用于方法可能抛出的异常。因为`Kotlin`没有受检异常，所以也没有期望所有可能的异常都写文档，但是当它会为类的用户提供有用的信息时，仍然可以使用这个标签。

- `@sample` <标识符>    

    将具有指定限定的名称的函数的主体嵌入到当前元素的文档中，以显示如何使用该元素的示例。

- `@see` <标识符>     

    将到指定类或方法的链接添加到文档的另请参见块。

- @author

    指定要编写文档的元素的作者。

- `@since`    

    指定要编写文档的元素引入时的软件版本。

- `@suppress`     

    从生成的文档中排除元素。可用于不是模块的官方`API`的一部分但还是必须在对外可见的元素。

`KDoc`不支持`@deprecated`这个标签。作为替代，请使用`@Deprecated`注解。


#### 内联标记   

对于内联标记，`KDoc`使用常规`Markdown`语法，扩展了支持用于链接到代码中其他元素的简写语法。

链接到元素
要链接到另一个元素(类、方法、属性或参数)，只需将其名称放在方括号中：
```
为此目的，请使用方法 [foo]。
```
如果要为链接指定自定义标签(label)，请使用 Markdown 引用样式语法：
```
为此目的，请使用[这个方法][foo]。
```
你还可以在链接中使用限定的名称。请注意，与 JavaDoc 不同，限定的名称总是使用点字符
来分隔组件，即使在方法名称之前：
```
使用 [kotlin.reflect.KClass.properties] 来枚举类的属性。
```
链接中的名称与正写文档的元素内使用该名称使用相同的规则解析。
特别是，这意味着如果你已将名称导入当前文件，那么当你在`KDoc`注释中使用它时，
不需要再对其进行完整限定。

请注意`KDoc`没有用于解析链接中的重载成员的任何语法。因为`Kotlin`文档生成
工具将一个函数的所有重载的文档放在同一页面上，标识一个特定的重载函数
并不是链接生效所必需的。



常用操作符及函数
---

#### `let`操作符  

如果对象的值不为空，则允许执行这个方法。返回值是函数里面最后一行，或者指定`return`
```kotlin
private var test: String? = null

private fun switchFragment(position: Int) {
    test?.let {
        LogUtil.e("@@@", "test is not null")
    }
}    
```

说到可能有人会觉得没什么用，用`if`判断下是不是空不就完了.
```kotlin
private var test: String? = null

private fun switchFragment(position: Int) {
//        test?.let {
//            LogUtil.e("@@@", "test is null")
//        }

    if (test == null) {
        LogUtil.e("@@@", "test is null")
    } else {
        LogUtil.e("@@@", "test is not null ${test}")
        check(test) // 报错
    }
}    
```
但是会报错:`Smart cast to 'String' is impossible, beacuase 'test' is a mutable property that could have been changed by this time`

#### `sNullOrEmpty | isNullOrBlank`

```kotlin
public inline fun CharSequence?.isNullOrEmpty(): Boolean = this == null || this.length == 0

public inline fun CharSequence?.isNullOrBlank(): Boolean = this == null || this.isBlank()

// If we do not care about the possibility of only spaces...
if (number.isNullOrEmpty()) {
    // alert the user to fill in their number!
}

// when we need to block the user from inputting only spaces
if (name.isNullOrBlank()) {
    // alert the user to fill in their name!
}
```

#### `with`函数

`with`是一个非常有用的函数，它包含在`Kotlin`的标准库中。它接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。
这表示所有我们在括号中编写的代码都是作为对象（第一个参数）的一个扩展函数，我们可以就像作为`this`一样使用所有它的`public`方法和属性。
当我们针对同一个对象做很多操作的时候这个非常有利于简化代码。

```kotlin
fun testWith() {
    with(ArrayList<String>()) {
        add("testWith")
        add("testWith")
        add("testWith")
        println("this = " + this)
    }
}
// 运行结果
// this = [testWith, testWith, testWith]
```

#### `repeat`函数

`repeat`函数是一个单独的函数，定义如下:     
```kotlin
/**
 * Executes the given function [action] specified number of [times].
 *
 * A zero-based index of current iteration is passed as a parameter to [action].
 */
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }

    for (index in 0..times - 1) {
        action(index)
    }
}
```
通过代码很容易理解，就是循环执行多少次`block`中内容。
```kotlin
fun main(args: Array<String>) {
    repeat(3) {
        println("Hello world")
    }
}
```
运行结果是:    
```kotlin
Hello world
Hello world
Hello world
```

#### `apply`函数

`apply`函数是这样的，调用某对象的`apply`函数，在函数范围内，可以任意调用该对象的任意方法，并返回该对象
```kotlin
fun testApply() {
    ArrayList<String>().apply {
        add("testApply")
        add("testApply")
        add("testApply")
        println("this = " + this)
    }.let { println(it) }
}

// 运行结果
// this = [testApply, testApply, testApply]
// [testApply, testApply, testApply]
```

`run`函数和`apply`函数很像，只不过run函数是使用最后一行的返回，apply返回当前自己的对象。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
