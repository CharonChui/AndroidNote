Git简介
=======

`Git`和其他版本控制系统(包括`Subversion`及其他相似的工具)的主要差别在于`Git`对待数据的方法。概念上来区分，其它大部分系统以文件
变更列表的方式存储信息。
这类系统(`CVS、Subversion、Perforce、Bazaar`等等)将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。
存储每个文件与初始版本的差异。

`Git`不按照以上方式对待或保存数据。反之，`Git`更像是把数据看作是对小型文件系统的一组快照。每次你提交更新，或在`Git`中保存项目
状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，`Git`不再重新存储该文件，而是只保留一个
链接指向之前存储的文件。`Git`对待数据更像是一个快照流。

`Git`是分布式版本控制系统，集中式和分布式版本控制有什么区别呢？

- 集中式版本控制系统
  版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，
  再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，
  再放回图书馆。集中式版本控制系统最大的毛病就是必须联网才能工作，如果在局域网内还好，带宽够大，速度够快，可如果在互联网上，
  遇到网速慢的话，可能提交一个10M的文件就需要5分钟，这还不得把人给憋死啊。
  ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_jizhong.jpeg)
- 分布式版本控制系统
  分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在
  你自己的电脑上。既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上
  改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。
  和集中式版本控制系统相比，分布式版本控制系统的安全性要高很多，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，
  随便从其他人那里复制一个就可以了。而集中式版本控制系统的中央服务器要是出了问题，所有人都没法干活了。
  在实际使用分布式版本控制系统的时候，其实很少在两人之间的电脑上推送版本库的修改，因为可能你们俩不在一个局域网内，两台电脑互相
  访问不了，也可能今天你的同事病了，他的电脑压根没有开机。因此，分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器
  的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。

  ![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_fenbu.jpeg)

版本库
------

什么是版本库呢？版本库又名仓库，英文名`repository`，你可以简单理解成一个目录，这个目录里面的所有文件都可以被`Git`管理起来，
每个文件的修改、删除，`Git`都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

所以，创建一个版本库非常简单:

- 创建一个空目录
- 通过`git init`命令把这个目录变成`Git`可以管理的仓库
    瞬间`Git`就把仓库建好了，而且告诉你是一个空的仓库`（empty Git repository）`，细心的读者可以发现当前目录下多了一个`.git`的目录，
    这个目录是`Git`来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把`Git`仓库给破坏了。
- 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
- 使用命令`git commit`，完成。

Git的五种状态
-------------

`Git`有五种状态，你的文件可能处于其中之一:

- 未修改`(origin)`
- 已修改`(modified)`
- 已暂存`(staged)`
- 已提交`(committed)`
- 已推送`(pushed)`

已提交表示数据已经安全的保存在本地数据库中。
已修改表示修改了文件，但还没保存到数据库中。
已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

`Git`仓库目录是`Git`用来保存项目的元数据和对象数据库的地方。这是`Git`中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。
工作目录是对项目的某个版本独立提取出来的内容。这些从`Git`仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在`Git`仓库目录中。 有时候也被称作‘索引’，不过一般说法还是叫暂存区域。

基本的`Git`工作流程如下:

- 在工作目录中修改文件。
- 暂存文件，将文件的快照放入暂存区域。
- 提交更新，找到暂存区域的文件，将快照永久性存储到`Git`仓库目录。

四个区
------

`Git`主要分为四个区:

- 工作区`(Working Area)`
- 暂存区`(Stage或Index Area)`
- 本地仓库`(Local Repository)`
- 远程仓库`(Remote Repository)`

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_buzhou.jpg)

正常情况下，我们的工作流程就是三个步骤，分别对应上图中的三个箭头线:

```shell
git add . // 把所有文件放入暂存区
git commit -m "comment"  // 把所有文件从暂存区提交进本地仓库
git push  // 把所有文件从本地仓库推送进远程仓库
```

先上一张图

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git.png)

图中的`index`部分就是暂存区

## 三棵树

Git作为一个系统，是以它的一般操作来管理并操纵这三棵树的：

| 树                | 用途                                 |
| :---------------- | :----------------------------------- |
| HEAD              | 上一次提交的快照，下一次提交的父结点 |
| Index             | 预期的下一次提交的快照               |
| Working Directory | 沙盒                                 |

#### HEAD

HEAD是当前分支引用的指针，它总是指向该分支上的最后一次提交。这表示HEAD将是下一次提交的父结点。通常，理解HEAD的最简方式，
就是将它看做**该分支上的最后一次提交**的快照。

#### 索引

索引是你的**预期的下一次提交**。我们也会将这个概念引用为Git的“暂存区”，这就是当你运行`git commit`时Git看起来的样子。

#### 工作目录

最后，你就有了自己的**工作目录**（通常也叫**工作区**）。 另外两棵树以一种高效但并不直观的方式，将它们的内容存储在`.git`文件夹中。
工作目录会将它们解包为实际的文件以便编辑。你可以把工作目录当做**沙盒**。在你将修改提交到暂存区并记录到历史之前，可以随意更改。

#### Git目录下文件的状态:

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_file_lifecycle.png?raw=true)
你工作目录下的每一个文件都不外乎这两种状态:

- 已跟踪(Tracked)    
  已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有他们的记录，在工作一段时间后，它们的状态可能是未修改，已修改或
  已放入暂存区。简而言之，已跟踪的文件就是Git已经知道的文件
- 未跟踪(Untracked)     
  工作目录中除已跟踪文件外的其它所有文件都属于未跟踪文件，它们即不存在与上次快照的记录中，也没有被放入暂存区。

## 常用命令

### git config

