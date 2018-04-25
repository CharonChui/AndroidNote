Kotlin学习教程(六)
===

### 注解  

注解是将元数据附加到代码的方法。要声明注解，请将`annotation`修饰符放在类的前面:  

```kotlin
annotation class Fancy
```

注解的附加属性可以通过用元注解标注注解类来指定:  

- `@Target`指定可以用该注解标注的元素的可能的类型(类、函数、属性、表达式等)
- `@Retention`指定该注解是否存储在编译后的`class`文件中，以及它在运行时能否通过反射可见(默认都是`true`)
- `@Repeatable`允许在单个元素上多次使用相同的该注解
- `@MustBeDocumented`指定该注解是公有`API`的一部分，并且应该包含在生成的`API`文档中显示的类或方法的签名中


```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### 用法 

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

如果需要对类的主构造函数进行标注，则需要在构造函数声明中添加`constructor`关键字，并将注解添加到其前面:   

```kotlin
class Foo @Inject constructor(dependency: MyDependency) {
    // ……
}
```

### 反射

反射是这样的一组语言和库功能，它允许在运行时自省你的程序的结构。
`Kotlin`让语言中的函数和属性做为一等公民、并对其自省（即在运行时获悉一个名称或者一个属性或函数的类型）与简单地使用函数式或响应式风格紧密相关。

在`Java`平台上，使用反射功能所需的运行时组件作为单独的`JAR`文件(`kotlin-reflect.jar`)分发。这样做是为了减少不使用反射功能的应用程序所需的
运行时库的大小。如果你需要使用反射，请确保该`.jar`文件添加到项目的`classpath`中。


### 类引用

最基本的反射功能是获取`Kotlin`类的运行时引用。要获取对静态已知的`Kotlin`类的引用，可以使用类字面值语法:   

```kotlin
val c = MyClass::class
```
该引用是`KClass`类型的值。
通过使用对象作为接收者，可以用相同的`::class`语法获取指定对象的类的引用:   

```kotlin
val widget: Widget = ……
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

当我们有一个命名函数声明如下:  

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```
我们可以很容易地直接调用它`isOdd(5)`，但是我们也可以把它作为一个值传递。例如传给另一个函数。为此我们使用`::`操作符:    

```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 输出 [1, 3]
```

### 扩展     

扩展是`kotlin`中非常重要的一个特性，它能让我们对一些已有的类进行功能增加、简化，使他们更好的应对我们的需求。

```kotlin
//  对Context的扩展，增加了toast方法。为了更好的看到效果，我还加了一段log日志
fun Context.toast(msg : String){
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()
    Log.d("text", "Toast msg : $msg")
}

// Activity类，由于所有Activity都是Context的子类，所以可以直接使用扩展的toast方法
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        ......
        toast("hello, Extension")
    }
}

// 输出
Toast msg : hello, Extension
```

按照通常的做法，会写一个`ToastUtils`工具类，或者在`BaseActivity`中实现`toast`。但是使用扩展函数就会简单很。 
上面的例子就是在`Context`中添加新的方法，让我们以更简单的方式去显示`toast`，并且不用传入任何`context`参数，可以被任何`Context`或者
它的子类调用。我们可以在任何地方(例如一个工具类文件中)声明这个函数，然后在`Activity`中将它作为普通方法来直接调用。当然了`Anko`中已经包括了
自己的`toast`扩展函数。有关`Anko`后面会讲到。

`Kotlin`扩展函数允许我们在不改变已有类的情况下，为类添加新的函数。
扩展函数是指对类的方法进行扩展，写法和定义方法类似，但是要声明目标类，也就是对哪个类进行扩展，`kotlin`中称之为`Top Level`。
扩展函数表现得就像是属于这个类的一样，而且我们可以使用`this`关键字和调用所有`public`方法。
扩展函数可以在已有类中添加新的方法，不会对原类做修改，扩展函数定义形式:   
```kotlin
fun receiverType.functionName(params){
    body
}
receiverType：表示函数的接收者，也就是函数扩展的对象
functionName：扩展函数的名称
params：扩展函数的参数，可以为NULL
```

在上面我们举的扩展的例子就是扩展函数.其中`Context`就是目标类`Top Level`，我们把它放到方法名前，用点`.`表示从属关系。在方法体中用关键字
`this`对本体进行调用。和普通方法一样，如果有返回值，在方法后面跟上返回类型，我这里没有返回值，所以直接省略了。


##### 扩展属性


扩展属性和扩展方法类似，是对目标类的属性进行扩展。扩展属性也会有`set`和`get`方法，并且要求实现这两个方法，不然会提示编译错误。
因为扩展并不是在目标类上增加了这个属性，所以目标类其实是不持有这个属性的，我们通过`get`和`set`对这个属性进行读写操作的时候也不能使用
`field`指代属性本体。可以使用`this`，依然表示的目标类。

```kotlin
// 扩展了一个属性paddingH
var View.panddingH : Int
    get() = (paddingLeft + paddingRight) / 2
    set(value) {
        setPadding(value, paddingTop, value, paddingBottom)
    }

// 设置值
text.panddingH = 100
```

给`View`扩展了一个属性`paddingH`，并给属性增加了`set`和`get`方法，然后可以在`activity`中通过`textview`调用。


##### 静态扩展

`kotlin`中的静态用关键字`companion`表示，但是它不是修饰属性或方法，而是定义一个方法块，在方法块中的所有方法和属性都是静态的，
这样就将静态部分统一包装了起来。静态部分的访问和`java`一致，直接使用类名+静态属性/方法名调用。   

```kotlin
// 定义静态部分
class Extension {
    companion object part{
        var name = "Extension"
    }
}

// 通过类名+属性名直接调用
toast("hello, ${Extension.name}")

// 输出
Toast msg : hello, Extension
```
上面例子中，`companion object`一起是修饰关键字，`part`是方法块的名称。其中方法块名称`part`可以省略，如果省略的话，默认缺省名为
`Companion`

静态的扩展和普通的扩展类似，但是在目标类要加上静态方法块的名称，所以如果我们要对一个静态部分扩展，就要先知道静态方法块的名称才行。

```kotlin
class Extension {
    companion object part{
        var name = "Extension"
    }
}

// part为静态方法块名称
fun Extension.part.upCase() : String{
    return name.toUpperCase()
}

// 调用一下
toast("hello, ${Extension.name}")
toast("hello, ${Extension.upCase()}")

//输出
Toast msg : hello, Extension
Toast msg : hello, EXTENSION
```



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

