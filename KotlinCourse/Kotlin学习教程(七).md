Kotlin学习教程(七)
===

这篇文章主要学习下`lambda`表达式。因为后续一些例子会用到。


> “Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示闭包（注意和数学传统意义上的不同）。

`Java 8`的一个大亮点是引入`Lambda`表达式，使用它设计的代码会更加简洁。

```java
// 没有使用Lambda的老方法:   
button.addActionListener(new ActionListener(){
	public void actionPerformed(ActionEvent ae){
		System.out.println("Actiondetected");
	}
});
// 使用Lambda:  
button.addActionListener(()->{ 
	System.out.println("Actiondetected");
});


// 不采用Lambda的老方法: 
Runnable runnable1=new Runnable(){
	@Override
	public void run(){
		System.out.println("RunningwithoutLambda");
	}
};
// 使用Lambda:   
Runnable runnable2=()->{
	System.out.println("RunningfromLambda");
};
```

`Lambda`能让代码更简洁，而主打简洁的`Kotlin`怎么可能不支持呢？ 当然会支持。    

下面来看看一个简短的概述：

lambda 表达式总是被大括号括着，
其参数(如果有的话)在 -> 之前声明(参数类型可以省略)，
函数体(如果存在的话)在 -> 后面。


Lambda表达式是定义匿名函数的简单方法。由于Lambda表达式避免在抽象类或接口中编写明确的函数声明，进而也避免了类的实现部分，所以它是非常有用的。在Kotlin语言中，可以将一函数作为另一函数的参数。例如，可以将需要回调(callback)的函数简单化为：


Lambda表达式由箭头左侧函数的参数（在圆括号里的内容）定义的，将值返回到箭头右侧。在这个例子中，得到的View返回给Unit(无参数)。按此思路，可以上述代码略做简化：
view.setOnClickListener({ view -> toast("Click")})
在定义函数时，必须在箭头的左侧用方括号，并指定参数值，而函数的执行代码在箭头右侧。如果左侧不使用参数，甚至可以省去左侧部分：
view.setOnClickListener({ toast("Click") })
如果函数的最后一个参数是一个函数的话，可以将作为参数的函数移到圆括号外面：
view.setOnClickListener() { toast("Click") }




先看一个例子:    

```kotlin
fun compare(a: String, b: String): Boolean {
    return a.length < b.length
}
max(strings, compare)
```
就是找出`strings`里面最长的那个。但是我个人觉得`compare`还是很碍眼的，因为我并不想在后面引用他，那我怎么办呢，就是用“匿名函数”方式。
```kotlin
max(strings, (a,b)->{a.length < b.length})
```

`(a,b)->{a.length < b.length}`就是一个没有名字的函数，直接作为参数赋给`max`方法的第二个参数。但这个方法有很多东西都没有写明，如:   

- 参数的类型
- 返回值的类型

但这些真的必要吗？`a.length < b.length`很明显返回一个`Boolean`的值，再就是`max`的定义中肯定也定义了这个函数的参数类型和返回值类型。这么明显的事为什么不让计算机自己去做而要让人写代码去做呢？这就是匿名函数的好处了。到这里，我们已经和`Lambda`很接近了。

