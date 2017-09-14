Parcelable及Serializable
===

`Serializable`的作用是为了保存对象的属性到本地文件、数据库、网络流、`rmi`以方便数据传输，
当然这种传输可以是程序内的也可以是两个程序间的。而`Parcelable`的设计初衷是因为`Serializable`效率过慢，
为了在程序内不同组件间以及不同`Android`程序间(`AIDL`)高效的传输数据而设计，这些数据仅在内存中存在，`Parcelable`是通过`IBinder`通信的消息的载体。

`Parcelable`的性能比`Serializable`好，在内存开销方面较小，所以在内存间数据传输时推荐使用`Parcelable`，
如`activity`间传输数据，而`Serializable`可将数据持久化方便保存，所以在需要保存或网络传输数据时选择
`Serializable`，因为`android`不同版本`Parcelable`可能不同，所以不推荐使用`Parcelable`进行数据持久化。

区别:    
- Parcelable is faster than serializable interface
- Parcelable interface takes more time for implemetation compared to serializable interface
- serializable interface is easier to implement
- serializable interface create a lot of temporary objects and cause quite a bit of garbage collection
- Parcelable array can be pass via Intent in android.

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 