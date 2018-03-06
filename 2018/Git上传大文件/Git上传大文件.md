---
title: Git上传大文件
---



>最近开发项目中，使用了几个体积超过100M的第三方框架，这样导致在提交代码入库时，会被拒绝，以GitHub为例，单个文件超过50M入库会警告，超过100M会不然入库。



### 解决方案

#### <span id="1">1、将单个文件大于100M的文件不入库</span>

[GitHub官方解决方案](https://help.github.com/enterprise/11.10.340/user/articles/working-with-large-files/) 

```shell
git rm --cached giant_file
# Stage our giant file for removal, but leave it on disk

git commit --amend -CHEAD
# Amend the previous commit with your change
# Simply making a new commit won't work, as you need
# to remove the file from the unpushed history as well

git push
# Push our rewritten, smaller commit
```

如果上面的没有解决，也可以使用下面的命令

```shell
git filter-branch -f --prune-empty --index-filter 'git rm -rf --cached --ignore-unmatch FrameworkFold/XXXFramework/xxx' --tag-name-filter cat -- --all

git commit --amend -CHEAD

git push
```



这样导致虽然可以入库成功，但是本地已经删除了这个大文件，项目运行起来还需要重新将大文件加入到项目中才行。GitHub也推荐使用 [BGF](https://rtyley.github.io/bfg-repo-cleaner/) 。



#### **2、突破GitHub的限制，使用 [git-lfs(Git Large File Storage)](https://git-lfs.github.com) 支持单个文件超过100M**

![](./Git-lfs.gif) 

> LFS 并不能像"变魔术一样"处理所有的大型数据：它需要记录并保存每一个变化。然而，这就把负担转移给了远程服务器 - 允许本地仓库保持相对的精简。
>
> 为了实现这个可能，LFS 耍了一个小把戏：它在本地仓库中并不保留所有的文件版本，而是仅根据需要提供检出版本中必需的文件。
>
> 但这引发了一个有意思的问题：如果这些庞大的文件本身没有出现在你的本地仓库中....改用什么来代替呢? [LFS 保存轻量级指针](https://www.git-tower.com/learn/git/ebook/en/desktop-gui/advanced-topics/git-lfs?utm_source=gitlab-blog&utm_campaign=GitLab%20LFS&utm_medium=guest-post)中有真实的文件数据。当你用一个这样的指针去迁出一个修订版时，LFS 会很轻易地找到源文件（不在他上面可能就在服务器上，特殊缓存）然后你下载就行了。
>
> 因此，你最终只会得到你真正想要的文件 - 而不是一些你可能永远都不需要冗余数据。

```shell
# 1、安装git-lfs
brew install git-lfs

# 2、没有特别说明的情况下，LFS 不会处理大文件问题，因此，我们必须明确告诉 LFS 该处理哪些文件。将 FrameworkFold/XXXFramework/xxx的文件设置成大文件标示。
git lfs track "FrameworkFold/XXXFramework/xxx"

# 3、常规的push操作
git add .
git commit -m "add large file"
git push
```

**追踪文件路径（标示大文件）：**

1、追踪单个文件：

```shell
git lfs track "FrameworkFold/XXXFramework/xxx"
```

或者修改仓库路径下的 `.gitattributes` 文件：

```objective-c
FrameworkFold/XXXFramework/xxx filter=lfs diff=lfs merge=lfs -text
```

2、追踪指定类型的文件：

```shell
git lfs track "*.exe"
```

3、追踪指定目录下的文件：

```shell
git lfs track "FrameworkFold/*"
```



### 相关知识

[突破github的100M单个大文件上传限制](http://blog.csdn.net/tyro_java/article/details/53440666)

[Git LFS 入门指南](http://www.oschina.net/translate/getting-started-with-git-lfs-tutorial)

[Pro Git（中文版）](http://git.oschina.net/progit/)



### 一些问题

**1、Remote "origin" does not support the LFS locking API. Consider disabling it with**

```shell
# 在最后一步push的时候
git push -u origin develop1.0
Remote "origin" does not support the LFS locking API. Consider disabling it with:
  $ git config lfs.https://git.oschina.net/harrydeng/xxx.git/info/lfs.locksverify false
Git LFS: (0 of 1 files) 0 B / 207.25 MB                                                                                                    
batch request: Access denied
exec request failed on channel 0: exit status 255
error: failed to push some refs to 'git@git.oschina.net:harrydeng/xxx.git'
```

[解决方式：](https://www.cnblogs.com/fionacai/p/8456811.html) 

```shell
git config lfs.https://git.oschina.net/harrydeng/xxx.git/info/lfs.locksverify false
```



**2、batch request: Access denied**

```shell
# 在最后一步push的时候
git push -u origin develop1.0
Remote "origin" does not support the LFS locking API. Consider disabling it with:
  $ git config lfs.https://git.oschina.net/harrydeng/xxx.git/info/lfs.locksverify false
Git LFS: (0 of 1 files) 0 B / 207.25 MB                                                                                                    
batch request: Access denied
exec request failed on channel 0: exit status 255
error: failed to push some refs to 'git@git.oschina.net:harrydeng/xxx.git'
```

[解决方式：](https://github.com/git-lfs/git-lfs/issues/2291)

```shell
# 删除 .git/hooks/pre-push 文件即可
That looks like a server issue with deploy keys. For now, try removing .git/hooks/pre-push.
```



**3、GitHub 目前 Git LFS的总存储量为1G左右，超过需要付费。**(上传失败时，可以开启VPN进行上传)



**4、batch response: Repository or object not found**

```shell
$ git lfs push origin master
Git LFS: (0 of 1 files) 0 B / 207.25 MB                                                                                                    
batch response: Repository or object not found: https://gitee.com/harrydeng/LargeFileStorage.git/info/lfs/objects/batch
Check that it exists and that you have proper access to it
```

失败原因：

```
是gitee.com这个git仓库并不支持lfs，所以在大文件入库的时候，提示失败
```

[解决方式](#1)

目前来说，GitHub、GitLab、Coding。gitee(也就是git.oschina.net)目前还不支持。











