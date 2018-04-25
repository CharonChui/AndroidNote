Kotlin学习教程(四)
===


### 数据类:使用`data class`定义

数据类是一种非常强大的类。在[Kotlin学习教程(一)][1]中最开始的用的简洁的示例代码就是一个数据类。这里我们再拿过来:   
```java
public class Artist {
    private long id;
    private String name;
    private String url;
    private String mbid;

    public long getId() {
        return id;
    }


    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getMbid() {
        return mbid;
    }

    public void setMbid(String mbid) {
        this.mbid = mbid;
    }

    @Override public String toString() {
        return "Artist{" +
          "id=" + id +
          ", name='" + name + '\'' +
          ", url='" + url + '\'' +
          ", mbid='" + mbid + '\'' +
          '}';
    }
}
```
使用`Kotlin`:  
```kotlin
data class Artist(
    var id: Long,
    var name: String,
    var url: String,
    var mbid: String)
```

通过数据类，会自动提供以下函数：
- 所有属性的`get() set()`方法
- `equals()`
- `hashCode()`
- `copy()`
- `toString()`
- 一系列可以映射对象到变量中的函数(后面再说)。

如果我们使用不可修改的对象，就像我们之前讲过的，假如我们需要修改这个对象状态，必须要创建一个新的一个或者多个属性被修改的实例。
这个任务是非常重复且不简洁的。

举个例子,如果要修改`Person`类中`charon`的`age`: 

```kotlin
data class Person(val name: String,
                  val age: Int)
```

```kotlin
val charon = Person("charon", 18)
val charon2 = charon.copy(age = 19)
```
如上，我们拷贝了`charon`对象然后只修改了`age`的属性而没有修改这个对象的其它状态。

### 多声明

多声明，也可以理解为变量映射，这就是编译器自动生成的`componentN()`方法。

```kotlin
var personD = PersonData("PersonData", 20, "male")
var (name, age) = personD


Log.d("test", "name = $name, age = $age")

//输出
name = PersonData, age = 20
```

上面的多声明，大概可以翻译成这样：

```kotlin
var name = f1.component1()
var age = f1.component2()
```


### 继承

在`Kotlin`中所有类都有一个共同的超类`Any`，这对于没有超类型声明的类是默认超类:   
```kotlin
class Person // 从 Any 隐式继承
```

`Any`不是`java.lang.Object`。它除了`equals()`、`hashCode()`和`toString()`外没有任何成员。
`Kotlin`中所有的类默认都是不可继承的(`final`)，为什么要这样设计呢？引用`Effective Java`书中的第17条:要么为继承而设计，并提供文档说明，
要么就禁止继承。所以我们只能继承那些明确声明`open`或者`abstract`的类:要声明一个显式的超类型，我们把类型放到类头的冒号之后:   
```kotlin
open class Person(num: Int)
// 继承
class SuperPerson(num: Int) : Person(num)
```
如果该类有一个主构造函数，其基类必须用基类型的主构造函数参数就地初始化。
如果类没有主构造函数，那么每个次构造函数必须使用`super`关键字初始化其基类型，或委托给另一个构造函数做到这一点。
注意，在这种情况下，不同的次构造函数可以调用基类型的不同的构造函数:   
```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 覆盖

##### 方法覆盖


只能重写显示标注可覆盖的方法:   
```kotlin
open class Person(num: Int) {
    open fun changeName(name: String) {

    }

    fun changeAge(age: Int) {

    }
}

class SuperPerson(num: Int) : Person(num) {
    override fun changeName(name: String) {
        // 通过super关键字调用超类实现
        super.changeName(name)
    }
}
```
`SuperPerson.changeName()`方法前面必须加上`override`标注，不然编译器将会报错。如果像上面`Person.changeAge()`方法没有标注`open`,
则子类中不能定义相同的方法:   
```kotlin
class SuperPerson(num: Int) : Person(num) {
    override fun changeName(name: String) {
        super.changeName(name)
    }

