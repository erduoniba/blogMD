### iOS10及Xcode8新特性分析



![](https://onevcat.com/assets/images/2016/ios-10-title.png) 

#### iOS10新特性：

[关于iOS10你需要理解的30个新特性](http://www.feng.com/Topic/iOS10_TOP30.shtml) 

##### 1、删除预装App：

![](http://resource.feng.com/resource/h061/h50/img201609211012500.jpg)

![](http://pic3.36krcnd.com/nil_class/d99177d0-d712-40b1-9c59-8297f39b46b5/ios10.png!heading) 

​	“删除预装 App”这项功能的到来让不少用户感到非常兴奋，一些从来不用又或者是很少使用的应用终于可以“删掉”了。

​	不过，所有这些能够删除的系统预装 App 都不是真正从 iPhone 中删除，因此，我们需要给它打个引号。事实上，我们将这项功能称为“隐藏预装 App”或者“移除预装 App”要更加恰当一些。

　　这些被删的应用可以在没有网络的情况下从 App Store 安装回来，所以更像是苹果暂时隐藏了这些应用。而且，苹果表示支持“删除”的系统应用加起来仅为 150MB，即便应用真的被删除干净，这对 16GB 用户来说似乎也没有多大帮助。

​	据苹果负责软件的VP Craig Federighi向知名苹果博主John Gruber[透露](http://daringfireball.net/2016/06/thoughts_and_observations_wwdc_2016)，这种卸载实际上并不是真的删除app。Federighi解释说删除app的动作的确可以将app从主屏幕上面移除，同时也会清理掉相关的用户数据，但是这些预装服务实际上是融入iOS里面的—出于安全签名的原因，这些app属于iOS二进制代码的一部分，所以尽管卸载了app，但是二进制代码还是存在的。正是因为这种结构，所以iOS升级的时候这些内置app只有功能更新而没有软件升级。作为此删除功能的配合，苹果在本周早些时候已经解绑了这些内置app，并作为独立app列在了在应用商店上面。“删除”这些预装app后用户还可以到应用商店下载安装，只不过重新下载安装这些app相当于把app跟hook和用户数据再重新挂钩，实际上并没有“下载”这个动作。



##### 2、信息应用：

![](http://resource.feng.com/resource/h061/h50/img201609211021370.png) 

​	随着 iOS 10 的到来，Messages 赫然将会变得一个小型的平台,因为 iOS 10 中将会新增 iMessage App ，而且还将融入各种第三方 App 。

###### i.   预加载网页：未点选网页连结时就先预览网页，避免误触恶意链接，此外我们还能直接预览影片。

###### ii.  Bubble强化：Messages 显示对话内容的泡泡框(Bubble)也会新增全新效果，它带来了放大、震动以及 Invisible Ink 的隐藏效果，让对话更添趣味。

###### iii. Digital Touch：可以给朋友发送涂鸦、轻敲、火球和心跳，这也为用户们带来了一种别样的表达方式与情趣。

###### iv. 新的平台：支持第三方应用 “进驻” Messages，比如通过 Messages 来呼叫 Uber、好友转帐、资源分享或者是使用开发者社区所能提供的一切功能和服务，而用户所做这的一切都无需切换到另外一个应用程序，只需在 Messages 上完成。



##### 3、锁屏特性：

![](http://resource.feng.com/resource/h061/h50/img201609211039540.jpg) 

###### i.   按下主屏幕按钮以解锁 :

在 iOS 10 系统中，iPhone 一直以来所采用的“滑动来解锁”已经被“按下主屏幕按钮以解锁”的长串文字取代。不支持 Touch ID 的 iPhone 用户需要按下 Home 键来输入密码。

###### ii.  通知界面的可操作性更高

苹果在 iOS 10 系统中对通知横幅进行了重新设计，用户不需要打开应用程序也可以和通知消息进行交互，比如回复或者删除等等。



##### 4、机型的兼容性：

![](http://resource.feng.com/resource/h061/h50/img201609211038150.png) 

**iPhone：**
　　iOS 10 系统支持的 iPhone 机型是 iPhone 5 及更新的机型，即 iPhone 5/5c/5s/SE/6/6 Plus/6s/6s Plus，当然还有最新亮相的 iPhone 7 系列机型。遗憾的是，iPhone 4s 并不在支持的名单中。
**iPad：**
　　平板方面，iPad Pro 的 12.9 英寸版本和 9.7 英寸版本、iPad Air、iPad Air 2、第四代 iPad 以及 iPad mini 2/3/4 这几款机型的用户可以更新至最新的 iOS 10 正式版本。
**iPod：**
　　支持 iOS 10 系统的 iPod 机型只有第六代 iPod touch 这一款机型。



##### 5、恢复应用时可优先下载：

![](http://resource.feng.com/resource/h061/h50/img201609210924480.jpg)

​	当你设置一部新的 iPhone 或 iPad 的时候，从 iCloud 备份进行恢复能够让你无需手里拥有一台电脑，不过痛苦的是整个应用的恢复过程可能会很漫长，特别是你刚好想用到某个应用的时候，系统可能最后才会帮你下载这个应用，这时候你会有一种全世界都与你为敌的感觉。

​	iOS 10 系统可以允许你优先下载某个应用。然而有一个前提，那就是你的设备需要支持 3D Touch，因为该功能只能通过 3D Touch 访问。



##### 更多：抬起唤醒、全能Siri、照片升级、全新地图应用、Home应用、电话骚扰



#### Xcode8 新特性：

[Xcode官方说明](https://developer.apple.com/xcode/) 

[Xcode 8新特性](http://www.skyfox.org/new-features-in-xcode8.html)

[Xcode8 及iOS10适配问题汇总](http://www.zhimengzhe.com/IOSkaifa/120848.html) 

[iOS开发之Xcode8兼容适配iOS 10资料整理笔记](http://allluckly.cn/%E6%8A%95%E7%A8%BF/tougao61) 

##### 1、运行时问题 Runtime Issues：(专门开篇说明)

![](http://www.skyfox.org/wp-content/uploads/2016/06/516F209E-6DF7-455F-BC69-939BC9BB5380.jpg) 

这Xcode新特性,自动识别跟踪找到漏洞并且报告问题, 有些很难跟踪的bug，直到您的应用程序到了用户手中,也可能没有被发现。

```shell
Thread Sanitizer spots:新的线程污点清理器, 解决多线程情况下的资源竞争条件,数据的变化和其它相关线程的bug
View Debugger:使用更新的带有更大的保真度和视觉精度检查UI约束问题的视图调试器
Memory Debugger:可以用新的内存调试跟踪器跟踪发出的内存泄漏警报。
```

##### 2、Interface Builder 界面构建器——加速：

![](http://www.skyfox.org/wp-content/uploads/2016/06/f6adcf48-723b-44ec-85f9-3d5c9087d97b.png) 

Interface Builder 设计画布已经彻底再造工程,让你更快地工作并且提供更大的控制。在任何充满活力的苹果设备上看到一个完全实时的应用程序预览。当为size classes定制UI,可以在不同的设备之间快速切换,你总会看到相同的界面。平移和缩放非常快,甚至你可以缩小故事板鸟瞰图时编辑你的界面。

##### 3、编辑器扩展 Editor Extensions：

新的Xcode源码编辑器扩展，让您自定义编码经验。使用扩展编辑器的 导航编辑的文本，选择、修改和改变你的代码。绑定快捷键到你最喜欢的扩展，使普通重复化任务易如反掌。Xcode中包括一个新的模板，以便您可以轻松创建编辑器的扩展并且在Mac App Store分发它们，或与登录您的开发者ID在线共享您的扩展。由于扩展在一个单独的进程中运行时，保持Xcode安全稳定。

```shell
旧金山Mono字体的新主题
快速自动生成帮助文档
高亮当前行
在Swfit代码中 图像和颜色文本
代码完成的图片
```

##### 4、签名变的简单而强大：

Provisioning Profile 文件选取，已经从Buiid Settings移动到了General中,Buiid Settings中已经标识了 Deprecated。

当你有个多个Mac的时候, Xcode会在每个Mac中自动生成对应的开发者证书。（疑虑）

##### 5、通知模块：

 iOS 10 中以前杂乱的和通知相关的 API 都被统一了，现在开发者可以使用独立的 UserNotifications.framework 来集中管理和使用 iOS 系统中通知的功能。在此基础上，Apple 还增加了撤回单条通知，更新已展示通知，中途修改通知内容，在通知中展示图片视频，自定义通知 UI 等一系列新功能，非常强大。

##### 6、问题汇总：

###### i. 代码注释失败：(然后重启电脑)

```shell
sudo /usr/libexec/xpcachectl
```

###### ii. 模拟器调试,控制台疯狂打印:

```objective-c
2016-09-25 22:20:13.364125 HDMasterProject[2651:335822] subsystem: com.apple.UIKit, category: HIDEventFiltered, enable_level: 0, persist_level: 0, default_ttl: 0, info_ttl: 0, debug_ttl: 0, generate_symptoms: 0, enable_oversize: 1, privacy_setting: 2, enable_private_data: 0
```

![](http://upload-images.jianshu.io/upload_images/1713611-1d35b400837f3eee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

加入 OS_ACTIVITY_MODE = disable 即可

###### iii. ATS的问题:

iOS 9中默认非HTTS的网络是被禁止的，当然我们也可以把NSAllowsArbitraryLoads设置为YES禁用ATS。不过iOS 10从2017年1月1日起苹果不允许我们通过这个方法跳过ATS，也就是说强制我们用HTTPS，如果不这样的话提交App可能会被拒绝。但是我们可以通过NSExceptionDomains来针对特定的域名开放HTTP可以容易通过审核。

细节提示:在iOS9以后的系统中如果使用到网络图片，也要注意网络图片是否是HTTP的哦，如果是，也要把图片的域设置哦！

###### iv. iOS 10 隐私权限设置:

iOS 10 开始对隐私权限更加严格，如果你不设置就会直接崩溃，现在很多遇到崩溃问题了，一般解决办法都是在info.plist文件添加对应的Key-Value就可以了。

###### ![](http://upload-images.jianshu.io/upload_images/1377514-db1c23310db7aa5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

```xml
<!-- 相册 --> 
<key>NSPhotoLibraryUsageDescription</key> 
<string>App需要您的同意,才能访问相册</string> 
<!-- 相机 --> 
<key>NSCameraUsageDescription</key> 
<string>App需要您的同意,才能访问相机</string> 
<!-- 麦克风 --> 
<key>NSMicrophoneUsageDescription</key> 
<string>App需要您的同意,才能访问麦克风</string> 
<!-- 位置 --> 
<key>NSLocationUsageDescription</key> 
<string>App需要您的同意,才能访问位置</string> 
<!-- 在使用期间访问位置 --> 
<key>NSLocationWhenInUseUsageDescription</key> 
<string>App需要您的同意,才能在使用期间访问位置</string> 
<!-- 始终访问位置 --> 
<key>NSLocationAlwaysUsageDescription</key> 
<string>App需要您的同意,才能始终访问位置</string> 
<!-- 日历 --> 
<key>NSCalendarsUsageDescription</key> 
<string>App需要您的同意,才能访问日历</string> 
<!-- 提醒事项 --> 
<key>NSRemindersUsageDescription</key> 
<string>App需要您的同意,才能访问提醒事项</string> 
<!-- 运动与健身 --> 
<key>NSMotionUsageDescription</key> <string>App需要您的同意,才能访问运动与健身</string> 
<!-- 健康更新 --> 
<key>NSHealthUpdateUsageDescription</key> 
<string>App需要您的同意,才能访问健康更新 </string> 
<!-- 健康分享 --> 
<key>NSHealthShareUsageDescription</key> 
<string>App需要您的同意,才能访问健康分享</string> 
<!-- 蓝牙 --> 
<key>NSBluetoothPeripheralUsageDescription</key> 
<string>App需要您的同意,才能访问蓝牙</string> 
<!-- 媒体资料库 --> 
<key>NSAppleMusicUsageDescription</key> 
<string>App需要您的同意,才能访问媒体资料库</string>
```



###### v. iOS 10 UIScrollView新增refreshControl:

iOS 10 以后只要是继承UIScrollView那么就支持刷新功能：

```objective-c
@property (nonatomic, strong, nullable) UIRefreshControl *refreshControl NS_AVAILABLE_IOS(10_0) __TVOS_PROHIBITED;
- (instancetype)init;

@property (nonatomic, readonly, getter=isRefreshing) BOOL refreshing;

@property (null_resettable, nonatomic, strong) UIColor *tintColor;
@property (nullable, nonatomic, strong) NSAttributedString *attributedTitle UI_APPEARANCE_SELECTOR;

// May be used to indicate to the refreshControl that an external event has initiated the refresh action
- (void)beginRefreshing NS_AVAILABLE_IOS(6_0);
// Must be explicitly called when the refreshing has completed
- (void)endRefreshing NS_AVAILABLE_IOS(6_0);
```

###### vi.  iOS 10开始项目中有的文字显示不全问题:

Xcode 7 创建一个Label然后让它自适应大小，字体大小都是17最后输出的宽度是不一样的，我们再看一下，下面的数据就知道为什么升级iOS 10 之后App中有的文字显示不全了：

```objective-c
Xcode 8打印   			Xcode 7.3打印
1个文字宽度：17.5 		1个文字宽度：17
2个文字宽度：35   		2个文字宽度：34
3个文字宽度：52   		3个文字宽度：51
4个文字宽度：69.5 		4个文字宽度：68
5个文字宽度：87   		5个文字宽度：85
6个文字宽度：104  		6个文字宽度：102
7个文字宽度：121.5   		7个文字宽度：119
8个文字宽度：139  		8个文字宽度：136
9个文字宽度：156  		9个文字宽度：153
10个文字宽度：173.5   	10个文字宽度：170 
```

英文字母会不会也有这种问题，通过测试，后来发现英文字母没有问题，只有汉字有问题。目前只有一个一个修改控件解决这个问题，暂时没有其他好办法来解决。