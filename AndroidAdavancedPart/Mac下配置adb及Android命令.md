Mac下配置adb及Android命令
===

带着欣喜若狂的心情，买了一个`Mac`本，刚拿到它的时候感觉真的是一件艺术品，哈哈。废话不多说，里面配置`Android`开发环境。

1. 找到`android sdk`的本地路径，
    我的是：
    - `ADB`
        `/Android/adt-bundle-mac-x86_64-20131030/sdk/platform-tools`
    - `Android`
        `/Android/adt-bundle-mac-x86_64-20131030/sdk/tools`

2. 打开终端输入
    - `touch .bash_profile`创建
    - `open -e .bash_profile`打开

3. 添加路径
    `.bash_profile`打开了，我们在这里添加路径，
    - 如果打开的文档里面已经有内容,我们只要之后添加`:XXXX`**(注意前面一定要用冒号隔开)**       
    - 如果是一个空白文档的话，我们就输入一下内容
        `export PATH=${PATH}:XXXX`
    我的是     
    `export PATH=${PATH}:/Android/adt-bundle-mac-x86_64-20131030/sdk/tools:${PATH}:/Android/adt-bundle-mac-x86_64-20131030/sdk/platform-tools;`
    保存，关掉这个文档，
4. 终端输入命令  
    `source .bash_profile`
5. 大功告成

 
 
 ---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 