安装好git后我们要先配置一下。以便`git`跟踪。
```
    git config --global user.name "xxx"
    git config --global user.email "xxx@xxx.com"
```
上面修改后可以使用`cat ~/.gitconfig`查看
如果指向修改仓库中的用户名时可以不加`--global`，这样可以用`cat .git/config`来查看
`git config --list`来查看所有的配置。
如果需要查看当前的user.name和user.email的值可以通过`git config user.name`

### git init

新建仓库
``` 
mkdir gitDemo 
cd gitDemo 
git init
```
这样就创建完了。

### git clone仓库

在某一目录下执行.
`git clone [git path]`
执行后`Git`会自动把当地仓库的`master`分支和远程仓库的`master`分支对应起来，远程仓库默认的名称是`origin`。

### git add提交文件更改(修改和新增),把当前的修改添加到暂存区

`git add xxx.txt`添加某一个文件
`git add .`添加当前目录所有的文件

### `git commit`提交，把修改由暂存区提交到仓库中

`git commit`提交，然后在出来的提示框内查看当前提交的内容以及输入注释。
或者也可以用`git commit -m "xxx"` 提交到本地仓库并且注释是xxx

`git commit`是很小的一件事情，但是往往小的事情往往引不起大家的关注，不妨打开公司的任一个`repo`，查看`commit log`，
满篇的`update`和`fix`，完全不知道这些`commit`是要做啥。在提交`commit`的时候尽量保证这个`commit`只做一件事情，
比如实现某个功能或者修改了配置文件。注意是保证每个`commit`只做一件事，而不是让你做了一件事`commit`后就`push`，
那样就有点过分了。

### `git cherry-pick`

`git cherry-pick`可以选择某一个分支中的一个或几个`commit(s)`来进行操作。例如，假设我们有个稳定版本的分支，叫`v2.0`，
另外还有个开发版本的分支`v3.0`，我们不能直接把两个分支合并，这样会导致稳定版本混乱，但是又想增加一个`v3.0`中的功能到`v2.0`中，
这里就可以使用`cherry-pick`了。
就是对已经存在的`commit`进行 再次提交；
简单用法:
`git cherry-pick <commit id>`

`git rebase`命令基本是是一个自动化的`cherry-pick`命令。它计算出一系列的提交，然后再以它们在其他地方以同样的顺序一个一个的`cherry-picks`出它们。

### `git status`查看当前仓库的状态和信息，会提示哪些内容做了改变已经当前所在的分支。

### `git diff`

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_diff.webp?raw=true)
`git diff`直接查看当前修改未add(暂存staged)的差别
`git diff --staged`查看已add(到暂存区)的差别`git diff HEAD -- xx.txt`查看工作区与版本库最新版的差别。

- 首先如果我们只是本地修改了一个文件，但是还没有执行`git add .`之前，该如何查看有那些修改。这种情况下直接执行`git diff`就可以了。
- 那如果我们执行了`git add .`操作，然后你再执行`git diff`这时就会发现没有任何结果，这时因为`git diff`这个命令只是检查工作区和暂存区之间的差异。
    如果我们要查看暂存区和本地仓库之间的差异就需要加一个参数使用`--staged`参数或者`--cached`，`git diff --cached`。这样再执行就可以看到暂存区和本地仓库之间的差异。
- 现在如果我们把修改使用`git commit`从暂存区提交到本地仓库，再看一下差异。这时候再执行`git diff --cached`就会发现没有任何差异。
    如果我们行查看本地仓库和远程仓库的差异，就要换另一个参数，执行`git diff master origin/master`这样就可以看到差异了。 这里面`master`是本地的仓库，而`origin/master`是远程仓库，因为默认都是在主分支上工作，所以两边都是`master`而`origin`代表远程。

### `git push` 提交到远程仓库

可以直接调用`git push`推送到当前分支       
或者`git push origin master`推送到远程`master`分支       
`git push origin devBranch`推送到远程`devBranch`分支         
### `git log`查看当前分支下的提交记录

用`git log`可以查看提交历史，以便确定要回退到哪个版本。
如果已经使用`git log`查出版本`commit id`后`reset`到某一次提交后，又要重返回来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

```
git log -p -2 // -p 是仅显示最近的x次提交   
git log --stat // stat简略的显示每次提交的内容梗概，如哪些文件变更，多少删除，多少添加
git log --oneline --graph
git log --grep="1"
```
下面是常用的参数:

- `–-author=“Alex Kras”` ——只显示某个用户的提交任务
- `–-name-only` ——只显示变更文件的名称
- `–-oneline`——将提交信息压缩到一行显示
- `–-graph` ——显示所有提交的依赖树
- `–-reverse` ——按照逆序显示提交记录（最先提交的在最前面）
- `–-after` ——显示某个日期之后发生的提交
- `–-before` ——显示发生某个日期之前的提交
- `--grep` ——过滤内容

#### Git日志搜索

如果你想知道某一个东西是什么时候存在或者引入的。git log命令有许多强大的工具可以通过提交信息甚至是diff的内容来找到某个特定的提交。
例如，如果我们想找到ZLIB_BUF_MAX常量是什么时候引入的，我们可以使用-S选项来显示新增和删除该字符串的提交:

```shell
git log -S ZLIB_BUF_MAX --oneline
```
### `git reflog`

可以查看所有操作记录包括`commit`和`reset`操作以及删除的`commit`记录

### `git reset`

`git reset`命令用于将当前HEAD复位到指定状态。一般用于撤消之前的一些操作(如:`git add`,`git commit`等)。
在`git`的一般使用中，如果发现错误的将不想暂存的文件被`git add`进入索引之后，想回退取消，则可以使用命令:`git reset HEAD <file>`，
同时`git add`完毕之后，`git`也会做相应的提示，比如:

