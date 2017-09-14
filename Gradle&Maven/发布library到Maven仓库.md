发布library到Maven仓库
---

在`Android Studio`中想要使用一些第三方类库的时候非常方便，只需要在`build.gradle`中加入一行代码就可以了：   
```java
dependencies {
    compile 'com.google.code.gson:gson:2.3.1'
}
```
刚从`Eclipse`转过来的时候感觉太方便了，也不用下`jar`包然后拷贝到`libs`目录了，重要的是以后升级起来灰常方便，改个数字就好了。
那究竟它里面是怎么运转的呢？其实`Android Studio`是从`Maven`仓库中下载所配置的`libraray`。     
总的来说，`Android`只有两种存放`library`的服务器就是`jCenter`和`Maven Central`:

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
- `Maven Central`         
    `Maven Central`是由[sonatype](https://issues.sonatype.org)维护的`Maven`仓库。想要在项目中使用`Maven Central`需要在项目的`build.gradle`文件中像下面这样进行声明:     

    ```java
    allprojects {
        repositories {
            mavenCentral()
        }
    }
    ```	
	虽然`jCenter`和`Maven Central`都是`Android`标准的`library maven`仓库，但是他俩是由不同提供者维护，并且存放到完全不同的服务器上，所以他俩之间毫无关系。     
	除了这两个标准的服务器外，如果我们使用的`library`作者把该`library`放到自己的服务器上，我们还可以自己定义特有的`Maven`仓库服务器。例如`Twitter`的`Fabric.io`就是这种情况，它们在[https://maven.fabric.io/public](https://maven.fabric.io/public)上维护了一个自己的`Maven`仓库，如果你想使用`Fabric.io`上的`library`你必须要自己定义仓库的`url`:       
	```java
	repositories {
		maven { url 'https://maven.fabric.io/public' }
	}
	然后才能正常使用
	dependencies {
		compile 'com.crashlytics.sdk.android:crashlytics:2.2.4@aar'
	}
	```	

	上传到自建服务器需要定义仓库的`url`，所以会比上传到标准的`maven`仓库使用起来要更麻烦。

	至于上传到`jCenter`还是`Maven Central`上面就要看开发者个人的爱好了，当然在这两个上面都上传是最方便的。在`Android Studio`开始的几个版本中，它将`Maven central` 作为默认仓库。新建项目之后`build.gradle`中会自动生成`Maven central`仓库的配置。
	但是`Maven Central`最大的问题就是上传`library`非常困难，同时还会由安全方面的原因，所以后来`Android Studio`将默认仓库替换成`jCenter`。所以最近的几个版本中创建项目之后,`build.gradle`中会默认定义`jCenter`而不是`Maven Central`。     

`jCenter`相对`Maven Central`来说更好的几个主要原因:    

- `jCenter`通过`CDN`发送`library`，开发者会由更快的下载体验。
- `jCenter`作为世界最大的`Java`仓库，所以它上面的`library`会更多。
- `jCenter`上传`library`更简单，不需要像在`Maven Central`上那么复杂。
- `jCenter`的用户界面更友好。
- `jCenter`的`bintray`网站上可以通过一键实现将`library`上传到`Maven Central`上。

所以我们后面要讲的就是如何上传`library`到`bintray`的`jCenter`中然后再一键上传到`Maven Central`上。    

说之前先通过`picasso`为例认识一下几个主要的部分:      
我们在使用`picasso`时，它会告诉我们按照如下使用： 
```java
compile 'com.squareup.picasso:picasso:2.5.2'
```
或者
```java
<dependency>
  <groupId>com.squareup.picasso</groupId>
  <artifactId>picasso</artifactId>
  <version>2.5.2</version>
</dependency>
```

这两种方法分别对应了`jCenter`和`Maven Central`中的使用方法.`jCenter`中`com.squareup.picasso`就是`grounId`，通常我们以开发者的报名后面紧跟`library`的名字来命名`groupId`, `picasso`是`artifactId`,`artifactId`也就是`library`真实的名称。,后面的`2.5.2`就是当前的`version`.

看到这里应该清楚其实就是`Android Studio`从`Maven`服务器上去下载我们所配置的`library`然后与我们项目一起进行编译。从`Maven`仓库上下载的是该`library`的`jar`或者`aar`文件。

仓库中存储的有两种文件类型的`library`：`jar`和`aar`。`jar`包大家都知道。那什么是`aar`呢？`aar`是在`jar`的基础上进行开发的，这是因为`Androd Library`需要植入一些特有的文件，例如资源文件、`assets`、`jni`、清单文件等。而这些都不是`jar`的标准，因为`aar`就是在`jar`包的基础上新增了对其他文件的封装。当然他和`jar`包一样，在本质上都是`zip`包，下面我们看一下`aar`的目录结构:     
```java
- /AndroidManifest.xml
- /classes.jar
- /res/
- /R.txt
- /assets/
- /libs/*.jar
- /jni/<abi>/*.so
- /proguard.txt
- /lint.jar
```

讲到这里已经清楚了`groupId`、`artifactId`以及`aar`了，那接下来我们就讲一下具体的发布流程了:          
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/public_jcenter.png?raw=true)   

在Bintray上创建一个包
---
               
- 登陆[bintray](https://bintray.com/)。
- 选择`Maven`后进入。            
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/bintray.png?raw=true)
- 添加新包。            
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/and_new_package.png?raw=true)
- 填写包信息            
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_package_information.png?raw=true)

    包名这里对于单词之间要用`-`分割。