    // 编译器报错
    fun changeAge(age: Int) {

    }
    // 重载是可以的
    fun changeAge(name: String) {

    }
    // 重载是可以的
    fun changeAge(age: Int, name: String) {

    }
}
```

标记为`override`的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，可以使用`final`关键字:    

```kotlin
open class SuperPerson(num: Int) : Person(num) {
    final override fun changeName(name: String) {
        super.changeName(name)
    }
}
```

##### 属性覆盖

属性覆盖与方法覆盖类似，只能覆盖显示标明`open`的属性，并且要用`override`开头:  

```kotlin
open class Person(num: Int) {
    open val name: String = ""

    open fun changeName(name: String) {

    }

    fun changeAge(age: Int) {

    }
}

open class SuperPerson(num: Int) : Person(num) {
    override val name: String
        get() = super.name

    final override fun changeName(name: String) {
        super.changeName(name)
    }

}
```

每个声明的属性可以由具有初始化器的属性或者具有`get`方法的属性覆盖，你也可以用一个`var`属性覆盖一个`val`属性，但反之则不行。



### 抽象类

类和其中的某些成员可以声明为`abstract`。抽象成员在本类中可以不用实现。 需要注意的是，我们并不需要用`open`标注一个抽象类或者函数——因为这不
言而喻。

我们可以用一个抽象成员覆盖一个非抽象的开放成员:  
```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

### 修饰符

`Kotlin`中修饰符是与`Java`中的有些不同。在`kotlin`中默认的修饰符是`public`，这节约了很多的时间和字符。

- `private`        
    `private`修饰符是最限制的修饰符，和`Java`中`private`一样。它表示它只能被自己所在的文件可见。所以如果我们给一个类声明为`private`，
    我们就不能在定义这个类之外的文件中使用它。
    另一方面，如果我们在一个类里面使用了private修饰符，那访问权限就被限制在这个类里面了。甚至是继承这个类的子类也不能使用它。

- `protected`.    
    与`Java`一样，它可以被成员自己和继承它的成员可见。

- `internal`
    如果是一个定义为`internal`的包成员的话，对所在的整个`module`可见。如果它是一个其它领域的成员，它就需要依赖那个领域的可见性了。
    比如如果写了一个`private`类，那么它的`internal`修饰的函数的可见性就会限制与它所在的这个类的可见性。

- `public`.   
    你应该可以才想到，这是最没有限制的修饰符。这是默认的修饰符，成员在任何地方被修饰为public，很明显它只限制于它的领域。


### 数组  

数组用类`Array`实现，并且还有一个`size`属性及`get`和`set`方法，由于使用`[]`重载了`get`和`set`方法，所以我们可以通过下标很方便的获取或者
设置数组对应位置的值。       
`Kotlin`标准库提供了`arrayOf()`创建数组和`xxArrayOf`创建特定类型数组      
```kotlin
val array = arrayOf(1, 2, 3)
val countries = arrayOf("UK", "Germany", "Italy")
val numbers = intArrayOf(10, 20, 30)
val array1 = Array(10, { k -> k * k })
val longArray = emptyArray<Long>()
val studentArray = Array<Student>(2)
studentArray[0] = Student("james")
```

和`Java`不一样的是`Kotlin`的数组是容器类，提供了`ByteArray`,`CharArray`,`ShortArray`,`IntArray`,`LongArray`,`BooleanArray`,
`FloatArray`和`DoubleArray`。

### 集合 


`Kotlin`的`List<out T>`类型是一个提供只读操作如`size`、`get`等的接口。和`Java`类似，它继承自`Collection<T>`进而继承自`Iterable<T>`。
改变`list`的方法是由`MutableList<T>`加入的。这一模式同样适用于`Set<out T>/MutableSet<T>`及`Map<K, out V>/MutableMap<K, V>`。

`Kotlin`没有专门的语法结构创建`list`或`set`。要用标准库的方法如`listOf()`、`mutableListOf()`、`setOf()`、`mutableSetOf()`。
创建`map`可以用`mapOf(a to b, c to d)`。

```kotlin
fun main(args : Array<String>) {
    var lists = listOf("a", "b", "c")
    for(list in lists) {
        println(list)
    }
}
```

