使用Jenkins实现自动化打包
===

[Jenkins](https://jenkins.io/)个开源的持续集成工具，不仅可以用来进行`Android`打包，也可以用来进行`iOS`打包、`NodeJs`打包、`Java`服务打包等。

> The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

`Jenkins`是使用`Java`开发的，官方提供一个`war`包，并且自带`servlet`容器，可以独立运行也可以放在`Tomcat`中运行。当然它也提供了`mac`等客户端，可以直接下载。 
因为一般我们都是将其部署到服务器上，所以这里就用下载`jenkins.war`放到`Tomcat`的方式来讲解。 


安装Tomcat
---

去[Tomcat](https://tomcat.apache.org/)官网下载最新的安装包，安装完成后启动`tomcat`.

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/tomcat_org.png" width="100%" height="100%">


安装启动都很简答，就是下载后解压，然后打开`terminal`进入解压后的目录下的`bin`目录，然后执行`startup`命令:

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/tomcat_start.png" width="100%" height="100%">

可以看到上面我执行`./startup.sh`时提示权限问题了，这是因为用户没有权限导致无法运行，需要要`chmod`修改`bin`目录下`.sh`的权限。
修改完权限后启动就可以了，看到提示启动成功后，我们可以在浏览器输入`http://localhost:8080`，如果能显示出来可爱的小喵咪，那就说明启动成功了。         

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/tomcat_startpage.png" width="100%" height="100%">

有关`tomcat`更多的信息就不介绍了，一般在`javaweb`的学习过程中都会学到。    

部署Jenkins到Tomcat
---

去[Jenkins官网](https://jenkins.io/)进行下载，然后选择意向版本的`.war`包

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_download_list.png" width="100%" height="100%">

把下载后的`war`包放在本地`tomcat`目录下的`webapps`目录下。   

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_tomcat.png" width="100%" height="100%">

然后在浏览器中访问`http://localhost:8080/jenkins/`，如果看到以下界面，代表已经成功部署了.  

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_startpage.png" width="100%" height="100%">

启动完成后会提示输入一个密码，上面有路径，我们直接进去打开拷贝就可以了。  

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_secret_code.png" width="100%" height="100%">

按照上面的路径，进入拷贝.

<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/get_secret_code.png" width="100%" height="100%">

然后会出现安装选择页面，我们选择默认的配置就可以。       
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_install.png" width="100%" height="100%">

然后就会出现以下界面，我们等待安装就可以了，这个安装过程会灰常慢。 我们也可以看到他会安装`ant、gradle、git、svn、email`等插件。  
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_installing.png" width="100%" height="100%">

等安装完成后会看到用户名设置界面。      
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_set_user.png" width="100%" height="100%">

好了，大功告成，开始使用后的页面如下:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_first_page.png" width="100%" height="100%">

那就开始创建一个项目的项目:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_demo.png" width="100%" height="100%">
 
创建后就会进入到项目设置页面:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_setting_general.png" width="100%" height="100%">

然后我们去设置源码管理，配置好分支和项目地址:     
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_source_manager.png" width="100%" height="100%">

下面点击证书后面的`add`进行添加，然后选择用用户名和密码的登录方式，输入用户名和密码:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_username_psw.png" width="100%" height="100%">

然后继续进行设置构建部分，因为`android`打包需要使用`Gradle`所以，我们选择使用`Gradle`然后进行配置`Gradle`:    
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_gradle_setting.png" width="100%" height="100%">

然后设置`Gradle`版本:        
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_gradle_setting2.png" width="100%" height="100%">

这里你会发现没有`Gradle`版本，这是因为我们没有去配置`Gradle`导致的，因为`android`打包需要`Gradle`和`JDK`所以我们要先去配置下他俩，
进入到`jenkins`首页选择系统管理-全局工具配置:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_gradle_install.png" width="100%" height="100%">

里面的`jdk`和`gradle`都可以选择在线安装，如果`jdk`使用在线安装的话需要输入`oracle`账号的的用户名和密码。
配置弯沉恭候，再回到刚才创建的`JenkisDemo`项目的配置页面继续进行构建配置:    
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_setting_3.png" width="100%" height="100%">

然后就可以选择我们刚才新添加的`gradle`版本了:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_gradle_version_choose.png" width="100%" height="100%">

然后再`Tasks`里面输入对应的`Task`命令就可以了。

在里面构建后操作中选择增加构建后操作步骤，可以选择构建完后自动发邮件等。      
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/build_over_operation.png" width="100%" height="100%">

到这里就配置完了，下面直接执行项目里面的立即构建就可以自动打包了。   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_build_start.png" width="100%" height="100%">

但是报错了，我们点本次构建列表中点入，再点击控制台输出可以查看详细的错误信息，提示说需要配置`SDK`,前面只配置了`jdk`忘了配置`sdk`了       
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_failed_msg.png" width="100%" height="100%">

打开系统管理-系统设置然后在全局变量-环境变量中增加`Android_home`配置:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_android_home.png" width="100%" height="100%">
 
好了，这样就可以了。 

但是我们再打包的时候不仅仅是想这样打一个简单的版本，而是想配置一下参数，来满足一些版本号，渠道等的需求。
我们可以来到项目的设置页面，选择参数化构建过程-添加参数-选择选项参数等进行添加:   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_demo_setting.png" width="100%" height="100%">

可以配置`APP_VERSION`、`BUILD_TIME`、`PRODUCT_FLAVORS`、`BUILD_TYPE`等  
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_build_product.png" width="100%" height="100%">


注意这里配置完成后，还要去在`gradle task`中去使用才可以(在命令后面加上配置的参数)：   
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_build_add_mingling.png" width="100%" height="100%">

好了，配置完成后再返回到项目首页你就会发现之前的立即构建变成了`Build with Parameters`，点击它就可以直接构建了。  
<img src="https://raw.githubusercontent.com/CharonChui/Pictures/master/jenkins_build_change_name.png" width="100%" height="100%">

打包完成后可以自动放到服务器目录中，通过`tomcat`提供链接对外下载，也可以发邮件、生成二维码等。这里就不仔细介绍了。
`jenkins`可以让我们更自由的进行配置，操作，也可以通过`shell`脚本等进行更加深度的定制，提高了开发中打包的效率。  

而且`Jenkins`还提供了很多静态代码分析检查的插件，可以直接去集成使用[Static Code Analysis Plug-ins](Static Code Analysis Plug-ins)



---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
