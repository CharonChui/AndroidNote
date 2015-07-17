发布library到jCenter
---

在`Android Studio`中想要使用一些第三方类库的时候非常方便，只需要在`build.gradle`中加入一行代码就可以了：   
```java
dependencies {
    compile 'com.google.code.gson:gson:2.3.1'
}
```
刚从`Eclipse`转过来的时候感觉太方便了，也不用下`jar`包然后拷贝到`libs`目录了，重要的是以后升级起来灰常方便，改个数字就好了。
那究竟它里面是怎么运转的呢？其实`Android Studio`是从`Maven`仓库中下载所配置的`libraray`。总的来说，`Android`只有两种存放`library`
的服务器就是`jCenter`和`Maven Central`。

- `jCenter`
    `jCenter`是由[bintray](https://bintray.com/)维护的`Maven`仓库。你可以在[这里](http://jcenter.bintray.com/)查看它所有的`library`。
	想要在项目中使用`jCenter`必须要在项目的`build.gradle`中像如下这样进行声明：    
	```java
	allprojects {
		repositories {
			jcenter()
		}
	}
	```
	
	

- 登陆[bintray](https://bintray.com/)。
- 选择`Maven`后进入。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/bintray.png?raw=true)
- 添加新包。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/and_new_package.png?raw=true)
- 填写包信息
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_package_information.png?raw=true)
- 然后会提示已经上传。

- 注册Sonatype账号     
    在[Sonatype](https://issues.sonatype.org/secure/Dashboard.jspa)上注册账号。
    注册完登陆后需要在`JIRA`中创建一个`issue`，这样他就会允许你上传匹配`Maven Central`提供的`GROUP_ID`的`library`。    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/create_issue.png?raw=true)

Project: Community Support - Open Source Project Repository Hosting

Issue Type: New Project

Summary: 你的 library名称的概要，比如The Cheese Library。

Group Id: 输入根GROUP_ID，比如，com.inthecheeselibrary 。一旦批准之后，每个以com.inthecheeselibrary开始的library都允许被上传到仓库，比如com.inthecheeselibrary.somelib。

Project URL: 输入任意一个你想贡献的library的URL，比如， https://github.com/nuuneoi/FBLikeAndroid。

SCM URL: 版本控制的URL，比如 https://github.com/nuuneoi/FBLikeAndroid.git。
其他的都不用管，写完之后创建就可以了。 然后就是开始等，大约一周左右就会获准将自己的`library`分享到`Maven Central`。
- 下面就是`Bintray`中的账户选项中填写`Sonatype`用户名。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_account.png?raw=true)

- 开启自动注册功能，伟了能让我们通过`jcenter`上传的`libraray`自动发布到`Maven Central`中，我们需要先注册后的上传机制.     
    - 使用下面的命令生成一个key。如果是windows用户需要使用cygwin。否则不能执行
        `gpg --gen-key`
         后面会一步步的进行提示，按照提示输入相应内容就好。
         ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_account.png?raw=true)

         创建完成后可以使用`gpg --list-keys`命令来查看创建key的信息.
         接下来需要将生成的key上传到keyserver上才能发挥作用。
        ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/list_key.png?raw=true)
    -  
    ```
	gpg -a --export yourmail@email.com > public_key_sender.asc
gpg -a --export-secret-key yourmail@email.com > private_key_sender.asc
    ```
    运行完上面的两个命令后，会在当前目录分别生成public_key_sender.asc以及private_key_sender.ask两个文件，下面我们就打开这两个文件的内容，填写到Bintray的`Edit Profile`页面中的`GPG`注册部分。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/GPG.png?raw=true)
    - 开启自动注册。
        在`Edit Your Profile`页面找到`Repositories`后，选择`maven`后的`edit`。然后将GPG sign uploaded files using Bintray's public/private key pair.打上勾就可以了。
        ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/maven_edit.png?raw=true)
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/maven_gpg.png?raw=true)


