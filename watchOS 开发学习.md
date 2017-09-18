### watchOS 开发学习

>  **1、设计概要：** 

[watchOS Human Interface Guidelines](https://developer.apple.com/watch/human-interface-guidelines/) 

当你设计你的 `Watch app` 时 , 你需要了解 `Apple Watch` 本身的基础设计:

![](https://developer.apple.com/watch/human-interface-guidelines/images/overview-lightweight.png)   

**轻量级的交互**  ( `Apple Watch` 需要快速的交互设计来感知用户的手腕动作，信息需要快速而简单的访问和释放，最好的应用需要支持快速的交互和展示用户最关系的内容 )；



 ![](https://developer.apple.com/watch/human-interface-guidelines/images/overview-holistic.png)  

**整体的设计** （苹果将设备的软件本身的界限变得模糊， `Force Touch`  和 `Digital Crown` 让使用者无缝的在界面上进行交互，你的 `app` 应该加强用户的感知从而让硬件和软件整体化）；



![](https://developer.apple.com/watch/human-interface-guidelines/images/overview-personal.png) 

**人性化通讯** （因为 `Apple Watch` 是穿戴而设计的，所以它的 `UI` 是为了适用用户而存在，没有其他的 `Apple` 设备如此和用户亲密接触，所以在设计中需要密切关注这点）



>  **2、开发**

[App Programming Guide for watchOS](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969-CH8-SW1) 

**概要：**

通过 `Apple Watch` ，用户只需要轻轻的抬手就可以快速的访问自己想要的信息，也可以接受处理通知、通过 [glance](#glance) 或者 `complication`  来看到一些基本信息或者更多。开发 `Apple Watch` 意味着方便有效的给用户提供重要的、有用的、准确的信息。

创建 `Apple Watch` 项目由两个独立的资源包组成：`Watch app` 和 `WatchKit extension` 。`Watch app`  包含了 `storyboards` 和访问该项目的所以资源文件； `WatchKit extension`  包含了 `extension delegate ` 和 管理响应界面交互的 `controllers` 。这些包分布在一个 `iOS app` 中 , 然后安装在用户的 `Apple Watch` 并在 `Apple Watch` 中运行。

开发 `Apple Watch` 项目必须包含一个 `Watch app` ，也可能包含可选界面模块，比如  [glance](#glance) 、 [notifications](#notifications) 以及 [complications](#complications) ，尽管这些是可选的或者是次要的,但是却是很多用户的主要界面。下面将介绍这些可选界面模块：



##### glance

[Glance Essentials](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/ImplementingaGlance.html#//apple_ref/doc/uid/TP40014969-CH5-SW1)

glance这个名字意味着快速预览，在滑动glance时下面也有相应的page展示，glance需要展示的是你项目的最重要的信息。glance必须和  `Apple Watch` 的屏幕尺寸一致，不支持内部滑动，不包含按钮、开关或者其他交互的控件。点击glance便会进入你的app主界面。

##### notifications

[Notification Essentials](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/BasicSupport.html#//apple_ref/doc/uid/TP40014969-CH18-SW1)

[Registering Your Actionable Notification Types](https://developer.apple.com/library/watchos/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/IPhoneOSClientImp.html#//apple_ref/doc/uid/TP40008194-CH103-SW26)

[Local and Remote Notification Programming Guide](https://developer.apple.com/library/watchos/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/Introduction.html#//apple_ref/doc/uid/TP40008194) 

 `Apple Watch` 与其配对 `iPhone` 显示本地和远程通知。通知一来,  `Apple Watch` 使用最小界面显示的通知（short look）。如果用户的手腕抬起, 最小的界面变化成更详细的界面来显示通知的内容（long look）。你可以添加自定义图形或通知获取的数据来构建long look界面来展示通知。

 `Apple Watch` 还提供了支持可操作的通知。可操作的通知让你通知添加按钮或文本输入界面,以便用户可以直接回应通知。例如,一个会议邀请通知可能包括按钮接受或拒绝邀请。当你的iOS应用程序注册支持可操作的通知, `Apple Watch` 会在 `notifications` 对应的界面上展示你添加的按钮，`WatchKit extension`  中处理这些点击事件。

##### complications

[Complication Essentials](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/ComplicationEssentials.html#//apple_ref/doc/uid/TP40014969-CH27-SW1)

`complications`（有点类似插件） 是通过一些小的视觉控件来给用户展示重要的信息。`complications` 会在用户通过  `Apple Watch` 看时间的时候自动展示出来的。让使用者可以把特定的应用、功能（比如说航班信息、电动车的充电状态等）提到最前面显示。新的床头模式在启用时（需充电），数码表冠会化身为一颗「小睡」按钮。而且它还有另外一个作用，是在查看行事历的时候，轻轻转一下就能找到接下来要做的事了。



**2.1 项目结构：**

![](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/Art/target_structure_2x.png)

上图展示了项目的结构， `iOS app` 包含了 `Watch app` ， 而 `Watch app` 包含了 `WatchKit extension` , 在用户在 `iPhone` 上安装  `iOS app` 时，系统会自动在其匹配的 `Apple Watch ` 上安装  `Watch app` (包含  `WatchKit extension` )

`Watch app` 必须是作为 `iOS app` 的一部分而进行发布，你可以在已有的 `iOS app` 添加 `Watch app` ，也可以创建新的  `iOS app` 来包含 `Watch app` 。调试过程中也很方便，运行 `Watch app` ，再在 `iPhone` 模拟器上点开对应的 `iOS app` 即可调试测试。



**2.2 项目生命周期：**

![](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/Art/launch_cycle_2x.png) 

上图（项目启动过程）中的 [init](https://developer.apple.com/library/watchos/documentation/WatchKit/Reference/WKInterfaceController_class/index.html#//apple_ref/occ/instm/WKInterfaceController/init) 和 [awakeWithContext:](https://developer.apple.com/library/watchos/documentation/WatchKit/Reference/WKInterfaceController_class/index.html#//apple_ref/occ/instm/WKInterfaceController/awakeWithContext:) 方法可以用来请求数据，设置界面的值，为界面的展示做准备。不要在 [willActivate](https://developer.apple.com/library/watchos/documentation/WatchKit/Reference/WKInterfaceController_class/index.html#//apple_ref/occ/instm/WKInterfaceController/willActivate) 做这些事，但是可以把它作为最后来处理将要展示界面的方法。



![](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/Art/watch_app_lifecycle_simple_2x.png) 

上图（controller的生命周期）中， `WatchKit extension` 只是在用户通过 `Apple Watch` 和你的app交互的时候运行。因为和  `Apple Watch` 的交互是短暂的，所以界面需要轻量级而不是一个长期的任务执行。当用户退出或者停止和  `Apple Watch`  交互之后，iOS 就和当前界面失去连接最终暂停 `WatchKit extension` 。



![](https://developer.apple.com/library/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/Art/wkextension_states_2x.png)  `Watch app` 的生命周期如上











