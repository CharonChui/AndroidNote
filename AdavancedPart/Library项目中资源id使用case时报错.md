Library项目中资源id使用case时报错
===

在平时开发的过程中对一些常量的判断，都是使用`switch case`语句，因为`switch case`比`if else`的效率稍高。     
但是也遇到了问题，就是在`library`中使用时会报错:      
```java
public void testClick(View view) {
    int id = view.getId();
    switch (id) {
        case R.id.bt_pull:

            break;
        case R.id.bt_push:

            break;
    }
}
```
我们来看一下错误提示:     

![Image](https://github.com/CharonChui/Pictures/blob/master/library_r.error.png?raw=true)

意思就是在`Android Library`的`Module`中的`switch`语句不能使用资源`ID`.这是为什么呢？ 我们都知道`R`文件中都是常量啊，`public static final`的，为什么不能用在`switch`语句中，继续看下去:      
```
Resource IDs are non final in th library projects since SDK tools r14, means that the library code cannot treat this IDs as constants.
```
说的灰常明白了，也就是说从14开始，library中的资源`id`就不是`final`类型的了，所以不是常量了。       

在一个常规的`Android`项目中，资源`R`文件中的常量都是如下这样声明的:      
`public static final int main=0x7f030004;`
然后，从`ADT`14开始，在`library`项目中，它们将被这样声明:       
`public static int main=0x7f030004;`

也就是说在`library`项目中这些常量不是`final`的了。为什么这样做的原因也非常简单：当多个`library`项目结合时，这些字段的实际值(必须唯一的值)可能会冲突。在`ADT`14之前，所有的字段都是`final`的，这样就导致了一个结果，就是所有的`libraries`不论何时在被使用时都必须把他们所有的资源和相关的`Java`代码都伴随着主项目一起重新编译。这样做是不合理的，因为它会导致编译非常慢。这也会阻碍一些没有源码的`libraray`项目进行发布，极大的限制了`library`项目的使用范围。      

那些字段不再是`final`的原因就是意味着`library jars`可以被编译一次，然后在其他项目中直接被服用。也就是意味着允许发布二进制版本的`library`项目(r15中开始支持)，这让编译变的更快。

然而，这对`library`的源码会有一个影响。类似下面这样的代码就将无法编译:    
```java
public void testClick(View view) {
    int id = view.getId();
    switch (id) {
        case R.id.bt_pull:

            break;
        case R.id.bt_push:

            break;
    }
}
```
这是因为`swtich`语句要求所有`case`后的字段，例如`R.di.bt_pull`在编译的时候都应该是常量(就像可以直接把值拷贝进`.class`文件中)

当然针对这个问题的解决方法也很简单：就是把`switch`语句改成`if-else`语句。幸运的是，在`eclise`中这是非常简单的。只需要在`switch`的关键字上按`Ctrl+1`（`Mac`上是`Cmd+1`)。(`eclipse`都辣么方便了，`Studio`更不用说了啊，直接按修复的快捷键就阔以了)。      
上面的代码就将变成如下:     

```java
public void testClick(View view) {
    int id = view.getId();
    if (id == R.id.bt_pull) {
    } else if (id == R.id.bt_push) {
    }
}
```
这在UI代码和性能的影响通常是微不足道的 - -！。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