```shell
# Changes to be committed: 
#   (use "git reset HEAD <file>..." to unstage) 
# 
# new file:   test.py
```
`git reset [--hard|soft|mixed|merge|keep] [<commit>或HEAD]`:将当前的分支重设`(reset)`到指定的`<commit>`或者`HEAD`(默认，如果不显示指定`<commit>`，默认是`HEAD`，即最新的一次提交)，并且根据`[mode]`有可能更新索引和工作目录。`mode`的取值可以是`hard、soft、mixed、merged、keep`。下面来详细说明每种模式的意义和效果:

- `--hard`:彻底回退到某一个版本，本地的源码也会变为上一个版本的内容。重删除工作空间改动代码，撤销commit，撤销git add .。所有变更集都会被丢弃。
- `--mixed`:默认方式，它回退到某个版本，只保留源码，不删除工作空间改动代码，撤销commit，并且撤销git add . 。所有变更集都放在工作区。
- `--soft`: 回退到某个版本，不删除工作空间改动代码，撤销commit，不撤销git add . ，所有变更集都放在暂存区，如果还要提交直接重新commit即可。

#### 示例

假设我们进入到一个新目录，其中有一个文件。 我们称其为该文件的v1版本，将它标记为蓝色。 
现在运行git init，这会创建一个Git仓库，其中的HEAD引用指向未创建的master分支。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex1.png?raw=true)
此时，只有工作目录有内容。
现在我们想要提交这个文件，所以用git add来获取工作目录中的内容，并将其复制到索引中。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex2.png?raw=true)
接着运行git commit，它会取得索引中的内容并将它保存为一个永久的快照，然后创建一个指向该快照的提交对象，最后更新master来指向本次提交。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex3.png?raw=true)
此时如果我们运行 git status，会发现没有任何改动，因为现在三棵树完全相同。

现在我们想要对文件进行修改然后提交它。 我们将会经历同样的过程；首先在工作目录中修改文件。 我们称其为该文件的v2版本，并将它标记为红色。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex4.png?raw=true)
如果现在运行git status，我们会看到文件显示在 “Changes not staged for commit” 下面并被标记为红色，因为该条目在索引与工作目录之间存在不同。
接着我们运行git add来将它暂存到索引中。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex5.png?raw=true)
此时，由于索引和HEAD不同，若运行git status的话就会看到“Changes to be committed” 下的该文件变为绿色 ——也就是说，现在预期的下一次提交
与上一次提交不同。 最后，我们运行git commit来完成提交。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-ex6.png?raw=true)
现在运行git status会没有输出，因为三棵树又变得相同了。

切换分支或克隆的过程也类似。当检出一个分支时，它会修改HEAD指向新的分支引用，将索引填充为该次提交的快照，
然后将索引的内容复制到工作目录中。

#### 重置的作用

在以下情景中观察reset命令会更有意义。

为了演示这些例子，假设我们再次修改了file.txt 文件并第三次提交它。 现在的历史看起来是这样的：
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-start.png?raw=true)
让我们跟着reset看看它都做了什么。它以一种简单可预见的方式直接操纵这三棵树。它做了三个基本操作。

##### 第 1 步：移动 HEAD

reset 做的第一件事是移动 HEAD 的指向。 这与改变 HEAD 自身不同（checkout 所做的）；reset 移动 HEAD 指向的分支。 这意味着如果 HEAD 设置为 master 分支（例如，你正在 master 分支上）， 运行 git reset 9e5e6a4 将会使 master 指向 9e5e6a4。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-soft.png?raw=true)
无论你调用了何种形式的带有一个提交的 reset，它首先都会尝试这样做。 使用 reset --soft，它将仅仅停在那儿。

现在看一眼上图，理解一下发生的事情：它本质上是撤销了上一次 git commit 命令。 当你在运行 git commit 时，Git 会创建一个新的提交，并移动 HEAD 所指向的分支来使其指向该提交。 当你将它 reset 回 HEAD~（HEAD 的父结点）时，其实就是把该分支移动回原来的位置，而不会改变索引和工作目录。 现在你可以更新索引并再次运行 git commit 来完成 git commit --amend 所要做的事情了

##### 第 2 步：更新索引（--mixed）

注意，如果你现在运行 git status 的话，就会看到新的 HEAD 和以绿色标出的它和索引之间的区别。

接下来，reset 会用 HEAD 指向的当前快照的内容来更新索引。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-mixed.png?raw=true)
如果指定 --mixed 选项，reset 将会在这时停止。 这也是默认行为，所以如果没有指定任何选项（在本例中只是 git reset HEAD~），这就是命令将会停止的地方。

现在再看一眼上图，理解一下发生的事情：它依然会撤销一上次 提交，但还会 取消暂存 所有的东西。 于是，我们回滚到了所有 git add 和 git commit 的命令执行之前。

##### 第 3 步：更新工作目录（--hard）

reset 要做的的第三件事情就是让工作目录看起来像索引。 如果使用 --hard 选项，它将会继续这一步。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-hard.png?raw=true)
现在让我们回想一下刚才发生的事情。 你撤销了最后的提交、git add 和 git commit 命令 以及 工作目录中的所有工作。

必须注意，--hard 标记是 reset 命令唯一的危险用法，它也是 Git 会真正地销毁数据的仅有的几个操作之一。 其他任何形式的 reset 调用都可以轻松撤消，但是 --hard 选项不能，因为它强制覆盖了工作目录中的文件。 在这种特殊情况下，我们的 Git 数据库中的一个提交内还留有该文件的 v3 版本， 我们可以通过 reflog 来找回它。但是若该文件还未提交，Git 仍会覆盖它从而导致无法恢复。

回顾reset 命令会以特定的顺序重写这三棵树，在你指定以下选项时停止：

