Zipalign优化
===

`Zipalign`优化工具是`SDK`中自带的优化工具,在`android-sdk-windows\build-tools\23.0.1`，在我们上传`Google Pay`的时候都会遇到您上传的`Apk`没有经过`Zipalign`处理
的失败提示，就是说如果你的`apk`没有使用`zipalign`优化，那`google play`是拒绝给你上架的，从这里能看出`zipalign`优化是多么滴重要。

```
zipalign is an archive alignment tool that provides important optimization to Android application (.apk) files. 
The purpose is to ensure that all uncompressed data starts with a particular alignment relative to the start of the file. 
Specifically, it causes all uncompressed data within the .apk, such as images or raw files, to be aligned on 4-byte boundaries. 
This allows all portions to be accessed directly with mmap() even if they contain binary data with alignment restrictions. 
The benefit is a reduction in the amount of RAM consumed when running the application.
```
                           
```
Caution: zipalign must only be performed after the .apk file has been signed with your private key. 
If you perform zipalign before signing, then the signing procedure will undo the alignment. 
Also, do not make alterations to the aligned package. 
Alterations to the archive, such as renaming or deleting entries, 
will potentially disrupt the alignment of the modified entry and all later entries. 
And any files added to an "aligned" archive will not be aligned.
```

大意就是它提供了一个灰常重要滴功能来确保所有未压缩的数据都从文件的开始位置以指定的4字节对齐方式排列，例如图片或者
`raw`文件。当然好处也是大大的，就是能够减少内存的资源消耗。最后他还特意提醒了你一下就是已经在对`apk`签完名之后再用`zipalign`
优化，如果你在之前用，那无效。

废多看用法:      

- 首先我要检查下我的`apk`到底用没用过`zipalign`优化呢?                                   
    `zipalign -c -v 4 test.apk`                      
    这个4是神马呢？就是４个字节的队列方式                                          
    命令一顿执行，然后打出来了`Verification failed`，我不想再解释了。
	
- 	如何使用?                                
    `zipalign -f -v 4 test.apk zip.apk`                             
	就是把当前的`test.apk`使用`zipalign`优化，优化完成后的是`zip.apk`
	
Flag:     

- -f : overwrite existing outfile.zip
- -v : verbose output
- -c : confirm the alignment of the given file


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
