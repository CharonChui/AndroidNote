Git命令
===

先上一张图
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git.jpg)

- 安装好git后我们要先配置一下。以便`git`跟踪。
    ```
	git config --global "user.name xxx"
	git config --global "user.email xxx@xxx.com"
	```
	上面修改后可以使用`cat ~/.gitconfig`查看
	如果指向修改仓库中的用户名时可以不加`--global`，这样可以用`cat .git/config`来查看
	
	`git config --list`来查看所有的配置。

- 新建仓库
    `mkdir gitDemo`
    `cd gitDemo`
    `git init`
    这样就创建完了。

	```
- clone仓库
	在某一目录下执行.        
    `git clone [git path]`
	只是后`Git`会自动把当地仓库的`master`分支和远程仓库的`master`分支对应起来，远程仓库默认的名称是`origin`。

- `git add`提交文件更改(修改和新增),把当前的修改添加到暂存区     
    `git add xxx.txt`添加某一个文件         
    `git add .`添加当前目录所有的文件

- `git commit`提交，把修改由暂存区提交到仓库中
    `git commit`提交，然后在出来的提示框内查看当前提交的内容以及输入注释。     
    或者也可以用`git commit -m "xxx"` 提交到本地仓库并且注释是xxx

- `git status`查看当前仓库的状态和信息，会提示哪些内容做了改变已经当前所在的分支。

- `git diff`对比区别
    `git diff` 直接查看所有的区别
	`git diff HEAD -- xx.txt`查看工作区与版本库最新版的差别。   
	
- `git push` 提交到远程仓库
    可以直接调用`git push`推送到当前分支
	或者`git push origin master`推送到远程`master`分支
	`git push origin devBranch`推送到远程`devBranch`分支

- `git merge`合并目标分支到当前分支

- `git log`查看当前分支下的提交记录

- `git reset`命令回退到某一版本 
    `Git`中用`HEAD`表示当前版本，上一版本就是`HEAD^`,上上一版本就是`HEAD^^`.如果往前一千个版本呢？ 那就是`HEAD~1000`.     
    `git reset —-hard HEAD^`      
    `git reset —-hard commit_id`
	
	`git reset HEAD file`可以把用`git add`之后但是还没有`commit`之前暂存区中的修改撤销。

    说到这里就说一个问题，如果你reset到某一个版本之后，发现弄错了，还想返回去，这时候用`git log`已经找不到之前的`commit id`了。那怎么办？这时候可以使用下面的命令来找。
	
- `git reflog`
    可以查看所有操作记录包括`commit`和`reset`操作以及删除的`commit`记录
	
- `git checkout`撤销修改或者切换分支
    `git checkout --xx.txt`意思就是将`xx.txt`文件在工作区的修改全部撤销。可能会有两种情况:     
	- 修改后还没有调用`git add`添加到暂存区，现在撤销后就会和版本库一样的状态。
	- 修改后已经调用`git add`添加到暂存区后又做了修改，这时候撤销就会回到暂存区的状态。
	总的来说`git checkout`就是让这个文件回到最近一次`git commit`或者`git add`的状态。
	这里还有一个问题就是我胡乱修改了某个文件内容然后调用了`git add`添加到缓存区中，这时候想丢弃修改该怎么办？也是要分两步:
	- 使用`git reset HEAD file`命令，将暂存区中的内容回退，这样修改的内容会从暂存区回到工作区。
	- 使用`git checkout --file`直接丢弃工作区的修改。
	
	`git checkout`把当前目录所有修改的文件从`HEAD`都撤销修改。
	为什么分支的地方也是用`git checkout`这里撤销还是用它呢？他们的区别在于`--`，如果没有`--`那就是检出分支了。
	
	`git checkout origin/developer`  // 切换到orgin/developer分支

- `git rm`删除文件
    
- `git push`
    把本地仓库的内容推送到远程仓库中。
	
- 分支
    `git`分支的创建和合并都是非常快的，因为增加一个分支其实就是增加一个指针，合并其实就是让某个分支的指针指向某一个位置。
    ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_master_branch.png?raw=true)
	创建分支
	`git branch devBranch`创建名为`devBranch`的分支。
	`git checkout devBranch`切换到`devBranch`分支。
	`git branch`查看当前仓库中的分支
	`git branch -r`查看远程仓库的目录
	```
	origin/HEAD -> origin/master
	origin/developer
	origin/developer_sg
	origin/master
	origin/master_sg
	origin/offline
	```
	
	`git branch -d devBranch`删除`devBranch`分支。
	当时如果在新建了一个分支后进行修改但是还没有合并到其他分支的时候就去使用`git branch -d xxx`删除的时候系统会手提示说这个分支没有被合并，删除失败。
	这时如果你要强行删除的话可以使用命令`git branch -D xxx`.
	
- `git merge`合并指定分支到当前分支
   `git merge devBranch`将`devBranch`分支合并到`master`。
   
- 打`tag`
    `git tag v1.0`来进行打`tag`，默认为`HEAD`             
	`git tag`查看所有`tag`         
    如果我想在之前提交的某次`commit`上打`tag`，`git tag v1.0 commitID`    
	当然也可以在打`tag`时带上参数 `git tag v1.0 -m "version 1.0 released" commitID`
	
	`git show tagName`来查看某`tag`的详细信息。

- 打完`tag`后怎么推送到远程仓库
    `git push origin tagName`	
	
- 删除`tag`
    `git tag -d tagName`	
	
- 删除完`tag`后怎么推送到远程仓库，这个写法有点复杂
    `git push origin:refs/tags/tagName`	
	
- 忽略文件	
    在`git`根目录下创建一个特殊的`.gitignore`文件，把想要忽略的文件名填进去就可以了。
	其实不用一个个的去写，具体可以根据项目参考[https://github.com/github/gitignore](https://github.com/github/gitignore)
	当然不要忘了把该文件提交上去
	在用`linux`的时候会自动生成一些以`~`结尾的备份文件，如果ignore掉呢？[https://github.com/github/gitignore/blob/master/Global/Linux.gitignore](https://github.com/github/gitignore/blob/master/Global/Linux.gitignore)
	
	
	
关于`git`的工作区、缓存区可以看下图`index`标记部分的区域就是暂存区
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_stage.jpg?raw=true)	
从这个图中能看到缓存区的存在，这就是为什么我们新加或者修改之后都要调用`git add`方法后再调用`git commit`。
其实我一直有点分不开`reset`和`checkout`的区别，从这个图里能明显看出来了。 
- 当执行`git reset HEAD`命令时，暂存区的目录树会被重写，会被`master`分支指向的目录树所替换，但是工作区不受影响。
- 当执行`git checkout .`或`git checkout -- file`命令是，会用暂存区全部的文件或指定的文件替换工作区的文件。这个操作很危险，会清楚工作区中未添加到暂存区的改动。
- 当执行`git checkout HEAD .`或`git checkout HEAD` 命令时，会用`HEAD`指向的`master`分支中的全部或部分文件替换暂存区和工作区中的文件。这个命令也是极度危险的。因为不但会清楚工作区中未提交的改动，也会清楚暂存区中未提交的改动。

		
----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	