- 移动 HEAD 分支的指向 （若指定了 --soft，则到此停止）
- 使索引看起来像 HEAD （若未指定 --hard，则到此停止）
- 使工作目录看起来像索引

##### 通过路径来重置

前面讲述了 reset 基本形式的行为，不过你还可以给它提供一个作用路径。 若指定了一个路径，reset 将会跳过第 1 步，并且将它的作用范围限定为指定的文件或文件集合。 这样做自然有它的道理，因为 HEAD 只是一个指针，你无法让它同时指向两个提交中各自的一部分。 不过索引和工作目录 可以部分更新，所以重置会继续进行第 2、3 步。

现在，假如我们运行 git reset file.txt （这其实是 git reset --mixed HEAD file.txt 的简写形式，因为你既没有指定一个提交的 SHA-1 或分支，也没有指定 --soft 或 --hard），它会：

- 移动 HEAD 分支的指向 （已跳过）
- 让索引看起来像 HEAD （到此处停止）

所以它本质上只是将 file.txt 从 HEAD 复制到索引中。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-path1.png?raw=true)
它还有 取消暂存文件 的实际效果。 如果我们查看该命令的示意图，然后再想想 git add 所做的事，就会发现它们正好相反。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-path2.png?raw=true)
这就是为什么 git status 命令的输出会建议运行此命令来取消暂存一个文件。我们可以不让 Git 从 HEAD 拉取数据，而是通过具体指定一个提交来拉取该文件的对应版本。 我们只需运行类似于 git reset eb43bf file.txt 的命令即可。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-path3.png?raw=true)
它其实做了同样的事情，也就是把工作目录中的文件恢复到 v1 版本，运行 git add 添加它， 然后再将它恢复到 v3 版本（只是不用真的过一遍这些步骤）。 如果我们现在运行 git commit，它就会记录一条“将该文件恢复到 v1 版本”的更改， 尽管我们并未在工作目录中真正地再次拥有它。

还有一点同 git add 一样，就是 reset 命令也可以接受一个 --patch 选项来一块一块地取消暂存的内容。 这样你就可以根据选择来取消暂存或恢复内容了。

### `git checkout`撤销修改或者切换分支

`git checkout -- xx.txt`意思就是将`xx.txt`文件在工作区的修改全部撤销。可能会有两种情况:

- 修改后还没有调用`git add`添加到暂存区，现在撤销后就会和版本库一样的状态。
- 修改后已经调用`git add`添加到暂存区后又做了修改，这时候撤销就会回到暂存区的状态。

总的来说`git checkout`就是让这个文件回到最近一次`git commit`或者`git add`的状态。
这里还有一个问题就是我胡乱修改了某个文件内容然后调用了`git add`添加到缓存区中，这时候想丢弃修改该怎么办？也是要分两步:

- 使用`git reset HEAD file`命令，将暂存区中的内容回退，这样修改的内容会从暂存区回到工作区。
- 使用`git checkout --file`直接丢弃工作区的修改。

`git checkout`把当前目录所有修改的文件从`HEAD`都撤销修改。
为什么分支的地方也是用`git checkout`这里撤销还是用它呢？他们的区别在于`--`，如果没有`--`那就是检出分支了。
`git checkout origin/developer`  // 切换到orgin/developer分支

上面介绍了两个回退操作`git reset`和`git checkout`，这里就总结一下如何来对修改进行撤销操作:

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_reset_checkout.png?raw=true)

- 已经修改，但是并未执行`git add .`进行暂存
    如果只是修改了本地文件，但是还没有执行`git add .`这时候我们的修改还是在工作区，并未进入暂存区，我们可以使用:`git checkouot .`或者`git reset --hard`来进行撤销操作。

    `git add .`的反义词是`git checkout .`做完修改后，如果想要向前一步，让修改进入暂存区执行`git add .`如果想退后一步，撤销修改就执行`git checkout .`。
- 已暂存，未提交
    如果已经执行了`git add .`但是还没有执行`git commit -m "comment"`这时候你意识到了错误，想要撤销，可以执行:

  ```
  git reset   // git reset 只是把修改退回到了git add .之前的状态，也就是让文件还处于已修改未暂存的状态
  git checkout .   // 上面让文件处于已修改未暂存的状态，还要执行git checkout .来撤销工作区的状态
  ```
    或`git reset --hard`

    上面两个例子中都使用了`git reset --hard`这个命令也可以完成，这个命令可以一步到位的把你的修改完全恢复到本地仓库的未修改的状态。
- 已提交，未推送
    如果执行了`git add .`又执行了`git commit -m "comment"`提交了代码，这时候代码已经进入到了本地仓库，然而你发现问题了，想要撤销，怎么办？
    执行`git reset --hard origin/master`还是`git reset --hard`命令，只不过这次多了一个参数`origin/master`，这代表远程仓库，既然本地仓库已经有了
    你提交的脏代码，那么就从远程仓库中把代码恢复把。

    但是上面这样会导致你之前修改的代码都没有了，如果我只是想撤回提交，还想要我之前修改的东西重新回到本地仓库呢？
    `git reset --soft HEAD^`，这样就成功的撤销了你的commit。注意，仅仅是撤回commit操作，您写的代码仍然保留。
- 已推送到远程仓库
    如果你执行`git add .`后又`commit`又执行了`git push`操作了，这时候你的代码已经进入到了远程仓库中，如果你发现你提交的代码又问题想恢复的话，那你只能先把本地仓库的代码恢复，然后再强制执行`git push`仓做，`push`到远程仓库就可以了。

  ```
  git reset --hard HEAD^  // HEAD^代表最新提交的前一次  
  git push -f  // 强制推送
  ```

### reset checkout的区别

