### git submodule 管理子工程

##### 摘要：当多人共同维护一个项目时，必然需要进行模块化开发，所以使用submodule来管理子工程很有必要。本文以图文并貌的形势进行一步步搭建主工程及绑定子工程。

##### 1、在Github上分别建立主工程HDMasterProject和两个子工程（动态／静态库）HDSubProjectOne、HDSubProjectTwo.

    SZ-denglibing:~ fangdd$ cd /Harry/Projects/HDMaster-SubProject/HDMasterProject
    SZ-denglibing:HDMasterProject fangdd$ git init
    Initialized empty Git repository in /Harry/Projects/HDMaster-SubProject/HDMasterProject/.git/
    SZ-denglibing:HDMasterProject fangdd$ git add .
    SZ-denglibing:HDMasterProject fangdd$ git commit -m '初始化工程'
    [master (root-commit) 10f33c6] 初始化工程
     12 files changed, 639 insertions(+)
     create mode 100644 HDMasterProject.xcodeproj/project.pbxproj
     create mode 100644 HDMasterProject.xcodeproj/project.xcworkspace/contents.xcworkspacedata
     create mode 100644 HDMasterProject.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/HDMasterProject.xcscheme
     create mode 100644 HDMasterProject.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/xcschememanagement.plist
     create mode 100644 HDMasterProject/AppDelegate.h
     create mode 100644 HDMasterProject/AppDelegate.m
     create mode 100644 HDMasterProject/Base.lproj/LaunchScreen.storyboard
     create mode 100644 HDMasterProject/Base.lproj/Main.storyboard
     create mode 100644 HDMasterProject/Info.plist
     create mode 100644 HDMasterProject/ViewController.h
     create mode 100644 HDMasterProject/ViewController.m
     create mode 100644 HDMasterProject/main.m
    SZ-denglibing:HDMasterProject fangdd$ git remote add origin https://github.com/erduoniba/HDMasterProject.git
    SZ-denglibing:HDMasterProject fangdd$ git push origin master
    Counting objects: 21, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (19/19), done.
    Writing objects: 100% (21/21), 7.45 KiB | 0 bytes/s, done.
    Total 21 (delta 2), reused 0 (delta 0)
    To https://github.com/erduoniba/HDMasterProject.git
     * [new branch]      master -> master
    
    SZ-denglibing:HDMasterProject fangdd$ cd ./HD
    HDMasterProject/           HDMasterProject.xcodeproj/
    SZ-denglibing:HDMasterProject fangdd$ cd ../
    HDMasterProject/ HDSubProjectOne/ HDSubProjectTwo/
    SZ-denglibing:HDMasterProject fangdd$ cd ../HDSubProjectOne/
    SZ-denglibing:HDSubProjectOne fangdd$ git init
    Initialized empty Git repository in /Harry/Projects/HDMaster-SubProject/HDSubProjectOne/.git/
    SZ-denglibing:HDSubProjectOne fangdd$ git add .
    SZ-denglibing:HDSubProjectOne fangdd$ git commit -m '初始化子工程1'
    [master (root-commit) 9c5c421] 初始化子工程1
     6 files changed, 438 insertions(+)
     create mode 100644 HDSubProjectOne.xcodeproj/project.pbxproj
     create mode 100644 HDSubProjectOne.xcodeproj/project.xcworkspace/contents.xcworkspacedata
     create mode 100644 HDSubProjectOne.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/HDSubProjectOne.xcscheme
     create mode 100644 HDSubProjectOne.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/xcschememanagement.plist
     create mode 100644 HDSubProjectOne/HDSubProjectOne.h
     create mode 100644 HDSubProjectOne/Info.plist
    SZ-denglibing:HDSubProjectOne fangdd$ git remote add origin https://github.com/erduoniba/HDSubProjectOne.git
    SZ-denglibing:HDSubProjectOne fangdd$ git push origin master
    Counting objects: 14, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (12/12), done.
    Writing objects: 100% (14/14), 4.56 KiB | 0 bytes/s, done.
    Total 14 (delta 0), reused 0 (delta 0)
    To https://github.com/erduoniba/HDSubProjectOne.git
     * [new branch]      master -> master
    
    SZ-denglibing:HDSubProjectOne fangdd$ cd ../HDSubProjectTwo/
    SZ-denglibing:HDSubProjectTwo fangdd$ git init
    Initialized empty Git repository in /Harry/Projects/HDMaster-SubProject/HDSubProjectTwo/.git/
    SZ-denglibing:HDSubProjectTwo fangdd$ git add .
    SZ-denglibing:HDSubProjectTwo fangdd$ git commit -m '初始化子工程2'
    [master (root-commit) 9be396f] 初始化子工程2
     6 files changed, 438 insertions(+)
     create mode 100644 HDSubProjectTwo.xcodeproj/project.pbxproj
     create mode 100644 HDSubProjectTwo.xcodeproj/project.xcworkspace/contents.xcworkspacedata
     create mode 100644 HDSubProjectTwo.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/HDSubProjectTwo.xcscheme
     create mode 100644 HDSubProjectTwo.xcodeproj/xcuserdata/fangdd.xcuserdatad/xcschemes/xcschememanagement.plist
     create mode 100644 HDSubProjectTwo/HDSubProjectTwo.h
     create mode 100644 HDSubProjectTwo/Info.plist
    SZ-denglibing:HDSubProjectTwo fangdd$ git remote add origin https://github.com/erduoniba/HDSubProjectTwo.git
    SZ-denglibing:HDSubProjectTwo fangdd$ git push origin master
    Counting objects: 14, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (12/12), done.
    Writing objects: 100% (14/14), 4.55 KiB | 0 bytes/s, done.
    Total 14 (delta 0), reused 0 (delta 0)
    To https://github.com/erduoniba/HDSubProjectTwo.git
     * [new branch]      master -> master
    SZ-denglibing:HDSubProjectTwo fangdd$