- 然后会提示已经上传。            
 到这里我们已经在`bintray`上创建了一个`Maven`仓库，接下来要做的就是上传`library`。

在Sonatype上传件一个`Maven Central`账户。
---

- 注册Sonatype账号      
    在[Sonatype](https://issues.sonatype.org/secure/Dashboard.jspa)上注册账号。
    注册完登陆后需要在`JIRA`中创建一个`issue`，这样他就会允许你上传匹配`Maven Central`提供的`GROUP_ID`的`library`。    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/create_issue.png?raw=true)
    Project: Community Support - Open Source Project Repository Hosting              
    Issue Type: New Project          
    Summary: 你的 library名称的概要，比如The Cheese Library。           
    Group Id: 输入根GROUP_ID，比如， com.charonchui。一旦批准之后，每个以com.charon开始的library都允许被上传到仓库，比如com.charonchui.xxx。        
    Project URL: 输入任意一个你想贡献的library的URL，比如， [https://github.com/CharonChui/CyberLink4Android](https://github.com/CharonChui/CyberLink4Android)。           
    SCM URL: 版本控制的URL，比如 [https://github.com/CharonChui/CyberLink4Android.git](https://github.com/CharonChui/CyberLink4Android.git)。         
    其他的都不用管，写完之后创建就可以了。 然后就是开始等，大约一周左右就会获准将自己的`library`分享到`Maven Central`。
- 下面就是`Bintray`中的账户选项中填写`Sonatype`用户名。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_account.png?raw=true)

- 开启自动注册功能，为了能让我们通过`jcenter`上传的`libraray`自动发布到`Maven Central`中  
    - 使用下面的命令生成一个key。如果是windows用户需要使用cygwin。否则不能执行        
        `gpg --gen-key`
         后面会一步步的进行提示，按照提示输入相应内容就好。

         创建完成后可以使用`gpg --list-keys`命令来查看创建key的信息.
         接下来需要将生成的key上传到keyserver上才能发挥作用。
        ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/list_key.png?raw=true)
	
	    如上图所示将`key`上传到`keyserver`让它发挥作用，运行如下命令，将下面的PUBLIC_KEY_ID替换成上面2048R/后面的数字，如我的是B861C367
	    `gpg --keyserver hkp://pool.sks-keyservers.net --send-keys PUBLIC_KEY_ID`
		
    - 运行以下命令        
    ```java
	gpg -a --export yourmail@email.com > public_key_sender.asc
    gpg -a --export-secret-key yourmail@email.com > private_key_sender.asc
    ```
    运行完上面的两个命令后，会在当前目录分别生成public_key_sender.asc以及private_key_sender.ask两个文件，下面我们就打开这两个文件的内容，填写到Bintray的`Edit Profile`页面中的`GPG`注册部分。
    ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/GPG.png?raw=true)
    - 开启自动注册。          
        在`Edit Your Profile`页面找到`Repositories`后，选择`maven`后的`edit`。然后将GPG sign uploaded files using Bintray's public/private key pair.打上勾就可以了。
        ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/maven_edit.png?raw=true)
        ![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/maven_gpg.png?raw=true)

接下来就是如何将`Android Studio`中的`Library Module`进行上传
---