你大概还想知道 checkout 和 reset 之间的区别。 和 reset 一样，checkout 也操纵三棵树，不过它有一点不同，这取决于你是否传给该命令一个文件路径。

不带路径运行 git checkout [branch] 与运行 git reset --hard [branch] 非常相似，它会更新所有三棵树使其看起来像 [branch]，不过有两点重要的区别。

首先不同于 reset --hard，checkout 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢。 其实它还更聪明一些。它会在工作目录中先试着
简单合并一下，这样所有还未修改过的 文件都会被更新。而 reset --hard 则会不做检查就全面地替换所有东西。

第二个重要的区别是 checkout 如何更新 HEAD。 reset 会移动 HEAD 分支的指向，而 checkout 只会移动 HEAD 自身来指向另一个分支。

例如，假设我们有 master 和 develop 分支，它们分别指向不同的提交；我们现在在 develop 上（所以 HEAD 指向它）。
如果我们运行 git reset master，那么 develop 自身现在会和 master 指向同一个提交。 而如果我们运行 git checkout master 的话，
develop 不会移动，HEAD 自身会移动。 现在 HEAD 将会指向 master。

所以，虽然在这两种情况下我们都移动 HEAD 使其指向了提交 A，但做法是非常不同的。 reset 会移动 HEAD 分支的指向，而checkout则移动HEAD自身。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/reset-checkout.png?raw=true)

#### 带路径

运行 checkout 的另一种方式就是指定一个文件路径，这会像 reset 一样不会移动 HEAD。 它就像 git reset [branch] file 那样用该次提交中的那个文件来更新索引，但是它也会覆盖工作目录中对应的文件。 它就像是 git reset --hard [branch] file（如果 reset 允许你这样运行的话）， 这样对工作目录并不安全，它也不会移动 HEAD。

此外，同 git reset 和 git add 一样，checkout 也接受一个 --patch 选项，允许你根据选择一块一块地恢复文件内容。

### `git revert`撤销提交

`git revert`在撤销一个提交的同时会创建一个新的提交，这是一个安全的方法，因为它不会重写提交历史。

- `git revert`是生成一个新的提交来撤销某次提交，此次提交之前的`commit`都会被保留
- `git reset`是回到某次提交，提交及之前的`commit`都会被保留，但是此次之后的修改都会被退回到暂存区

相比`git reset`它不会改变现在得提交历史。`git reset`是直接删除指定的`commit`并把`HEAD`向后移动了一下。而`git revert`是一次新的特殊的`commit`，`HEAD`继续前进，本质和普通`add commit`一样，仅仅是`commit`内容很特殊。内容是与前面普通`commit`变化的反操作。
比如前面普通`commit`是增加一行`a`，那么`revert`内容就是删除一行`a`。
在 Git 开发中通常会控制主干分支的质量，但有时还是会把错误的代码合入到远程主干。虽然可以直接回滚远程分支，但有时新的代码也已经合入，
直接回滚后最近的提交都要重新操作。 那么有没有只移除某些Commit的方式呢？可以用一次revert操作来完成。

考虑这个例子，我们提交了 6 个版本，其中 3-4 包含了错误的代码需要被回滚掉。 同时希望不影响到后续的 5-6。

```shell
* 982d4f6 (HEAD -> master) version 6
* 54cc9dc version 5
* 551c408 version 4, harttle screwed it up again
* 7e345c9 version 3, harttle screwed it up
* f7742cd version 2
* 6c4db3f version 1
```
这种情况在团队协作的开发中会很常见：可能是流程或认为原因不小心合入了错误的代码，也可能是合入一段时间后才发现存在问题。
总之已经存在后续提交，使得直接回滚不太现实。

下面的部分就开始介绍具体操作了，同时我们假设远程分支是受保护的（不允许 Force Push）。思路是从产生一个新的 Commit 撤销之前的错误提交。

使用 git revert <commit> 可以撤销指定的提交， 要撤销一串提交可以用 <commit1>..<commit2> 语法。 注意这是一个前开后闭区间，
即不包括commit1，但包括commit2。

```shell
git revert --no-commit f7742cd..551c408
git commit -a -m 'This reverts commit 7e345c9 and 551c408'
```
其中 f7742cd 是 version 2，551c408 是 version 4，这样被移除的是 version 3 和 version 4。 
注意 revert 命令会对每个撤销的 commit 进行一次提交，--no-commit 后可以最后一起手动提交。

此时 Git 记录是这样的：

```shell
* 8fef80a (HEAD -> master) This reverts commit 7e345c9 and 551c408
* 982d4f6 version 6
* 54cc9dc version 5
* 551c408 version 4, harttle screwed it up again
* 7e345c9 version 3, harttle screwed it up
* f7742cd version 2
* 6c4db3f version 1
```
现在的 HEAD（8fef80a）就是我们想要的版本，把它 Push 到远程即可。

git revert 命令本质上就是一个逆向的 git cherry-pick 操作。 它将你提交中的变更的以完全相反的方式的应用到一个新创建的提交中，本质上就是撤销或者倒转。

### `git rm`删除文件

该文件就不再纳入版本管理了。如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f（译注：即 force 的首字母），以防误删除文件后丢失修改的内容。
另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。
比如一些大型日志文件或者一堆 .a 编译文件，不小心纳入仓库后，要移除跟踪但不删除文件，以便稍后在 .gitignore 文件中补上，
用 --cached 选项即可：`git rm --cached readme.txt`

### 分支

`git`分支的创建和合并都是非常快的，因为增加一个分支其实就是增加一个指针，合并其实就是让某个分支的指针指向某一个位置。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_master_branch.png?raw=true)

#### 创建分支

