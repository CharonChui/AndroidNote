Vim使用教程
===

`Better, Stronger, Faster`

首先`Vim`有两种模式: 

- `Normal`            
    该模式下不能写入，修改要在该模式进行。在`Insert`模式中可以使用`ESC`键来返回到`Normal`模式。
- `Insert`      
    该模式下可以进行写入。在`Normal`模式下使用按`i`键进行`Insert`模式。
	
下面说一下`Normal`状态下的一些命令，所有的命令都要在`Normal`状态下执行：

- `i` 进入`insert`模式
- `x` 删除光标所在位置的一个字符
- `dd` 删除当前行，并把删除的行存到剪贴板中
- `p` 粘贴
- `yy` 拷贝当前行
- `a` 在光标后进行插入，直接进入`Insert`模式
- `o` 在当前行后插入一个新行，直接进入`Insert`模式
- `O` 在当前行钱插入一个新行，直接进入`Insert`模式
- `cw` 替换从光标位置开始到该单词结束位置的所有字符，直接进入`Insert`模式
- `0` 数字0光标移动当行头
- `u` 撤销、回退
- `Ctrl+r`  重新添加、前行

- `:wq` 保存并退出
- `:w` 存盘
- `:q!` 退出不保存
- `:saveas <pat>` 另存为


		
---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	