```kotlin
fun main(args : Array<String>) {
    var map = TreeMap<String, String>()
    map["0"] = "0 haha"
    map["1"] = "1 haha"
    map["2"] = "2 haha"
    
    println(map["1"])
}
```

```kotlin    
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // 输出 "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // 输出 "[1, 2, 3, 4]"
readOnlyView.clear()    // -> 不能编译

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```


### 可`null`类型  


因为在`Kotlin`中一切都是对象，一切都是可`null`的。当某个变量的值可以为`null`的时候，必须在声明处的类型后添加`?`来标识该引用可为空。
`Kotlin`通过`?`将是否允许为空分割开来，比如`str:String`为不能空，加上`?`后的`str:String?`为允许空，通过这种方式，将本是不能确定的变
量人为的加入了限制条件。而不符合条件的输入，则会在`IDE`上显示编译错误而无法执行。

```kotlin
var value1: String
value1 = null        // 编译错误 Null can not be a value of a non-null type String

var value2 : String? 
value2 = null       // 编译通过
```
在对变量进行操作时，如果变量是可能为空的，那么将不能直接调用，因为编译器不知道你的变量是否为空，所以编译器就要求你一定要对变量进行判断
```kotlin
var str : String? = null
// 编译错误 Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
str.length    
// 编译能通过，这表示如果str不为空的时候执行length方法
str?.length   
```

那么问题来了，我们知道在`java`中`String.length`返回的是`int`，上面的`str?.length`既然编译通过了，那么它返回了什么？我们可以这么写:  

`var result = str?.length`

这么写编译器是能通过的，那么`result`的类型是什么呢？在`Kotlin`中，编译器会自动根据结果判断变量的类型，翻译成普通代码如下:   

```kotlin
if(str == null)
    result = null;            // 这里result为一个引用类型
else
    result = str.length;    // 这里result为Int
```
那么如果我们需要的就是一个`Int`的结果(事实上大部分情况都是如此)，那又该怎么办呢？在`kotlin`中除了`?`表示可为空以外，还有一个新的符号`:`双
感叹号`!!`，表示一定不能为空。所以上面的例子，如果要对`result`进行操作，可以这么写:  
```kotlin
var str : String? = null
var result : Int = str!!.length
```

这样的话，就能保证`result`的数据类型，但是这样还有一个问题，那就是`str`的定义是可为空的，上面的代码中，`str`就是空，这时候下面的操作虽然
不会报编译异常，但是运行时就会见到我们熟悉的空指针异常`NullPointerExectpion`，这显然不是我们希望见到的，也不是`kotlin`愿意见到的。
`java`中的三元操作符大家应该都很熟悉了，`kotlin`中也有类似的，它很好的解决了刚刚说到的问题。在`kotlin`中，三元操作符是`?:`，写起来也
比`java`要方便一些。

```kotlin
var str : String? = null
var result = str?.length ?: -1
//等价于
var result : Int = if(str != null) str.length else -1
```

`if null`缩写

```kotlin
val data = ……
val email = data["email"] ?: throw IllegalStateException("Email is missing!")
```

如果`?:`左侧表达式非空，`elvis`操作符就返回其左侧表达式，否则返回右侧表达式。
请注意，当且仅当左侧为空时，才会对右侧表达式求值。


##### `!!`操作符

我们可以写`b!!`，这会返回一个非空的`b`值
(例如:在我们例子中的`String`)或者如果`b`为空，就会抛出一个空指针异常:     
```kotlin
val l = b!!.length
```

因此，如果你想要一个 NPE，你可以得到它，但是你必须显式要求它，否则它不会不期而至。


#### 安全的类型转换

如果对象不是目标类型，那么常规类型转换可能会导致`ClassCastException`。
另一个选择是使用安全的类型转换，如果尝试转换不成功则返回`null{: .keyword }`:    

```kotlin
val aInt: Int? = a as? Int
```

#### 可空类型的集合

如果你有一个可空类型元素的集合，并且想要过滤非空元素，你可以使用`filterNotNull`来实现。
```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```

### 表达式    

