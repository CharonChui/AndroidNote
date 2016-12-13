Retrofit详解
===

之前写过一篇文章[volley-retrofit-okhttp之我们该如何选择网路框架](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/volley-retrofit-okhttp%E4%B9%8B%E6%88%91%E4%BB%AC%E8%AF%A5%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9%E7%BD%91%E8%B7%AF%E6%A1%86%E6%9E%B6.md)来分析`Volley`与`Retrofit`之间的区别。之前一直用`Volley`比较多。但是随着`Rx`系列的走红，目前越来越多的项目使用`RxJava+Retrofit`这一黄金组合。而且`Retrofit`使用注解的方式比较方便，今天简单的来学习记录下。

- 有关更多`Volley`的知识请查看[Volley源码分析](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/Volley%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)      
- 有关注解更多的知识请查看[注解使用](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/%E6%B3%A8%E8%A7%A3%E4%BD%BF%E7%94%A8.md)
- 有关更多[RxJava]的介绍请查看[Rx详解系列](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/RxJava%E8%AF%A6%E8%A7%A3(%E4%B8%8A).md)

简介
---

[Retrofit](http://square.github.io/retrofit/)

> A type-safe HTTP client for Android and Java




---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
