# Coil简介

Coil作为图片加载库的新秀，和Glide、Picasso这些老牌图片库相比，它们的优缺点是什么以及Coil未来的展望？先来了解一下什么是Coil。

Coil是基于Kotlin开发的首个图片加载库，来自Instacart团队，来看看官网对它的最新的介绍。

- R8:Coil下完全兼容R8，所以不需要添加任何与Coil相关的混淆器规则。
- Fast:Coil进行了许多优化，包括内存和磁盘缓存，对内存中的图片进行采样，重新使用位图，自动暂停/取消请求等等。
- Lightweight:Coil为你的APK增加了2000个方法(对于已经使用了OkHttp和协程的应用程序)，这与Picasso相当，明显少于Glide和Fresco。
- Easy to use:Coil利用了Kotlin的语言特性减少了样版代码。
- Modern:使用了大量的高级特性，例如协程、OkHttp、和androidX lifecycle跟踪生命周期状态的，Coil是目前唯一支持androidX lifecycle的库。

Coil作为图片库的新秀，越来越受欢迎了，但是为什么会引起这么多人的关注？在当今主流的图片加载库环境中Coil是一股清流，它是轻量级的，因为它使用了许多Android开发者已经在他们的项目中包含的其他库（协程、Okhttp）。

当我第一次看到这个库时，我认为这些都是很好的改进，但是我很想知道和其他主流的图片加载库相比，这个库能带来哪些好处，这篇文章的主要目的是分析一下Coil的性能，接下来我们来对比一下Coil、Glide和Picasso。

作者从以下场景对Coil、Glide、Picasso做了全面的测试。

- 当缓存为空时，从网络中下载图片的平均时间。

    - 从网络中下载图片所用的时间。

    结果:Glide最快Picasso和Coil几乎相同。

    - 加载完整的图片列表所用的时间，以及平均时间。

    结果:Glide是最快的，其次是Picasso，Coil是最慢的。

- 当缓存不为空时，从缓存中加载图片的平均时间。

    - 从缓存中加载图片所用的时间。

    结果:Glide最快，Coil其次，Picasso最慢。

    - 加载完整的图片列表所用的时间，以及平均时间。

    结果:Glide和Coil几乎相同，Picasso是最慢的。

图片加载库的选择是我们应用程序中最重要的部分之一，根据以上结果，如果你的应用程序中没有大量使用图片的时候，我认为使用Coil更好，原因有以下几点：

- 与Glide和Fresco类似，Coil支持位图池，位图池是一种重新使用不再使用的位图对象的技术，这可以显著提高内存性能(特别是在oreo之前的设备上)，但是它会造成一些API限制。
- Coil是基于Kotlin开发的，为Kotlin使用而设计的，所以代码通常更简洁更干净。
- Kotlin作为Android首选语言，Coil是为Kotlin而设计的，Coil在未来肯定会大方光彩。
- 从Glide、Picasso迁移到Coil是非常的容易，API非常的相似。
- Coil支持androidX lifecycle跟踪生命周期状态，也是是目前唯一支持androidX lifecycle的网络图片加载库。
- Coil支持动态图片采样，假设本地有一个500x500的图片，当从磁盘读取500x500的映像时，我们将使用100x100的映像作为占位符。

如果你的是图片类型的应用，应用程序中包含了大量的图片，图片加载的速度是整个应用的核心指标之一，那么现在还不适合使用Coil。

Coil涵盖了Glide、Picasso等等图片加载库所支持的功能，除此之外Coil还有一个功能动态图片采样。

### 动态图片采样

更多关于图片采样信息可以访问[Coil](https://coil-kt.github.io/coil/getting_started/) ，这里简单的说明一下，假设本地有一个500x500的图片，当从磁盘读取500x500的图片时，将使用100x100的映像作为占位符，等待加载完成之后才会完全显示。





# 参考: 

- [Coil vs Picasso vs Glide: Get Ready… Go!](https://proandroiddev.com/coil-vs-picasso-vs-glide-get-ready-go-774add8cfd40)
