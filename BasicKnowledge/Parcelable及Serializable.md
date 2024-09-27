Parcelable及Serializable
===



### Serializable

在Java中Serializable接口是一个允许将对象转换为字节流(序列化)然后重新构造回对象(反序列化)的标记接口。   

它会使用反射，并且会创建许多临时对象，导致内存使用率升高，并可能产生性能问题。  


`Serializable`的作用是为了保存对象的属性到本地文件、数据库、网络流、`rmi`以方便数据传输，
当然这种传输可以是程序内的也可以是两个程序间的。而`Parcelable`的设计初衷是因为`Serializable`效率过慢，
为了在程序内不同组件间以及不同`Android`程序间(`AIDL`)高效的传输数据而设计，这些数据仅在内存中存在，`Parcelable`是通过`IBinder`通信的消息的载体。

`Parcelable`的性能比`Serializable`好，在内存开销方面较小，所以在内存间数据传输时推荐使用`Parcelable`，
如`activity`间传输数据，而`Serializable`可将数据持久化方便保存，所以在需要保存或网络传输数据时选择
`Serializable`，因为`android`不同版本`Parcelable`可能不同，所以不推荐使用`Parcelable`进行数据持久化。

Parcelable不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。


### Parcelable实现

```kotlin
data class Developer(val name: String, val age: Int) : Parcelable {

    constructor(parcel: Parcel) : this(
        parcel.readString(),
        parcel.readInt()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    // code removed for brevity

    companion object CREATOR : Parcelable.Creator<Developer> {
        override fun createFromParcel(parcel: Parcel): Developer {
            return Developer(parcel)
        }

        override fun newArray(size: Int): Array<Developer?> {
            return arrayOfNulls(size)
        }
    }
}

```
上面是实现Parcelable的代码，可以看到有很多重复的代码。      
为了避免写这些重复的代码，可以使用kotlin-parcelize插件，并在类上使用@Parcelize注解。

```kotlin
@Parcelize
data class Developer(val name: String, val age: Int) : Parcelable
```

当在一个类上声明`@Parcelize`注解后，就会自动生成对应的代码。     





Parcelable不会使用反射，并且在序列化过程中会产生更少的临时对象，这样就会减少垃圾回收的压力:    

- Parcelable不会使用反射
- Parcelable是Android平台特定的接口

所以Parcelable比Serializable更快。    


区别:    
- Parcelable is faster than serializable interface
- Parcelable interface takes more time for implemetation compared to serializable interface
- serializable interface is easier to implement
- serializable interface create a lot of temporary objects and cause quite a bit of garbage collection
- Parcelable array can be pass via Intent in android.

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
