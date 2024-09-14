# Jetpack Compose_Modifier

在传统开发中，使用XML文件来描述组件的样式，而Jetpack Compose设计了一个精妙的东西，它叫作Modifier。Modifier允许我们通过链式调用的写法来为组件应用一系列的样式设置，如边距、字体、位移等。在Compose中，每个基础的Composable组件都有一个modifier参数，通过传入自定义的Modifier来修改组件的样式。



- Modifier.size

设置被修饰组件的大小。

- Modifier.background

添加背景色，支持使用color设置纯色背景，也可以使用brush设置渐变色背景。     

传统视图中View的background属性可以用来设置图片格式的背景，Compose的background修饰符只能设置颜色背景，图片背景需要使用Box布局配合Image组件实现。   

- Modifier.fillMaxSize

组件在高度或宽度上填满父空间，此时可以使用fillMaxXXX系列方法：  

Box(Modifier.fillMaxSize().background(Color.Red))
Box(Modifier.fillMaxWidth().height(60.dp))
Box(Modifier.fillMaxHeight().width(60.dp))


```kotlin
setContent {
    Image(
        painter = painterResource(id = R.drawable.ic_launcher_background),
        contentDescription = null,
        modifier = Modifier
            .size(width = 60.dp, height = 50.dp)
            .background(
                brush = Brush.verticalGradient(
                    colors = listOf(
                        Color.Red,
                        Color.Yellow,
                        Color.White
                    )
                )
            )
    )
}
```


- Modifier.border & Modifier.padding

border用来为组件添加边框。   
边框可以指定颜色、粗细以及通过Shape指定形状，比如圆角矩形等。    

padding用来为被修饰组件增加间隙，可以在border前后各插入一个padding用来为为组件增加对外和对内的间距，例如:   

```kotlin
@Preview
@Composable
fun BoxTest() {
    Box(
        modifier = Modifier
            .padding(10.dp)
            .border(2.dp, Color.Red, shape = RoundedCornerShape(2.dp))
            .padding(20.dp)
    ) {
        Spacer(
            modifier = Modifier
                .size(10.dp, 10.dp)
                .background(Color.Green)
        )
    }
}
```

相对于传统布局有Margin和Padding之分，Compose中只有padding这一种修饰符，根据在调用链中的位置不同发挥不同的作用。     

Modifier调用顺序会影响最终UI的呈现效果。    
这是因为Modifier会由于调用顺序不同而产生不同的Modifier链，Compose会按照Modifier链来顺序完成页面测量布局和渲染。      


## 作用域限定

Compose充分发挥了Kotlin的语法特性，让某些Modifier修饰符只能在特定作用域中使用，有利于类型安全的调用它们。      

所谓的作用域，在Kotlin中就是一个带有Receiver的代码块。     

例如Box组件参数中的content就是一个Receiver类型为BoxScope的代码块，因此其子组件都处于BoxScope作用域中。    

```kotlin
inline fun Box(
    modifier: Modifier = Modifier, 
    contentAlignment: Alignment = Alignment.TopStart,
    propagateMinConstraints: Boolean = false,
    content: @Composable BoxScope.() -> Unit
)    

Box {
    // 该代码块Receiver类型即为BoxScope
}
```


- matchParentSize

matchParentSize是只能在BoxScope中使用的作用域限定修饰符。     
当使用matchParentSize设置尺寸时，可以保证当前组件的尺寸与父组件相同。     
而父组件默认的是wrapContent，会根据子组件的尺寸确定自身的尺寸。   

而如果使用fillMaxSize取代matchParentSize，那么该组件的尺寸会被设置为父组件所允许的最大尺寸，这样会导致铺满整个屏幕。 
```kotlin
@Composable 
// 只把UserInfo区域设置背景
fun MatchParentModifierDemo() {
    Box {
        Box(modifier = Modifier
            .matchParentSize()
            .background(Color.RED)
        )
        UserInfo()
    }
}


// 使用fillMaxSize取代matchParentSize，那么该组件的尺寸会被设置为父组件所允许的最大尺寸，这样会导致整个背景铺满整个屏幕
@Composable
fun MatchParentModifierDemo() {
    Box {
        Box(modifier = Modifier
            .fillMaxSize()
            .background(Color.READ)
        )
        UserInfo()
    }
}
```


- weight

在RowScope和ColumnScope中，可以使用专属的weight修饰符来设置尺寸。      

与size修饰符不同的是，weight修饰符允许组件通过百分比设置尺寸，也就是允许组件可以自适应适配各种屏幕尺寸。    








































