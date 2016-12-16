Ant打包
===

使用步骤：    
1. 对于已经存在的工程需要利用`Ant`命令更新一下：    
    `android update project -n Test -p D:/workspace/Test -s -t 1`            
　　-n (name) 后面跟的是这个工程的名子      
　　-p (path)后面跟的是这个工程的目录路径                          
　　-t (target)后面是当前共有的`SDK`版本。表明我们的*目标版本*(如果有了`project.properties`就不用再跟`target`这个参数了).          
　　`android list target`这样就能够列出来所有的sdk版本          

2. 将签名文件keystore复制到工程根目录下,并且在根目录下新建`ant.properties`内容如下(配置签名文件):       
    ```
　　key.store=keystore.keystore //把签名放到根目录中   
　　key.alias=tencent
　　key.store.password=1234
　　key.alias.password=1234
    ```

3. 刷新工程    
    在`eclipse`中的`Ant`视图中右键`add build files`选择工程中的`build.xml`，选择最下面的`release`或者是`debug`，
	注意`release`是生成带签名的`apk`包.生成的apk在`bin`目录中，名字为工程名`-release.apk`.

4. 常见错误：      
	有时候在用`ant`打包的时候会报一些错误，一般按照错误的提示进行修改即可，如文件的非法字符等。     
	```java
	Error occurred during initialization of VM
	Could not reserve enough space for object heap
	Error: Could not create the Java Virtual Machine.
	Error: A fatal exception has occurred. Program will exit.
	```
	如果发现以上错误，就是说明栈内存不足了，一种是内存设置的太小，还有一种情况就是你设置的内存大小已经超过了当前系统限制的大小。
	打开`D:\Java\adt-bundle-windows\sdk\build-tools\android-4.4\dx.bat`将`set defaultXmx=-Xmx1024M`改为`set defaultXmx=-Xmx512M`即可。
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
