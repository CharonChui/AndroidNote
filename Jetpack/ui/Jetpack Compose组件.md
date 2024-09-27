# Jetpack Compose组件

### Text组件

```kotlin
Text(text = "Hello World")
Text(
    text = stringResource(id = R.string.next),
    style = TextStyle(
        fontSize = 25.sp,
        fontWeight = FontWeight.Bold,
        lineHeight = 35.sp,
        color = Color.Red,
        textDecoration = TextDecoration.LineThrough,
        fontStyle = FontStyle.Italic
    )
)
```

#### AnnotatedString多样式文字

在很多应用场景中，我们需要在一段文字中对局部内容应用特别格式以示突出，比如一个超链接或者一个电话号码等，此时需要用到AnnotatedString。AnnotatedString是一个数据类，除了文本值，它还包含了一个SpanStyle和ParagraphStyle的Range列表。SpanStyle用于描述在文本中子串的文字样式，ParagraphStyle则用于描述文本中子串的段落样式，Range确定子串的范围。

```kotlin
val annotedText = buildAnnotatedString {
    withStyle(style = SpanStyle(fontSize = 23.sp)) {
        pushStringAnnotation(tag = "url", annotation = "https://www.baidu.com")
        append("haha")
    }

    withStyle(SpanStyle(fontSize = 30.sp)) {
        append("A")
    }
}

ClickableText(
    text = annotedText,
    onClick = { offset ->
        annotedText.getStringAnnotations(tag = "url", start = offset, end = offset)
            .firstOrNull()?.let {
            Log.e("@@@", it.item)
        }
    }
)
```

Compose提供了一种可点击文本组件ClickedText，可以响应我们对文字的点击，并返回点击位置。可以让AnnotatdString子串在相应的ClickedText中点击后，做出不同的动作。





### SelectionContainer

选中文字Text自身默认是不能被长按选择的，否则在Button中使用时，又会出现那种“可粘贴的Button”的例子。       
Compose提供了专门的SelectionContainer组件，对包裹的Text进行选中。可见Compose在组件设计上，将关注点分离的原则发挥到了极致。


```kotlin
SelectionContainer {
    Text("hahah")
}
```



### TextField输入框


TextField有两种风格，一种是默认的，也就是filled，另一种是OutlinedTextField。      
```kotlin
var text by remember { mutableStateOf("") }

TextField(value = text, onValueChange = {
    text = it
}, label = { Text("请输入用户名") })
```

这个text是一个可以变化的文本，用来显示TextField输入框中当前输入的文本内容。           在onValueChange回调中可以获取来自软键盘的最新输入，我们利用这个信息来更新可变状态text，驱动界面刷新显示最新的输入文本。


***来自软键盘的输入内容不会直接更新TextField, TextField需要通过观察额外的状态更新自身，这也体现了声明式UI中“状态驱动UI”的基本理念。***





### Column组件

很多产品中都有展示一组数据的需求场景，如果数据数量是可以枚举的，则仅需通过Column组件来枚举列出。 


然而很多时候，列表中的项目会非常多，例如通讯录、短信、音乐列表等，我们需要滑动列表来查看所有的内容，可以通过Column的Modifier添加verticalScroll()方法来让列表实现滑动。     


#### LazyComposables

给Column的Modifier添加verticalScroll()方法可以让列表实现滑动。    

但是如果列表过长，众多的内容会占用大量的内存。然而更多的内容对于用户其实都是不可见的，没必要记载到内存。     

所以Compose提供了专门用于处理长列表的组件，这些组件指挥在我们能看到的列表部分进行重组和布局，它们分别是LazyColumn和LazyRow。其作用类似于传统视图中的ListView或者RecyclerView。   




