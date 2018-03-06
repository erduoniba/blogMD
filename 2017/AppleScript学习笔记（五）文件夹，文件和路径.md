### AppleScript学习笔记（五）文件夹，文件和路径

1、在AppleScript中可以通过简单的两个单词来调出文件选择窗口

```vbscript
choose folder
```

然后选择一个文件夹（可以看到我们无法选择文件）：

![](http://img.blog.csdn.net/20140301001203046)

在点击选取后，结果输出为该文件夹的路径：

```vbscript
alias "Macintosh HD:Users:apple:Desktop:AFNetworkingDemo:" 
```

这里的冒号“:”相当于文件路径中的分割符号“/”。

这里的alias表示给出的是文件的ID，而不是文件本身的存储位置，这样使得该文件在被移动后脚本依然能够找到该文件的存储位置。

打开文件夹：

```vbscript
(*  
tell application "Finder"  
    open folder "Macintosh HD:Users:apple:Desktop:objc.io: #1 Light View Controllers:"  
end tell  
*)  
  
tell application "Finder"  
    open alias "Macintosh HD:Users:apple:Desktop:objc.io: #1 Light View Controllers:"  
end tell  
```





2、选择文件的方法：

```vbscript
choose file  
```

同样地，我们无法选取一个文件夹，只能选择一个文件。

打开文件：

```vbscript
(*  
tell application "Finder"  
    open file "Macintosh HD:Users:apple:Desktop:AFNetworkingDemo:podFile"  
end tell  
*)  
  
tell application "Finder"  
    open alias "Macintosh HD:Users:apple:Desktop:AFNetworkingDemo:podFile"  
end tell  
```

当然我们可以将choose file / folder和open file/folder/alias结合来使用。这样就不需要我们手动去填充文件和文件夹的路径了。例如：

```vbscript
set filePath to choose file  
tell application "Finder"  
    open file filePath  
end tell  
filePath  
```



3、将路径赋给变量：

可以将路径的值赋给变量，可以a reference to指令将其优化

```vbscript
set filePath to choose file
tell application "Finder"
	open file filePath
end tell

tell application "Finder"
	set filePath2 to a reference to file filePath
end tell

结果：
file (alias "Fast OS X:Users:denglibing:Desktop:embedded.mobileprovision") of application "Finder"
```



4、通过empty the trash指令来清空废纸篓，这里我们也可以将文件直接移动到废纸篓中：

```vbscript
tell application "Finder"
	move "Fast OS X:Users:denglibing:Desktop:embedded.mobileprovision" to the trash
end tell
```

使用move to指令可以让Finder程序将你的文件移动到其它位置。