Kotlin学习教程(三)
===

前面介绍了基本语法和编码规范后，接下来学习下基本类型。  

在`Kotlin`中，所有东西都是对象，在这个意义上讲我们可以在任何变量上调用成员函数和属性。 一些类型可以有特殊的内部表示——例如，
数字、字符和布尔值可以在运行时表示为原生类型值，但是对于用户来说，它们看起来就像普通的类。 在本节中，我们会描述`Kotlin`中使用的基本类型:
数字、字符、布尔值、数组与字符串。

### 数字  

`Kotlin`处理数字在某种程度上接近`Java`,但是并不完全相同。例如，对于数字没有隐式拓宽转换（如`Java`中`int`可以隐式转换为`long`)，
另外有些情况的字面值略有不同。

`Kotlin`提供了如下的内置类型来表示数字:    

```
Type    Bit width
Double  64
Float   32
Long    64
Int     32
Short   16
Byte    8
````

注意在`Kotlin`中字符不是数字,字符用`Char`类型表示。它们不能直接当作数字


### 字面常量

数值常量字面值有以下几种:   
- 十进制:123
- `Long`类型用大写`L`标记:`123L`
- 十六进制:`0x0F`
- 二进制:`0b00001011`

注意: 不支持八进制

`Kotlin`同样支持浮点数的常规表示方法:   

默认`double`:123.5、123.5e10，`Float`用`f`或者`F`标记:`123.5f`

你可以使用下划线使数字常量更易读

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 引用相等

引用相等由`===`以及其否定形式`!===`操作判断。`a === b`当且仅当`a`和`b`指向同一个对象时求值为`true`。

### 结构相等

结构相等由`==`以及其否定形式`!==`操作判断。按照惯例，像`a == b`这样的表达式会翻译成      
`a?.equals(b) ?: (b === null)`
也就是说如果`a`不是`null`则调用`equals(Any?)`函数，否则即`a`是`null`检查`b`是否与`null`引用相等。

```kotlin
val a: Int = 10000
print(a === a) // 输出“true”
val boxedA: Int? = agaomnh
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

另一方面，它保留了相等性:
```kotlin
val a: Int = 10000
print(a == a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 输出“true”
```

### 显式转换

由于不同的表示方式，较小类型并不是较大类型的子类型。 如果它们是的话，就会出现下述问题：

```kotlin
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(a == b) // 惊！这将输出“false”鉴于 Long 的 equals() 检测其他部分也是 Long
```

所以同一性还有相等性都会在所有地方悄无声息地失去。
因此较小的类型不能隐式转换为较大的类型。 这意味着在不进行显式转换的情况下我们不能把`Byte`型值赋给一个`Int`变量。

```kotlin
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```

我们可以显式转换来拓宽数字
```kotlin
val i: Int = b.toInt() // OK: 显式拓宽
```

每个数字类型支持如下的转换:

```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

### 运算

这是完整的位运算列表(只用于`Int`和`Long`):

```kotlin
shl(bits) – 有符号左移 (Java 的 <<)
shr(bits) – 有符号右移 (Java 的 >>)
ushr(bits) – 无符号右移 (Java 的 >>>)
and(bits) – 位与
or(bits) – 位或
xor(bits) – 位异或
inv() – 位非
相等性检测：a == b 与 a != b
比较操作符：a < b、 a > b、 a <= b、 a >= b
区间实例以及区间检测：a..b、 x in a..b、 x !in a..b
|| – 短路逻辑或
&& – 短路逻辑与
! - 逻辑非
```


### 字符串

字符串用`String`类型表示。字符串是不可变的。字符串的元素——字符可以使用索引运算符访问:`s[i]`。可以用`for`循环迭代字符串:   

```kotlin
for (c in str) {
    println(c)
}
```
`Kotlin`有两种类型的字符串字面值: 转义字符串可以有转义字符，以及原生字符串可以包含换行和任意文本。转义字符串很像`Java`字符串:
```kotlin
val s = "Hello, world!\n"
```
转义采用传统的反斜杠方式。

原生字符串 使用三个引号`"""`分界符括起来，内部没有转义并且可以包含换行和任何其他字符:   

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```
你可以通过`trimMargin()`函数去除前导空格:   

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

### 字符串模板

字符串可以包含模板表达式，即一些小段代码，会求值并把结果合并到字符串中。模板表达式以美元符`$`开头，由一个简单的名字构成:  

```kotlin
val i = 10
val s = "i = $i" // 求值结果为 "i = 10"
```

或者用花括号括起来的任意表达式:
```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // 求值结果为 "abc.length is 3"
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

