Android开发不申请权限来使用对应功能
===

从用户角度来说很难获取到正确的`android`权限。通常你只需要做一些很基础的事(例如编辑一个联系人)但实际你申请的权限却远远比这更强大(例如可以获取到所有的联系人明细等)。

这样能很容易的理解到用户会怀疑到你的应用。如果你的应用不是开源的，那他们就没有方法来验证你会不会下载所有的联系人数据并上传到服务器。及时你去解释为什么需要这个权限，但是人们不会相信你。原来我会选择不去使用这些敏感的权限来防止用户产生不信任。(当然很多应用他们申请权限只是为了后台获取你的联系人数据上传- -!以及之前被爆的某宝使用摄像头拍照的问题)

这就是说，有一件事在困扰着我，**如何能在做一些操作时不去申请权限。**

打个比方说:`android.permission.CALL_PHONE`这个权限。你需要它来在应用中拨打电话，是吗？这就是你怎么去实现拨号的吗？ 
```java
Intent intent = new Intent(Intent.ACTION_CALL);
intent.setData(Uri.parse("tel:1234567890"))
startActivity(intent);
```
错！，你通过这段代码需要该权限的原因是因为你可以在任何时间在不需要用户操作的情况下打电话。也就是说如果我的应用申请了这个权限，我可以在你不知情的情况下每天凌晨三点去拨打骚扰电话。

正确的方式是使用`ACTION_VIEW`或者`ACTION_DIAL`:    
```java
Intent intent = new Intent(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:1234567890"))
startActivity(intent);
```

这个问题的完美解决方案就是不需要申请权限了。原因就是你不是直接拨号，而是用指定的号码调起拨号器，仍然需要用户点击”拨号”来开始打电话。老实的说，这样让人感觉更好。

简单的说就是如果我想要的操作不是让用户在应用内点击某个按钮就直接开始拨打电话，而是让用户点击在应用内点击某个按钮是我们去调起拨号程序，并且显示指定号码，让用户在拨号器中点击拨号后再开始拨打电话。这样的话我们就完全不用申请拨号权限了。

另一个例子: 我想获取某一个联系人的号码，你可能会想这需要申请获取所有联系人的权限。这是错的！。
```java
Intent intent = new Intent(Intent.ACTION_PICK);
intent.setType(StructuredPostal.CONTENT_TYPE);
startActivityForResult(intent, 1);
```
我们可以使用上面的代码，来启动联系人管理器，让用户来选择某一个联系人。这样不仅是不需要申请任何权限，也不需要提供任何联系人相关的`UI`。这样也能完全保证你选择联系人时的体验。


`Android`系统最酷的部分之一就是`Intent`系统，这意味着我不需要自己来实现所有的东西。应用可以注册处理它所擅长的指定数据，像电话号码、短信或者联系人。如果这些都要自己在一个应用中去实现，那这将会是很大的工作量，也会让应用变得臃肿。

`Android`系统的另一个优势就是你可以使用其他应用申请的权限，而不用自己申请。这样才保证了上面的情况。拨号器需要申请拨打电话的权限，我只需要一个能调起拨号器的`Intent`就好了。用户信任拨号器拨打电话，而不是我们的应用。他们无论如何都宁愿使用系统的拨号器。

写这篇文章的意义是**在你想要申请一个权限的时候，你需要至少看看[Intent的官方文档](https://developer.android.com/reference/android/content/Intent.html)看能否请求另外一个应用来帮我们做这些操作。**如果想要深入的研究，可以学习下[关于权限的详细介绍](https://developer.android.com/guide/topics/security/permissions.html)，这里面包含了很多精细的权限。

使用更少的权限可以不但可以让用户更加信任你，而且可以让用户有一个更好的体验，因为他们仍然在使用他们所期望的应用。

1. 遗憾的是，不是一个真实的号码。
2. 不幸的是，`Intent`系统的属性也建立了[可能会被滥用的漏洞](http://css.csail.mit.edu/6.858/2012/projects/ocderby-dennisw-kcasteel.pdf)，但你也不会写一个滥用的应用，是吗？


- (译)[感谢Dan Lew](http://blog.danlew.net/2014/11/26/i-dont-need-your-permission/)

		
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 