##### 2、分别在两个子工程添加代码进行测试：

```objective-c
@implementation HDSubProjectMethodOne
+ (void)hdSubProjectMethodOne{
    NSLog(@"hdSubProjectMethodOne");
}
@end

@implementation HDSubProjectMethodTwo
+ (void)hdSubProjectMethodTwo{
    NSLog(@"hdSubProjectMethodTwo");
}
@end
```

然后在各自的`HDSubProjectOne.h`和`HDSubProjectTwo.h`添加对测试代码头文件



##### 3、添加子工程到主工程中（单纯的添加，而不是子模块绑定）

~~**打开主工程，建立2个工程文件夹，分别将 `HDSubProjectOne.xcodeproj` 和 `HDSubProjectTwo.xcodeproj` 拖到文件夹中**~~，如下：

![](http://7xqhx8.com1.z0.glb.clouddn.com/1.png) 

在主工程的AppDelegate中调用2个子工程的代码：

![](http://7xqhx8.com1.z0.glb.clouddn.com/2.png)

嗯哼，找不到文件，看看`HDSubProject`下面，发现什么都没有，这个是因为直接拖动`.xcodeproj`只是引用，而没有将代码拷贝：

![](http://7xqhx8.com1.z0.glb.clouddn.com/25.png) 

![](http://7xqhx8.com1.z0.glb.clouddn.com/3.png) 

该问题解决：[杜甲同学的专栏  iOS 创建多个子工程的方法](http://www.2cto.com/kf/201503/385498.html) 

![](http://7xqhx8.com1.z0.glb.clouddn.com/4.png) 



字段说明：

`$(SRCROOT) `表示工程`.xcodeproj `所在的相对路径，比如我的电脑可能是`harry/project/HDMasterProject/` 也可能是`hhh/xcodeProject/HDMasterProject/`

` "$(SRCROOT)/当前工程名字/需要包含头文件所在文件夹" ` 将上面的双引号里面的字符串拷贝之后，你会发现这个`“$(SRCROOT)”`，会自动变成当前工程所以的目录。这样就可以了，发给别人，别人也不用在去修改路径了。



运行还是有问题，这个时候需要对子工程进行配置：

![](http://7xqhx8.com1.z0.glb.clouddn.com/5.png) 

然后将子工程的动态库加入到主工程中：

![](http://7xqhx8.com1.z0.glb.clouddn.com/6.png)

运行：

![](http://7xqhx8.com1.z0.glb.clouddn.com/7.png) 

搞定！



##### 4、添加子工程到主工程中（子模块绑定）

相关Git Submodule学习资料：[咖啡兔 Git Submodule使用完整教程](http://www.kafeitu.me/git/2012/03/27/git-submodule.html)  

使用 

```shell
git submodule add https://github.com/erduoniba/HDSubProjectOne.git 	
```

和 

```shell
git submodule add https://github.com/erduoniba/HDSubProjectTwo.git
```

将子工程绑定到主工程中。

这个时候你会发现在`HDMasterProject`会多出2个文件夹，这里的代码其实就是对`HDSubProjectOne`，`HDSubProjectTwo2`个子工程的clone，以后的代码修改就是在这里修改。

![](http://7xqhx8.com1.z0.glb.clouddn.com/8.png) 

打开`.gitmodules`

![](http://7xqhx8.com1.z0.glb.clouddn.com/9.png) 

可以看到` .gitmodules`记录了每个`submodule`的引用信息，知道在当前工程的位置以及仓库的所在。现在提交主及子工程工程的代码，通知你的小伙伴clone代码吧。



##### 5、他人拉取代码：

现在因为只有一台电脑，所以我将主工程clone到另外一个目录(Desktop)下来 模拟他人操作:

![](http://7xqhx8.com1.z0.glb.clouddn.com/10.png) 

发现我们的子工程并没有clone下来，莫急：

在HDMasterProject中clone 2个子工程

```shell
git clone https://github.com/erduoniba/HDSubProjectOne.git
git clone https://github.com/erduoniba/HDSubProjectTwo.git
```

![](http://7xqhx8.com1.z0.glb.clouddn.com/11.png) 

打开clone下来的主工程，会出现这样的问题：

![](http://7xqhx8.com1.z0.glb.clouddn.com/12.png) 

这个是因为在这里：

是因为第3步的 “~~打开主工程，建立2个工程文件夹，分别将 `HDSubProjectOne.xcodeproj` 和 `HDSubProjectTwo.xcodeproj` 拖到文件夹中~~” 有问题，原因是 **在建立工程时，我们是将下图 1 中的子模块的.xcodeproj拖入到主工程的，但是HDMasterProject并没有包含他们**，解决方式：**重新将2中的.xcodeproj拖入主工程，提交即可**，相应的Destop下的主工程更新代码就OK了。

![](http://7xqhx8.com1.z0.glb.clouddn.com/13.png) 

运行：

![](http://7xqhx8.com1.z0.glb.clouddn.com/14.png) 



##### 6、子工程提交代码，他人更新：

甲同学提交子工程代码：

```shell
SZ-denglibing:HDMasterProject fangdd$ cd HDSubProjectOne/
SZ-denglibing:HDSubProjectOne fangdd$ git add .
SZ-denglibing:HDSubProjectOne fangdd$ git commit -m '添加测试代码'
[master 6bdc6eb] 添加测试代码
 2 files changed, 6 insertions(+)
SZ-denglibing:HDSubProjectOne fangdd$ git push origin master
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 542 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
To https://github.com/erduoniba/HDSubProjectOne.git
   364abed..6bdc6eb  master -> master
```

i. 乙同学更新甲同学的子工程代码 **方式1 （到子工程直接更新）**

```shell
SZ-denglibing:HDMasterProject fangdd$ cd HDSubProjectOne/
SZ-denglibing:HDSubProjectOne fangdd$ git pull
remote: Counting objects: 5, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 5 (delta 3), reused 5 (delta 3), pack-reused 0
Unpacking objects: 100% (5/5), done.
From https://github.com/erduoniba/HDSubProjectOne
   364abed..6bdc6eb  master     -> origin/master
Updating 364abed..6bdc6eb
Fast-forward
 HDSubProjectOne/HDSubProjectMethodOne.h | 2 ++
 HDSubProjectOne/HDSubProjectMethodOne.m | 4 ++++
 2 files changed, 6 insertions(+)
```



ii.乙同学更新甲同学的子工程代码 **方式2 （到主工程使用git submodule update更新）**

```shell
cd 主工程路径
git submodule update --remote --merge
```

但是意外的是改方式一直失败，解决办法：[咖啡兔 Git Submodule使用完整教程](http://www.kafeitu.me/git/2012/03/27/git-submodule.html) 

```shell
SZ-denglibing:HDMasterProject fangdd$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
[remote "origin"]
    url = https://github.com/erduoniba/HDMasterProject.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

可以看到git的config中并没有任何关于子工程的信息,解决办法：

```shell
SZ-denglibing:HDMasterProject fangdd$ git submodule init
Submodule 'HDSubProjectOne' (https://github.com/erduoniba/HDSubProjectOne.git) registered for path 'HDSubProjectOne'
Submodule 'HDSubProjectTwo' (https://github.com/erduoniba/HDSubProjectTwo.git) registered for path 'HDSubProjectTwo'
SZ-denglibing:HDMasterProject fangdd$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
[remote "origin"]
    url = https://github.com/erduoniba/HDMasterProject.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
[submodule "HDSubProjectOne"]
    url = https://github.com/erduoniba/HDSubProjectOne.git
[submodule "HDSubProjectTwo"]
    url = https://github.com/erduoniba/HDSubProjectTwo.git
```

现在赶紧试试吧 (大功告成！！)



##### 7、git submodule 管理子工程相关命令：

在主工程更新子工程代码：

```shell
1、cd 主工程目录
2、 git submodule update --remote --merge
```

在主工程提交子工程的代码：

```shell
1、cd 子工程目录
2、git checkout master （将子目录checkout到master分支上）
3、
git pull
git add .
git commit
git push
```



##### 8、代码下载地址：

[HDMasterProject地址](https://github.com/erduoniba/HDMasterProject.git) 

[HDSubProjectOne地址](https://github.com/erduoniba/HDSubProjectOne.git) 

[HDSubProjectTwo地址](https://github.com/erduoniba/HDSubProjectTwo.git) 