首先是修改项目的`build.gradle`文件。如下:    
```java
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // 注意gradle版本要再1.1.2以上，之前的版本存在问题。如果在工程中/gradle/wrapper/gradle-wrapper.properties重看到当前的gradle版本为2.4，下面的配置版本就要改为1.3.0
        classpath 'com.android.tools.build:gradle:1.2.3'
        // 下面的部分就为新添加的。
        // 用于上传项目到bintray
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        // 用于生成javaDoc和jar，如果gradle版本为2.4及以上，下面的版本就要改为1.3，具体可参考https://github.com/dcendents/android-maven-gradle-plugin
        classpath 'com.github.dcendents:android-maven-plugin:1.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

修改`local.properties`文件，在里面配置`apikey`、用户名以及密码，以便`bintray`进行验证。之所以要把这些东西配置到该文件中是因为这些信息都比较敏感，不能分享到别处，包括版本控制里面。而在创建项目的时候`local.properties`已经被添加到`.gitignore`中了，所以这些数据不会被上传：   
需要添加如下三行代码:     
```java
bintray.user=YOUR_BINTRAY_USERNAME  // 这里一定要用小写，不要用CharonChui不然会报错的，因为他会根据该用户名去找指定package，如果是大写他就找不到了。
bintray.apikey=YOUR_BINTRAY_API_KEY
bintray.gpg.password=YOUR_GPG_PASSWORD
```
`Api key`可以在`Binatry`官网上面的`Edit Profile`页面下的`API Key`中获取。      
最后要做的就是修改`library module`中的`build.gradle`文件。
下面这是最初的状态：       
```java
apply plugin: 'com.android.library'

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
下面就是修改之后的文件:      
```java
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: "com.jfrog.bintray"

// This is the library version used when deploying the artifact
version = "1.0.0"

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

def siteUrl = 'https://github.com/CharonChui/CyberLink4Android'      // Homepage URL of the library
def gitUrl = 'https://github.com/CharonChui/CyberLink4Android.git'   // Git repository URL
group = "com.charonchui.cyberlink"                      // Maven Group ID for the artifact

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'

                // Add your description here
                name 'cyberlink-android'
                description = 'CyberLink for Android is a development package for UPnP™ developers on Android development.'
                url siteUrl

                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'CharonChui'
                        name 'Charon Chui'
                        email 'charon.chui@gmail.com'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl

                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
artifacts {
    archives javadocJar
    archives sourcesJar
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

// https://github.com/bintray/gradle-bintray-plugin
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")

    configurations = ['archives']
    pkg {
        repo = "maven"
        // it is the name that appears in bintray when logged
        name = "cyberlink-android"
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
        version {
            gpg {
                sign = true //Determines whether to GPG sign the files. The default is false
                passphrase = properties.getProperty("bintray.gpg.password") //Optional. The passphrase for GPG signing'
            }
//            mavenCentralSync {
//                sync = true //Optional (true by default). Determines whether to sync the version to Maven Central.
//                user = properties.getProperty("bintray.oss.user") //OSS user token
//                password = properties.getProperty("bintray.oss.password") //OSS user password
//                close = '1' //Optional property. By default the staging repository is closed and artifacts are released to Maven Central. You can optionally turn this behaviour off (by puting 0 as value) and release the version manually.
//            }
        }
    }
}
```

到这一步就可以开始上传到`bintray`了，接下来开启`Android Studio`的`Terminal`中。
- 检查代码的正确性，以及编译`library`文件(`aar`,`pom`等)。执行如下命令:    
    `./gradlew install`如果提示权限不足就用`chmod 777 gradlew`改下权限就好了，windows下直接运行`gradlew install`
    正确的话会提示`BUILD SUCCESSFUL`
- 上传编译的文件到`bintray`，使用如下命令:     
    `./gradlew bintrayUpload`       
    成功的话会提示`SUCCESSFUL`      
接下来到`bintray`上检查一下你得`package`你会在版本区域发现新上传的版本。点击进去就能发现我们上传的`library`文件。      
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/publish_version.png?raw=true)

点击该版本进入后可以看到所有的文件目录。       
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/version_list.png?raw=true)

