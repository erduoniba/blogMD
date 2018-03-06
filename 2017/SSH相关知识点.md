> SSH（Secure SHell）基于密钥的安全验证：需要依靠密钥，也就是需要为自己创建一对密钥，把公有的密钥放在需要访问的服务器上，客户端向服务器发送请求时，需要使用密钥进行安全验证：服务器收到请求之后，先在该服务器的用户根目录下需要你的公有密钥，然后把它和你发送过来的公有密钥进行对比，如果一致则服务器认为你这次请求有效并且响应你。从而避免被“中间人”攻击。

#### SSH如何生成？

1、设置git的username和email:

```shell
git config --global user.name "denglibing"
git config --global user.email "denglibing@fangdd.com"

# 查看当前git用户及email
git config user.name
git config user.email
```

2、查看并生成 `SSH` 密钥：

```shell
cd ~/.ssh 			#如果没有密钥则不会有次文件夹	

ssh-keygen -t rsa -C "denglibing@fangdd.com"
#连续按回车，密码默认为空
...
...
...
```

这样便会在 `~/.ssh` 下生成了对应的一对默认名称的密钥：`id_rsa` 和 `id_rsa.pub` 

3、生成多个 `SSH` 密钥，你可能需要多对密钥来区分公司项目和自己的项目，这个时候需要生成多个  `SSH` 密钥：

```shell
# 生成一个新的自定义名称的密钥
ssh-keygen -t rsa -C "13049862397@163.com" -f ~/.ssh/oschina_denglibing
#连续按回车，密码默认为空
...
...
...
```

执行完成后，会在  `~/.ssh` 下生成 `oschina_denglibing` 和 `oschina_denglibing.pub` 

4、设置 `SSH` 的用户配置，在 `~/.ssh` 下修改 config 文件（如果没有新建一个）：

```shell
# 配置密钥对应的服务器, 比如:
Host teamcode
Hostname teamcode.fangdd.net
User denglibing
Port 29418
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

Host oschina
Hostname git.oschina.net
User harrydeng
PreferredAuthentications publickey
IdentityFile ~/.ssh/oschina_denglibing
```

5、生成好了密钥之后，将公用密钥添加到git服务器上，可以参考 [oschina添加用户sshkey](http://git.mydoc.io/?t=154712)  

6、添加好之后，测试 `SSH` 配置文件是否正常工作：

```shell
ssh -T git@git.oschina.net
Welcome to Git@OSC, 邓立兵!
```



#### 一些问题

1、提交代码提示 `Permission denied (publickey)` 

这个可能是没有将公有密钥（publickey）添加到本地 `SSH` 造成的，或者多日没有进行 `SSH` 登录操作，本地公有密钥过期，使用

```shell
ssh-add ~/.ssh/oschina_denglibing
```



2、https方式提交代码，提示：

```shell
remote: Permission to erduoniba/LargeFileStorage.git denied to midea-smart.
fatal: unable to access 'https://github.com/erduoniba/LargeFileStorage.git/': The requested URL returned error: 403
```

解决步骤哦依次如下：打开Finder ----> 应用程序 ---->实用工具 ---->钥匙串访问 ---->双击，即可进入到钥匙串访问记录保存页面，选择github.com名称的应用，右键删除即可。

最后，再次push代码，会提示重新输入用户名及密码，输入github账号及密码即可。



#### 相关链接

[破男孩-生成多个git ssh密钥](http://www.cnblogs.com/ayseeing/p/4445194.html) 

[oschina-生成并部署sshkey](http://git.mydoc.io/?t=154712) 





