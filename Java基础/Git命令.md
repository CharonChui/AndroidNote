Git命令
===

先上一张图
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git.jpg)

- 安装好git后我们要先配置一下。以便`git`跟踪。
    ```
	git config --global "user.name xxx"
	git config --global "user.email xxx@xxx.com"
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

- `git commit`提交
    `git commit -m "xxx"` 提交到本地仓库并且注释是xxx
	
- `git push` 提交到远程仓库

- `git merge`合并目标分支到当前分支

- `git log`查看当前分支下的某个提交记录




	
	
git reset --hard filename 撤销暂存，撤销工作区的修改。
		
----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	