到这里你的`library`就已经上传了，任何人都可以使用，但是别高兴的太早，因为现在只是上传到了你自己的私人`Maven`仓库，而不是`jCenter`上，如果别人使用你`library`，他必须要先定义仓库的`url`， 如下:     
```java
repositories {
    maven {
        url 'https://dl.bintray.com/charonchui/maven/'
    }
}
...
  
dependencies {
    compile 'com.charonchui.cyberlink:cyberlink-android:1.0.0'
}
```
这样终究是不方便的，因为别人还要单独的去配置你仓库的`url`那么接下来就是怎么将`bintray`上我们的仓库上传到`jCenter`中呢？      
把`library`同步到`jCenter`上非常容易。只需要访问`bintray`，然后点击`Add to jCeter`然后在新出来的页面直接点击`Send`即可。
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/add_to_jcenter.png?raw=true)
现在我们就需要等待`bintray`团队审核我们的请求，一般会在两三个小时左右，如果审核通过，会收到一封邮件通知。通过这一步之后任何开发者都可以不用配置仓库`url`直接使用了。  
想检查一下自己的`library`是否在`jCenter`上存在，可以直接访问[http://jcenter.bintray.com](http://jcenter.bintray.com),然后根据你的`groupId`直接进入相应的目录查看即可。这里要说一下，这个链接到`jCenter`是一个只需要做一次的操作，如果以后对你的`package`做了修改，或者发布新版本等，这些改变会自动同步到`jCenter`上。同时，如果你要像删除某一个`package`，但是在`jCenter`仓库中的`library`不会被删除。它会一直存在，没法直接删除，因此如果你想要完全删除的时候，最好在移除`package`之前先在网页上把删除每个版本。
如何通过后也可以在`bintray`上面看到，这里已经不显示`Add to jCenter`按钮了，他会直接显示出来`jCenter`.            
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/already_add_to_jcenter.png?raw=true)

到这里你就可以在项目中直接使用:　　　　　
```java
dependencies {
    compile 'com.charonchui.cyberlink:cyberlink-android:1.0.0'
}
```
但是，我悲剧了。提示失败了，为什么呢？　
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/library_name_error.png?raw=true)　　　　　　　　　　　　　　　　　
原因就出来`library`这个`module name`这里，因为我这里的`module`那么是`library`所以它上传之后的路径就是`library`而不是`cyberlink-android`:     
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/libray_name_error_path.png?raw=true)　　     
所以通过这里能看出来它是使用当前`module`的名字的。我们只能通过如下方式去使用:     
```java
dependencies {
    compile 'com.charonchui.cyberlink:library:1.0.0'
}
```
这样就能正常使用了。 如果有些人感觉这样不好，就想要`com.charonchui.cyberlink:cyberlink-android:1.0.0`的方式，那就将`Android Studio`中`library`的名字修改为`cyberlink-android`后重新上传发布吧。


最后一步:上传library到Maven Central      
---

并不是所有的开发者都使用`jCenter`。仍然会有一些人在使用`Maven Central`。因为此我们仍然需要将`library`上传到`Maven Central`上。要从`jCenter`中上传到`Maven Central`之前需要先完成以下两部分:      

- `Bintray`上的`pacage`已经链接到`jCenter`.
- `Maven Central`上的仓库已经认证通过.

如果确认已经完成上面的授权后，上传`library`到`Maven Central`就非常简单了，只需要在`package`的详情页面点击`Maven Central`的链接。
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/publish_to_maven_central.png?raw=true)
直接进入下一个页面:         
![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/send_to_maven.png?raw=true)
然后再出来的页面输入`Maven Central`对应的用户名和密码点击`Sync`就可以了。上传到`Maven Central`的`library`是非常严格的，比如`+`号是不能在`library`版本的依赖定义中使用的。    
完成之后，你可以在 [Maven Central Repository](https://oss.sonatype.org/content/repositories/releases/)中找到你的`library`。

总结下在使用过程中遇到的错误: 

- Maven gradle插件与gradle版本要对应好，如下，不要不对应。
    ```xml
    // Top-level build file where you can add configuration options common to all sub-projects/modules.
    
    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:1.3.0'
            // 用于上传项目到bintray
            classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
            // 用于生成javaDoc和jar
            classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }
    
    allprojects {
        repositories {
            jcenter()
        }
    }
    ```
- `windows`下执行`gradlew install`编译`javadoc`时会提示`GBK`编码错误，这时候需要修改编译时的编码，具体放狗搜下，当时我改在`mac`上用了，就没解决。
- 编译生成`javadoc`时提示很多找不到类，不能使用`<br/>`等错误，这里建议在写注释的时候一定要规范，不要使用汉字，而且不要使用`<br/>`，`<p>`等。


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 