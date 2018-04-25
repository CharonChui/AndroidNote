Kotlin学习教程(一)
===

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/android_kotlin.jpeg?raw=true)

在`5月18`日谷歌在`I/O`开发者大会上宣布，将`Kotlin`语言作为安卓开发的一级编程语言。并且会在`Android Studio 3.0`版本全面支持`Kotlin`。   

- `Kotlin`是一个基于`JVM`的新的编程语言，由[JetBrains](https://www.jetbrains.com/)开发。`JetBrains`作为目前广受欢迎的
`Java IDE IntelliJ`的提供商，在`Apache`许可下已经开源其`Kotlin`编程语言。     
- `Kotlin`可以编译成`Java`字节码，也可以编译成`JavaScript`，方便在没有`JVM`的设备上运行。      
- `Kotlin`已正式成为`Android`官方开发语言。

[Kotlin官网](https://kotlinlang.org/)

`JetBrains`这家公司非常牛逼，开发了很多著名的软件，他们在使用`Java`的过程中发现`java`比较笨重不方便，所以就开发了`kotlin`，`kotlin`是
一种全栈的开发语言，可以用它进行开发`web`、`web`后端、`Android`等。    

很多开发者都说`Google`学什么不好，非要学苹果，出个`android`的`swift`版本，一定会搞不起来没人用，所以不用浪费时间去学习。在这里想引用马云
的一句话: 
> 拥抱变化

`Google`做事，向来言出必行，之前在推行`Android Studio`时也是一片骂声，吐槽各种不好用，各种慢。但是现在`Android Studio`基本都已经普及了。
我相信`Kotlin`也不会例外。所以我们不仅要学，还要要认真的学。    


### `Kotlin`的特性

- 它更加易表现：这是它最重要的优点之一。你可以编写少得多的代码。
- `Kotlin`是一种兼容`Java`的语言
- `Kotlin`比`Java`更安全，能够静态检测常见的陷阱。如:引用空指针
- `Kotlin`比`Java`更简洁，通过支持`variable type inference，higher-order functions (closures)，extension functions，mixins 
and first-class delegation`等实现
- `Kotlin`可与`Java`语言无缝通信。这意味着我们可以在`Kotlin`代码中使用任何已有的`Java`库；同样的`Kotlin`代码还可以为`Java`代码所用
- `Kotlin`在代码中很少需要在代码中指定类型，因为编译器可以在绝大多数情况下推断出变量或是函数返回值的类型。这样就能获得两个好处:简洁与安全


### `Kotlin`优势 

- 全面支持`Lambda`表达式
- 数据类`Data classes`
- 函数字面量和内联函数`Function literals & inline functions`
- 函数扩展`Extension functions`
- 空安全`Null safety`
- 智能转换`Smart casts`
- 字符串模板`String templates`
- 主构造函数`Primary constructors`
- 类委托`Class delegation`
- 类型推判`Type inference`
- 单例`Singletons`
- 声明点变量`Declaration-site variance`
- 区间表达式`Range expressions`


上面说简洁简洁，到底简洁在哪里？这里先用一个例子开始，在`Java`开发过程中经常会写一些`Bean`类:  
```java
package com.charon.kotlinstudydemo;

public class Person {
    private int age;
    private String name;
    private float height;
    private float weight;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getHeight() {
        return height;
    }

    public void setHeight(float height) {
        this.height = height;
    }

    public float getWeight() {
        return weight;
    }

    public void setWeight(float weight) {
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "Person name is : " + name + " age is : " + age + " height is :"
                + height + " weight is :" + weight;
    }
}
```
使用`Kotlin`:  
```kotlin
package com.charon.kotlinstudydemo

data class Person(
        var name: String,
        var age: Int,
        var height: Float,
        var weight: Float)
```
这个数据类，它会自动生成所有属性和它们的访问器，以及一些有用的方法，比如`toString()`方法。   
这里插一嘴，从上面的例子中我们可以看到对于包的声明基本是一样的，唯一不同的是`kotlin`中后面结束不用分号。 


### 创建`Kotlin`项目      

`Google`宣布在`Android Studio 3.0`版本会全面支持`Kotlin`，目前早就有预览版了
[Android Studio Preview](https://developer.android.com/studio/preview/index.html)(个人感觉很好用，比2.3.3版本强多了)。
直接通过`New Project`创建就可以，与创建普通`Java`项目唯一不同的是要勾选`Include Kotlin support`的选项。   


![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/studio_create_kotlin.png?raw=true)

创建完成后我们看一下`MainActivity`的代码:    
```kotlin
// 定义包
package com.charon.kotlinstudydemo

// 导入
import android.support.v7.app.AppCompatActivity
import android.os.Bundle

// 定义类，继承AppCompatActivity
class MainActivity : AppCompatActivity() {
	
	// 重写方法用overide，函数名用fun声明  参数是a: 类型的形式 ?是啥？它是指明该对象可能为null，
	// 如果有了?那在调用该方法的时候参数可以传递null进入，如果没有?传递null就会报错
    override fun onCreate(savedInstanceState: Bundle?) {
    	// super 
        super.onCreate(savedInstanceState)
        // 调用方法
        setContentView(R.layout.activity_main)
    }
}
```

我们就从`MainActivity`的代码开始介绍一些基本的语法。   

### 变量

变量可以很简单地定义成可变`var`(可读可写)和不可变`val`(只读)的变量。

`val`与`Java`中使用的`final`很相似。一个不可变对象意味着它在实例化之后就不能再去改变它的状态了。如果你需要一个这个对象修改之后的版本，
那就会再创建一个新的对象。

声明:  
```kotlin
var age: Int = 18
val name: String = "charon"
```

再提示一下:`kotlin`中每行代码结束不需要分号了，不要和`java`是的每行都带分号

字面上可以写明具体的类型。这个不是必须的，但是一个通用的`Kotlin`实践时省略变量的类型我们可以让编译器自己去推断出具体的类型:   
```kotlin
var age = 18 // int
val name = "charon" // string
var height = 180.5f // flat
var weight = 70.5 // double
```

在`Kotlin`中，一切都是对象。没有像`Java`中那样的原始基本类型。
当然，像`Integer`，`Float`或者`Boolean`等类型仍然存在，但是它们全部都会作为对象存在的。基本类型的名字和它们工作方式都是与`Java`非常相似
的，但是有一些不同之处你可能需要考虑到:    

- 数字类型中不会自动转型。举个例子，你不能给`Double`变量分配一个`Int`。必须要做一个明确的类型转换，可以使用众多的函数之一:   
	```kotlin
	private var age = 18
	private var weight = age.toFloat()
	```
- 字符（`Char`）不能直接作为一个数字来处理。在需要时我们需要把他们转换为一个数字:       
	```kotlin
	val c: Char='c'
	val i: Int = c.toInt()
	```
- 位运算也有一点不同。在`Android`中，我们经常在`flags`中使用`或`:      
	```java
	// Java
	int bitwiseOr = FLAG1 | FLAG2;
	int bitwiseAnd = FLAG1 & FLAG2;
	```

	```kotlin
	// Kotlin
	val bitwiseOr = FLAG1 or FLAG2
	val bitwiseAnd = FLAG1 and FLAG2
	```

- 一个`String`可以像数组那样访问，并且被迭代:     
	```kotlin
	var s = "charon"
	var c = s[2]

	for (a in s) {
	    Log.e("@@@", a +"");
	}
	```


##### 编译期常量

已知值的属性可以使用`const`修饰符标记为编译期常量(类似`java`中的`public static final`)。
`const`只能修复`val`不能修复`var`,这些属性需要满足以下要求:      
- 位于顶层或者是`object`的一个成员
- 用`String`或原生类型值初始化
- 没有自定义`getter`

```kotlin
// Const val are only allowed on top level or in objects
const val NAME: String = "charon"

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```


##### 后端变量`Backing Fields`.  

在`kotlin`的`getter`和`setter`是不允许本身的局部变量的，因为属性的调用也是对`get`的调用，因此会产生递归，造成内存溢出。

例如:    

```kotlin
var count = 1
var size: Int = 2
set(value) {
    Log.e("text", "count : ${count++}")
    size = if (value > 10) 15 else 0
}
```
这个例子中就会内存溢出。   

`kotlin`为此提供了一种我们要说的后端变量，也就是`field`。编译器会检查函数体，如果使用到了它，就会生成一个后端变量，否则就不会生成。
我们在使用的时候，用`field`代替属性本身进行操作。


##### 延迟初始化   

我们说过，在类内声明的属性必须初始化，如果设置非`null`的属性，应该将此属性在构造器内进行初始化。
假如想在类内声明一个`null`属性，在需要时再进行初始化（最典型的就是懒汉式单例模式），与`Kotlin`的规则是相背的，此时我们可以声明一个属性并
延迟其初始化，此属性用`lateinit`修饰符修饰。

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var name : String

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        var test = MainActivity()
        // 要先调用方法让其初始化
        test.init()
        // 再使用其属性
        Log.e("@@@", test.name)
    }

    fun init() {
        // 延迟初始化
        name = "charon"
    }
}
```
需要注意的是，我们在使用的时候，一定要确保属性是被初始化过的，通常先调用初始化方法，否则会有异常。
如果只是用`lateinit`声明了，但是还没有调用初始化方法就使用，哪怕你判断了该变量是否为`null`也是会`crash`的。
```kotlin
private lateinit var test: String

private fun switchFragment(position: Int) {
    if (test == null) {
        LogUtil.e("@@@", "test is null")
    } else {
        LogUtil.e("@@@", "test is not null")
        check(test)
    }
}
```
会报`kotlin.UninitializedPropertyAccessException: lateinit property test has not been initialized`

除了使用`lateinit`外还可以使用`by lazy {}`效果是一样的：   
```kotlin
private val test by lazy { "haha" }

private fun switchFragment(position: Int) {
    if (test == null) {
        LogUtil.e("@@@", "test is null")
    } else {
        LogUtil.e("@@@", "test is not null ${test}")
        check(test)
    }
}    
```
执行结果:   
```
test is not null haha
```

那`lateinit`和`by lazy`有什么区别呢？    

- `by lazy{}`只能用在`val`类型而`lateinit`只能用在`var`类型
- `lateinit`不能用在可空的属性上和`java`的基本类型上,否则会报`lateinit`错误   


### 类的定义:使用`class`关键字   

类可以包含:  
- 构造函数和初始化块
- 函数
- 属性
- 嵌套类和内部类
- 对象声明


```kotlin
class MainActivity{

}
```

如果有参数的话你只需要在类名后面写上它的参数，如果这个类没有任何内容可以省略大括号：
```kotlin
class Person(name: String, age: Int)
```

##### 创建类的实例  

```kotlin
val person = Person("charon", 18)
```

上面的类有一个默认的构造函数。

注意:创建类的实例不用`new`了啊。


### 构造函数 

在`Kotlin`中的一个类可以有一个主构造函数和一个或多个次构造函数。

##### 主构造函数

主构造函数是类头的一部分:它跟在类名（和可选的类型参数）后:   
```kotlin
class Person constructor(name: String, surname: String) {
}
```
如果主构造函数没有任何注解或者可见性修饰符，可以省略`constructor`关键字:   
```kotlin
class Person(name: String, surname: String) {
}
```

主构造函数不能包含任何的代码。初始化的代码可以放到以`init`关键字作为前缀的初始化块中:    

```kotlin
class Person constructor(name: String, surname: String) {
    init {
        print("name is $name and surname is $surname")
    }
}
```

如果构造函数有注解或可见性修饰符，那么`constructor`关键字是必需的，并且这些修饰符在它前面:   
```kotlin
class Person private @Inject constructor(name: String, surname: String) {
    init {
        print("name is $name and surname is $surname")
    }
}
```

##### 次构造函数

类也可以声明前缀有`constructor`的次构造函数: 
```kotlin
class Person{
    constructor(name: String) {
        print("name is $name")
    }
}
```

如果类有一个主构造函数，每个次构造函数都需要委托给主构造函数(不然会报错)， 可以直接委托或者通过别的次构造函数间接委托。
委托到同一个类的另一个构造函数用`this`关键字即可:    
```kotlin
class Person constructor(name: String) {
    constructor(name: String, surName: String) : this(name) {
        Log.d("@@@", "name is : $name surName is : $surName")
    }
}
```
使用该对象:   
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Person("charon", "chui")
    }
}
```
就会在`logcat`上打印:   
`09-20 16:51:19.738 6010-6010/com.charon.kotlinstudydemo D/@@@: name is : charon surName is : chui`

如果一个非抽象类没有声明任何（主或次）构造函数，它会有一个生成的不带参数的主构造函数。构造函数的可见性是`public`。
如果你不希望你的类有一个公有构造函数，你需要声明一个带有非默认可见性的空的主构造函数：

```kotlin
class Person private constructor(name: String) {
}
```


### 接口:使用`interface`关键字    

```kotlin
interface FlyingAnimal {
    fun fly()
}
```

### 函数:通过`fun`关键字定义

```kotlin
fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
}
```
如果你没有指定它的返回值，它就会返回`Unit`与`Java`中的`void`类似，但是`Unit`是一个真正的对象。`Unit`可以省略，
你当然也可以指定任何其它的返回类型:      
```kotlin
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
```

然而如果返回的结果可以使用一个表达式计算出来，你可以不使用括号而是使用等号:         
```kotlin
fun add(x: Int,y: Int) : Int = x + y
```

我们可以给参数指定一个默认值使得它们变得可选，这是非常有帮助的。这里有一个例子，在`Activity`中创建了一个函数用来`Toast`一段信息:    
```kotlin
fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, length).show()
}
```
上面代码中第二个参数`length`指定了一个默认值。这意味着你调用的时候可以传入第二个值或者不传，这样可以避免你需要的重载函数：

```kotlin
toast("Hello")
toast("Hello", Toast.LENGTH_LONG)
```

##### 自定义`get set`方法:   

`Kotlin`会默认创建`set get`方法，我们也可以自定义`get set`方法:
`kotlin`预留了一个在`set`和`get`中访问的变量`field`关键字:  

```kotlin
class Person constructor() {
    var name: String = ""
        get() = field
        set(value) {
            field = "$value"
        }

    var age: Int = 0
        get() = field
        set(value) {
            field = value
        }
}
```
按照惯例`set`参数的名称是`value`，但是如果你喜欢你可以选择一个不同的名称。


##### 可变长参数函数:使用`vararg`关键字

```kotlin
fun vars(vararg v:Int){
    for(vt in v){
        print(vt)
    }
}

// 测试
fun main(args: Array<String>) {
    vars(1,2,3,4,5)  // 输出12345
}
```


### 注释  

和`Java`差不多

```kotlin

// 这是一个行注释

/* 这是一个多行的
   块注释。 */
```



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