```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

`Lambda`表达式就是被大括号括着的那一部分，在`->`符号之前有参数声明，函数体跟在一个`->`符号之后。
而且此`Lambda`表达式之前有一个匿名的函数声明(在此例中两个`Int`型的输入，一个`Int`型的返回值)，这个声明是可以不使用的。
则此`Lambda`表达式变成`val sum = { x: Int, y: Int -> x + y }`，此时`Lambda`表达式会根据主体中的最后一个（或可能是单个）表达式会视为返回值。
当然，在某些特定情况下，`x`、`y`的类型了是可以推断的，所以`val sum = { x, y -> x + y }`。




一、Lambda的定义

看上面的说法有点抽象，有点不明所以，我们先来看一个栗子

class Num {
    fun logic(a: Int, b: Int, calc: (Int, Int) -> Int){
        println("calc : ${calc(a,b)}")
    }
}

fun main(args : Array<String>){
    val num = Num()
    num.logic(1, 2, {x,y -> x+y})
}

// 输出
calc : 3
这个栗子和上一篇文章很像，只是在调用的时候改成了Lambda方式：num.logic(1, 2, {x,y -> x+y})，其中{x,y -> x+y}就是我们今天要讲的Lambda表达式，它的完整格式应该是这样

{ x: Int, y: Int -> x + y }
写成java代码是这个样子：

public int sum(int x, int y){
    return x+y;
}
可以很明显看出有几个规则：

参数写在->左边，格式与普通函数的参数格式一样，多个参数用逗号,分割
参数的类型可选，可忽略，编辑器会根据上下文推断（这就和普通函数不一样了吧，有种“我知道你懂得，所以就不写了”的感觉有木有，知己的感觉啊……）
函数体跟在->右边
Lambda表达式总是被大括号{}包围着
二、Lambda中的一些约定

如果 Kotlin 可以自己计算出签名，它允许我们不声明唯一的参数，并且将隐含地为我们声明其名称为 it。其实通常情况下它都可以自己计算出签名，也就是说，如果函数字面值只有一个参数， 那么它的声明可以省略（连同 ->），其名称是 it

fun oneParams(one : (Int) -> Int){
    println("oneParams : ${one(5)}")
}

fun main(args : Array<String>){
    val num = Num()
    num.oneParams({it * 2})
}

// 输出
oneParams : 10
说到省略，其实还有一种情况可以省略->，大家应该也能想到，就是无参函数。

fun empty(emptyM : () -> Unit){
    emptyM()
}

fun main(args : Array<String>){
    val num = Num()
    num.empty({println("empty method")})
}

// 输出
empty method
如果Lambda中的某个参数没有用到，可以用下划线_代替，也就是说，省了好多请名字的脑细胞有木有！这个特性从1.1开始可以使用，现在你看到的时候应该已经不止1.1了吧，所以这个限制看看就好~

fun unusedParams(unused : (Int,Int) -> Int){
    println("unusedParams : ${unused(5,10)}")
}

fun main(args : Array<String>){
    val num = Num()
    num.unusedParams { _, used -> used * 2 }
}

// 输出
unusedParams : 20
如果函数的最后一个参数是一个函数，那么我们在用Lambda表达最后一个函数参数的时候，可以把它放在括号()外面，所以下面的写法是等价的。

class Num {
    fun logic(a: Int, b: Int, calc: (Int, Int) -> Int){
        println("calc : ${calc(a,b)}")
    }
    fun sum(a: Int, b: Int) = a + b
}

fun main(args : Array<String>){
    val num = Num()
    // 写法1
    num.logic(1, 2, {x : Int,y : Int -> x+y})
    // 写法2
    num.logic(1, 2){x : Int,y : Int -> x+y}
    // 写法3
    num.logic(1, 2){x,y -> x+y}
}
那么这么写有什么好处呢？难道只是位置变了一下？当然不是，不要忘记，Lambda的->后面是方法体，也就是很多时候不是想栗子中这样只有一行，如果有多行的话，体会一下他们的区别：

num.logic(1, 2, {x,y ->
    println("extra line")
    x+y
})

num.logic(1, 2){x,y ->
    println("extra line")
    x+y
}
是不是感觉下面的写法要优雅很多，也明确很多？

其实在上面一点应该已经能看出，如果有需要的话，Lambda会隐式的返回最后一个表达式的值，就像上面的最后一行x+y。当然，我们也可以显示的表达返回值，下面的写法还是一样的：

// 写法1
num.unusedParams { _, used ->
    println("print first")
    return@unusedParams used * 2
}

// 写法2
num.unusedParams { _, used ->
    println("print first")
    used * 2
}
三、匿名函数

看了这么多，我们发现一个问题，Lambda的返回类型全是自动推断的，虽然很人性化，但是有时候我们就是想自己指定类型怎么办？当然是有办法的，那就是匿名函数。

fun(x:Int, y:Int):Int{return x+y}
匿名函数看起来非常像一个常规函数声明，除了其名称省略了。其函数体可以是表达式（如上所示）或代码块。由于这已经不算正经的Lambda，所以它不需要被大括号{}包裹，也不能像Lambda一样写在括号()外，而调用也是直接调用。

num.logic(1, 2, fun(x:Int, y:Int):Int{return x+y})
四、闭包

Lambda 表达式或者匿名函数（以及局部函数和对象表达式） 可以访问其 闭包 ，即在外部作用域中声明的变量。 与 Java 不同的是可以修改闭包中捕获的变量：

var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)






































---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
