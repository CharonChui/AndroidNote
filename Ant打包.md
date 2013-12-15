Ant打包
=======================

使用步骤：    
1. 对于已经存在的工程需要利用`Ant`命令更新一下：   
```
    android update project -n Test -p D:/workspace/Test -s -t 1 
```
　　-n (name) 后面跟的是这个工程的名子  
　　-p (path)后面跟的是这个工程的目录路径                 
　　-t (target)后面是当前共有的`SDK`版本。表明我们的*目标版本*(如果有了`project.properties`就不用再跟target这个参数了).    
　　android list target这样就能够列出来所有的sdk版本
2. 将签名文件keystore复制到工程根目录下,并且在根目录下新建`ant.properties`内容如下(配置签名文件):
```
　　key.store=keystore.keystore //把签名放到根目录中   
　　key.alias=ifeng
　　key.store.password=ifeng123
　　key.alias.password=ifeng123
```

3. 刷新工程，然后在`eclipse`中的`Ant`视图中右键`add build files`选择工程中的`build.xml`，选择最下面的release或者是debug，注意release是生成带签名的apk包.生成的apk在bin目录中，名字为工程名-release.apk.

-----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 