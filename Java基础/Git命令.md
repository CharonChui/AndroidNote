Git命令
===

先上一张图
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git.jpg)

- 安装好git后我们要先配置一下。以便`git`跟踪。
    ```
	git config --global "user.name xxx"
	git config --global "user.email xxx@xxx.com"

- 新建仓库
    `mkdir gitDemo`
    `cd gitDemo`
    `git init`
    这样就创建完了。

	```
- clone仓库
	在某一目录下执行.        
    `git clone [git path]`
	但是很多时候我们`clone`的并不是主分支，而是一些其他的`branch`。接下来我们看一下如何`clone`一些开发的`branch`。
- 查看所有分支
    `cd`到刚检出的目录中，然后执行。       
    `git branch -r`执行后，能看到当前所有的分支: // git branch 是创建或者查看一个分支 -r就是查看远程分支
	```
	origin/HEAD -> origin/master
	origin/developer
	origin/developer_sg
	origin/master
	origin/master_sg
	origin/offline
	```
- 检出`developer`分支
    `git checkout origin/developer`  // checkout是切换到一个分支
    至此，就已经成功检出`developer`分支。	

- `git add`提交文件更改(修改和新增)	
    `git add xxx.txt`添加某一个文件         
    `git add .`添加当前目录所有的文件

- `git commit`提交
    `git commit -m "xxx"` 提交到本地仓库并且注释是xxx

- `git status`查看当前仓库的状态和信息，会提示哪些内容做了改变已经当前所在的分支。

- `git diff`对比区别
	
- `git push` 提交到远程仓库

- `git merge`合并目标分支到当前分支

- `git log`查看当前分支下的提交记录

- `git reset`命令回退到某一版本 
    `Git`中用`HEAD`表示当前版本，上一版本就是`HEAD^`,上上一版本就是`HEAD^^`.如果往前一千个版本呢？ 那就是`HEAD~1000`.     
    `git reset —-hard HEAD^`      
    `git reset —-hard commit_id`
    


	
	
git reset --hard filename 撤销暂存，撤销工作区的修改。
		
----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	