`git branch devBranch`创建名为`devBranch`的分支。
`git checkout devBranch`切换到`devBranch`分支。
`git checkout -b devBranch`创建+切换到分支`devBranch`。
`git branch`查看当前仓库中的分支。
`git branch -r`查看远程仓库的分支。
`git branch -d devBranch`删除`devBranch`分支。

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
如何删除远程分支呢？

```
git branch -r -d origin/developer
git push origin :developer
```
如何本地创建分支并推送给远程仓库？

```
// 本地创建分支
git checkout master //进入master分支
git checkout -b frommaster //以master为源创建分支frommaster
// 推送到远程仓库
git push origin frommaster// 推送到远程仓库所要使用的名字
```
如何切到到远程仓库分支进行开发呢？
`git checkout -b frommaster origin/frommaster`
// 本地新建frommaster分支并且与远程仓库的frommaster分支想关联
提交更改的话就用
`git push origin frommaster`

// 重命名分支
`git branch -m new_branch wchar_support`
// 查看每一个分支的最后一次提交
`git branch -v`

#### `git merge`合并指定分支到当前分支

`git merge devBranch`将`devBranch`分支合并到`master`。

### 打`tag`

`git tag v1.0`来进行打`tag`，默认为`HEAD`
`git tag`查看所有`tag`
如果我想在之前提交的某次`commit`上打`tag`，`git tag v1.0 commitID`
当然也可以在打`tag`时带上参数 `git tag v1.0 -m "version 1.0 released" commitID`
`git tag -d xxx`删除xxx

`git show tagName`来查看某`tag`的详细信息。

- 打完`tag`后怎么推送到远程仓库
    `git push origin tagName`
- 删除`tag`
    `git tag -d tagName`
- 删除完`tag`后怎么推送到远程仓库，这个写法有点复杂
    `git push origin:refs/tags/tagName`
- 忽略文件
    在`git`根目录下创建一个特殊的`.gitignore`文件，把想要忽略的文件名填进去就可以了,匹配模式最后跟斜杠(/)说明要忽略的是目录,#是注释 。

### amend修改最后一次提交

有时候我们提交完了才发现漏掉了几个文件没有加或者提交信息写错了，想要撤销刚才的的提交操作。可以修改后重新git add 然后使用`--amend`选项重新提交:`git commit --amend`，然后再执行`git push`操作。

修补后的提交可能需要修补提交信息，当你在修补一次提交时，可以同时修改提交信息和提交内容。 如果你修补了提交的内容，那么几乎肯定要更新提交消息以反映修改后的内容。

另一方面，如果你的修补是琐碎的（如修改了一个笔误或添加了一个忘记暂存的文件）， 那么之前的提交信息不必修改，你只需作出更改，暂存它们，然后通过以下命令避免不必要的编辑器环节即可：

$ git commit --amend --no-edit

#### 修改多个提交信息

为了修改在提交历史中较远的提交，必须使用更复杂的工具。 Git 没有一个改变历史工具，但是可以使用变基工具来变基一系列提交，基于它们原来的 HEAD 而不是将其移动到另一个新的上面。 通过交互式变基工具，可以在任何想要修改的提交后停止，然后修改信息、添加文件或做任何想做的事情。 可以通过给 git rebase 增加 -i 选项来交互式地运行变基。 必须指定想要重写多久远的历史，这可以通过告诉命令将要变基到的提交来做到。

例如，如果想要修改最近三次提交信息，或者那组提交中的任意一个提交信息， 将想要修改的最近一次提交的父提交作为参数传递给 git rebase -i 命令，即 HEAD~2^ 或 HEAD~3。 记住 ~3 可能比较容易，因为你正尝试修改最后三次提交；但是注意实际上指定了以前的四次提交，即想要修改提交的父提交：

$ git rebase -i HEAD~3
再次记住这是一个变基命令——在 HEAD~3..HEAD 范围内的每一个修改了提交信息的提交及其 所有后裔 都会被重写。
不要涉及任何已经推送到中央服务器的提交——这样做会产生一次变更的两个版本，因而使他人困惑。
输入

$ git commit --amend
修改提交信息，然后退出编辑器。 然后，运行

$ git rebase --continue
这个命令将会自动地应用另外两个提交，然后就完成了。 如果需要将不止一处的 pick 改为 edit，需要在每一个修改为 edit 的提交上重复这些步骤。 每一次，Git 将会停止，让你修正提交，然后继续直到完成。

### 查看远程仓库克隆地址

`git remote -v`

关于`git`的工作区、缓存区可以看下图`index`标记部分的区域就是暂存区
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/git_stage.jpg?raw=true)

从这个图中能看到缓存区的存在，这就是为什么我们新加或者修改之后都要调用`git add`方法后再调用`git commit`。

### stash(贮藏)

stash会处理工作目录的脏的文件--即跟踪文件的修改与暂存的改动--然后将未完成的修改保存到一个栈上，而你可以在任何时候重新应用这些改动（甚至在不同的分支上）。
假设你现在在`a`分支上开发新版本内容，已经开发了一部分，但是还没有达到可以提交的程度。你需要切换到`b`分支进行另一个升级的开发。那么可以把当前工作的改变隐藏起来，要将一个新的存根推到堆栈上，运行`git stash`命令。

```shell
$ git stash
Saved working directory and index state WIP on master: ef07ab5 synchronized with the remote repository
HEAD is now at ef07ab5 synchronized with the remote repository
```
现在，工作目录是干净的，所有更改都保存在堆栈中。 现在使用`git status`命令来查看当前工作区状态:

```shell
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working directory clean
```
现在，可以安全地切换分支并在其他地方工作。通过使用`git stash list`命令来查看已存在更改的列表。

