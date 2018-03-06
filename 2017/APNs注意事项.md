1、APNs推送的两个环境：

```
1、开发环境：（即在Debug模式下进行测试）
对应的证书名类似这样：Apple Development IOS Push Services：com.harry.push

2、线上环境：（即在Release模式下进行测试：要么打包之后测试，要么将Build Configuration设置为Release然后在Run）
对应的证书名类似这样：Apple Push Services：com.harry.push
```



2、DeviceToken：

```
deviceToken其实就是根据注册远程通知的时候向APNs服务器发送的Token key，Token key中包含了设备的UDID和App的Bundle Identifier，然后苹果APNs服务器根据此Token key编码生成一个deviceToken。deviceToken可以简单理解为就是包含了设备信息和应用信息的一串编码。推送消息就根据deviceToken（UDID + App's Bundle Identifier）找到对应的设备以及该设备上对应的应用，从而把此推送消息推送给此应用。

iOS9之后：
在不卸载App的情况下，Release和Debug切换时，deviceToken会更改；
卸载App的情况下，deviceToken会更改；

iOS9之前：
一个设备的token是唯一的。除了升级系统等少量情况，基本不变。 而且在token变了以后，老的token，就被认为是无效了。 苹果不会对这部分无效的token推送。
```



3、PushMeBaby：

[PushMeBaby](https://github.com/erduoniba/PushMeBaby)

支持开发和线上环境的测试



4、相关资料：

[iOS推送之远程推送（iOS Notification Of Remote Notification）](http://www.jianshu.com/p/4b947569a548) 

[iOS推送的那些事](http://www.jianshu.com/p/86328aa32505) 

[iOS10 推送通知详解(UserNotifications)](http://www.jianshu.com/p/567abab2098f)

[iOS 推送问题全解答《十万个为啥吖》](http://www.jianshu.com/p/ac248731baa8) 