##### `if`表达式    

在`Kotlin`中，`if`是一个表达式，即它会返回一个值。因此就不需要三元运算符`条件 ? 然后 : 否则`，因为普通的`if`就能胜任这个角色。
`if`的分支可以是代码块，最后的表达式作为该块的值:   
```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```


##### `when`表达式    

`when`表达式与`Java`中的`switch/case`类似，但是要强大得多。这个表达式会去试图匹配所有可能的分支直到找到满意的一项。然后它会运行右边的表达
式。      
与`Java`的`switch/case`不同之处是参数可以是任何类型，并且分支也可以是一个条件。

对于默认的选项，我们可以增加一个`else`分支，它会在前面没有任何条件匹配时再执行。条件匹配成功后执行的代码也可以是代码块:     
```kotlin
when (x){
    1 -> print("x == 1") 
    2 -> print("x == 2") 
    else -> {
        print("I'm a block")
        print("x is neither 1 nor 2")
    }
}
```

因为它是一个表达式，它也可以返回一个值。我们需要考虑什么时候作为一个表达式使用，它必须要覆盖所有分支的可能性或者实现`else`分支。否则它不会被
编译成功:   

```kotlin
val result = when (x) {
    0, 1 -> "binary"
    else -> "error"
}
```

如你所见，条件可以是一系列被逗号分割的值。但是它可以更多的匹配方式。比如，我们可以检测参数类型并进行判断:  

```kotlin
when(view) {
    is TextView -> view.setText("I'm a TextView")
    is EditText -> toast("EditText value: ${view.getText()}")
    is ViewGroup -> toast("Number of children: ${view.getChildCount()} ")
    else -> view.visibility = View.GONE
}
```

##### for循环

```kotlin
val items = listOf("apple", "banana", "kiwi")
for (item in items) {
    println(item)
}

for (i in array.indices)
    print(array[i])
```

### 使用类型检测及自动类型转换

`is`运算符检测一个表达式是否某类型的一个实例。 如果一个不可变的局部变量或属性已经判断出为某类型，那么检测后的分支中可以直接当作该类型使用，
无需显式转换:   

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` 在这一分支自动转换为 `String`
    return obj.length
}
```

### 返回和跳转

`Kotlin`有三种结构化跳转表达式:    

- `return`:默认从最直接包围它的函数或者匿名函数返回。
- `break`:终止最直接包围它的循环。
- `continue`:继续下一次最直接包围它的循环。

在`Kotlin`中任何表达式都可以用标签`label`来标记。标签的格式为标识符后跟`@`符号，例如:`abc@`、`fooBar@`都是有效的标签。

要为一个表达式加标签，我们只要在其前加标签即可。
```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}
```


### Ranges

`Range`表达式使用一个`..`操作符。表示就是一个该范围内的数据的数组，包含头和尾

```kotlin
var nums = 1..100 
for(num in nums) {
	println(num)
	// 打印出1 2 3 ....100
}
```

```kotlin
if(i >= 0 && i <= 10) 
    println(i)
```
转换成 

```kotlin
if (i in 0..10) 
    println(i)
```
Ranges默认会自增长，所以如果像以下的代码：
```kotlin
for (i in 10..0)
    println(i)
```
它就不会做任何事情。但是你可以使用`downTo`函数：
```kotlin
for(i in 10 downTo 0)
    println(i)
```

我们可以在`Ranges`中使用`step`来定义一个从`1`到一个值的不同的空隙:   
```kotlin
for (i in 1..4 step 2) println(i)
for (i in 4 downTo 1 step 2) println(i)
```

### Until

上面的`Range`是包含了头和尾，那如果只想包含头不包含尾呢？ 就要用`until`

```kotlin
var nums = 1 until 100
for(num in nums) {
	println(num)
	// 这样打印出来是1 2 3 .....99
}
```


[1]: https://github.com/CharonChui/AndroidNote/blob/master/KotlinCourse/Kotlin%E5%AD%A6%E4%B9%A0%E6%95%99%E7%A8%8B(%E4%B8%80).md "Kotlin学习教程(一)"

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