```shell
$ git stash list
stash@{0}: WIP on master: ef07ab5 synchronized with the remote repository
```
这个命令所储藏的修改可以使用`git stash list`列出，使用`git stash show`进行检查，并使用`git stash apply`
或`git stash apply stash@{2}`恢复(可能在不同的提交之上)。或者可以用git stash pop将最近的一次stash恢复。调用没有任何参数
的`git stash`相当于`git stash save`。在17年10月下旬Git讨论废弃了git stash save命令，代之以现有的git stash push命令。
可以使用git stash drop加上要移除的贮藏的名字来移除它。

### 区间提交

你想要查看 experiment 分支中还有哪些提交尚未被合并入 master 分支。 你可以使用 master..experiment 来让 Git 显示这些提交。也就是“在 experiment 分支中而不在 master 分支中的提交”。

```shell
git log master..experiment
```
#### 三点

这个语法可以选择出被两个引用之一包含但又不被两者同时包含的提交。 再看看之前双点例子中的提交历史。 如果你想看 master 或者 experiment 中包含的但不是两者共有的提交，你可以执行：

```shell
git log master...experiment
```
### Rebase操作

官网中将rebase翻译为变基，我感觉理解成改变基点，重新实现更容易理解一些。
假设目前除master分支之外还有一个experiment分支:

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/basic-rebase-1.png?raw=true)
我们现在想要把master分支merge一下experiment分支的最新代码。整合分支最容易的方法是merge命令。它会把这两个分支的最新快照(C3和C4)以及两者
最近的共同祖先(C2)进行三方合并，合并的结果是生成一个新的快照(C5)并提交。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/basic-rebase-2.png?raw=true)
其实，还有一种方法：你可以提取在C4中引入的补丁和修改，然后在C3的基础上应用一次。在Git中，这种操作就叫做变基(rebase)。
你可以使用rebase命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。
在这个例子中，你可以检出experiment分支，然后将它变基到master分支上:

```shell
git checkout experiment
git rebase master
```
它的原理是首先找到这两个分支(即当前分支experiment、变基操作的目标基底分支master)的最近共同祖先C2，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底C3，最后以此将之前另存为临时文件的修改依序引用。
![将C4中的修改变基到C3上](https://raw.githubusercontent.com/CharonChui/Pictures/master/basic-rebase-3.png?raw=true)
现在回到master分支，进行一次快进合并。

```shell
git checkout master
git merge experiment
```
![master分支的快进合并](https://raw.githubusercontent.com/CharonChui/Pictures/master/basic-rebase-4.png?raw=true)
此时C4'指向的快照就和上面直接用merge中的C5指向的快照一模一样了。这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。
你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。
一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁--例如向某个他人维护的项目贡献代码时。在这种情况下，你首先在自己
的分支里进行开发，当开发完成时你需要先将你的代码变基到orgin/master上，然后再向主项目提交修改。这样的话，该项目的维护者就不再需要
进行整合工作，只需要快进合并即可。
请注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。变基是将一系列提交按照原有
次序依次应用到另一分支上，而合并是把最终结果合在一起。

##### 更有趣的变基例子

在对两个分支进行变基时，所生成的“重放”并不一定要在目标分支上应用，你也可以指定另外的一个分支进行应用。假如你创建了一个主题分支server，
为服务端添加了一些功能，提交了C3和C4.然后从C3上创建了主题分支client，为客户端添加了一些功能，提交了C8和C9.最后，你回到server分支，
又提交了C10.
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/interesting-rebase-1.png?raw=true)
假设你希望将client中的修改合并到主分支并发布，但暂时并不想合并server中的修改，因为他们还需要经过更全面的测试。这时，你就可以使用
git rebase命令的--onto选项，选中在client分支里但不在server分支里的修改(即C8和C9)，
将它们在master分支上重放: `git rebase --onto master server client`
以上命令的意思是：取出client分支，找出它从server分支分歧之后的补丁，然后把这些补丁在master分支上重放一遍，让client看起来像直接
基于master修改一样。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/interesting-rebase-2.png?raw=true)
现在可以快速合并master分支了(快速合并master分支，使之包含来自client分支的修改)：

```shell
git checkout master
git merge client
```
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/interesting-rebase-3.png?raw=true)
接下来你决定将server分支中的修改也整合进来，使用git rebase <basebranch> <topicbranch>命令可以直接将主题分支(即这里的server)变基到基分支(即这里的master)上。这样做能省去你先切换到server分支，再对其进行变基命令的多个步骤。
`git rebase master server`
如下图，将server中的修改变基到master上所示，server中的代码被“续”到了master后面。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/interesting-rebase-4.png?raw=true)
然后就可以快进合并主分支master了:

```shell
git checkout master
git merge server
```
至此，client和server分支中的修改都已经整合到主分支里了，你可以删除这两个分支，最终提交历史会变成下图的样子:
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/interesting-rebase-5.png?raw=true)

##### 变基的风险

奇妙的变基也并非完美无缺，要用它得遵守一条准则：
如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。
变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。如果你已经将提交推送至某个仓库，而其他人也已经从该
仓库拉取提交并进行了后续工作，此时，如果你用git rebase命令重新整理了提交并再次推送，你的同伴因此将不得不再次将他们手头的工作与你的
提交进行整合，如果接下来你还要拉去并整合他们修改过的提交，事情就会变的一团糟。
让我们来看一个在公开仓库上执行变基操作所带来的问题。假设你从一个中央服务器克隆然后在它的基础上进行了一些开发。你的提交历史如下图:
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/perils-of-rebasing-1.png?raw=true)
然后，某人又向中央服务器提交了一些修改，其中还包括一次合并。你抓取了这些在远程分支上的修改，并将其合并到你本地的开发分支，然后你的提交
记录就会变成这样:
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/perils-of-rebasing-2.png?raw=true)
接下来，这个人又决定把合并操作回滚，改用变基。继而又用git push --force命令覆盖了服务器上的提交历史。之后你从服务器抓取更新，会发现
多出来一些新的提交:
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/perils-of-rebasing-3.png?raw=true)
结果就是你们两个人的处境都十分尴尬。如果你执行git pull命令，你将合并来自两条提交历史的内容，生成一个新的合并提交，最终仓库也会变成:
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/perils-of-rebasing-4.png?raw=true)
这相当于是你将相同的内容又合并了一次，生成了一个新的提交。此时如果你执行git log命令，你会发现有两个调的作者、日期、日志居然是一样的，
这会令人感到混乱。此外，如果你将这一堆又推送到服务器上，你实际上是将那些已经被变基抛弃的提交又找了回来，这会令人感到更加混乱。
很明显对方并不想在提交历史中看到C4和C6，因为之前就是他把这两个提交丢弃的。

##### 用变基解决变基

如果你真的遭遇了类似的处境，Git 还有一些高级魔法可以帮到你。 如果团队中的某人强制推送并覆盖了一些你所基于的提交，你需要做的就是检查你做了哪些修改，以及他们覆盖了哪些修改。

实际上，Git 除了对整个提交计算 SHA-1 校验和以外，也对本次提交所引入的修改计算了校验和——即 “patch-id”。

如果你拉取被覆盖过的更新并将你手头的工作基于此进行变基的话，一般情况下 Git 都能成功分辨出哪些是你的修改，并把它们应用到新分支上。

举个例子，如果遇到前面提到的 有人推送了经过变基的提交，并丢弃了你的本地开发所基于的一些提交 那种情境，如果我们不是执行合并，而是执行 git rebase teamone/master, Git 将会：

- 检查哪些提交是我们的分支上独有的（C2，C3，C4，C6，C7）
- 检查其中哪些提交不是合并操作的结果（C2，C3，C4）
- 检查哪些提交在对方覆盖更新时并没有被纳入目标分支（只有 C2 和 C3，因为 C4 其实就是 C4'）
- 把查到的这些提交应用在 teamone/master 上面

从而我们将得到与 你将相同的内容又合并了一次，生成了一个新的提交 中不同的结果，如图 在一个被变基然后强制推送的分支上再次执行变基 所示。
![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/perils-of-rebasing-5.png?raw=true)
要想上述方案有效，还需要对方在变基时确保 C4' 和 C4 是几乎一样的。 否则变基操作将无法识别，并新建另一个类似 C4 的补丁（而这个补丁很可能无法整洁的整合入历史，因为补丁中的修改已经存在于某个地方了）。

在本例中另一种简单的方法是使用 git pull --rebase 命令而不是直接 git pull。 又或者你可以自己手动完成这个过程，先 git fetch，再 git rebase teamone/master。

如果你只对不会离开你电脑的提交执行变基，那就不会有事。 如果你对已经推送过的提交执行变基，但别人没有基于它的提交，那么也不会有事。 如果你对已经推送至共用仓库的提交上执行变基命令，并因此丢失了一些别人的开发所基于的提交，那你就有大麻烦了，你的同事也会因此鄙视你。

如果你或你的同事在某些情形下决意要这么做，请一定要通知每个人执行 git pull --rebase 命令，这样尽管不能避免伤痛，但能有所缓解。

#### 变基 vs. 合并

至此，你已在实战中学习了变基和合并的用法，你一定会想问，到底哪种方式更好。 在回答这个问题之前，让我们退后一步，想讨论一下提交历史
到底意味着什么。

有一种观点认为，仓库的提交历史即是记录实际发生过什么。它是针对历史的文档，本身就有价值，不能乱改。 从这个角度看来，改变提交历史是
一种亵渎，你使用谎言掩盖了实际发生过的事情。如果由合并产生的提交历史是一团糟怎么办？ 既然事实就是如此，那么这些痕迹就应该被保留下来，
让后人能够查阅。

另一种观点则正好相反，他们认为提交历史是项目过程中发生的事。没人会出版一本书的第一版草稿，软件维护手册也是需要反复修订才能方便使用。
持这一观点的人会使用 rebase 及 filter-branch 等工具来编写故事，怎么方便后来的读者就怎么写。

现在，让我们回到之前的问题上来，到底合并还是变基好？希望你能明白，这并没有一个简单的答案。 Git 是一个非常强大的工具，它允许你对
提交历史做许多事情，但每个团队、每个项目对此的需求并不相同。 既然你已经分别学习了两者的用法，相信你能够根据实际情况作出明智的选择。

总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。

### git fetch与git pull的区别

`git`中`fetch`命令是将远程分支的最新内容拉到了本地，但是`fecth`后是看不到变化的，如果查看当前的分支，会发现此时本地多了
一个`FETCH_HEAD`的指针，`checkout`到该指针后才可以查看远程分支的最新内容。

而`git pull`的作用相当于`fetch`和`merge`的组合，会自动合并:

```git
git fetch origin master
git merge FETCH_HEAD
```
### git pull 与git pull --rebase的使用

使用下面的关系区别这两个操作:

```git
git pull = git fetch + git merge
git pull --rebase = git fetch + git rebase
```
`git rebase`的过程中，有时会有`conflit`这时`Git`会停止`rebase`并让用户去解决冲突，解决完冲突后，
用`git add`命令去更新这些内容，然后不用执行`git commit`，直接执行`git rebase --continue`这样`git`会继续`apply`余下的补丁。

## 参考

[Git官方文档](https://git-scm.com/book/zh/v2)

---

- 邮箱 ：charon.chui@gmail.com
- Good Luck!
