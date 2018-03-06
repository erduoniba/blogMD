### appium1-macOS10.12下如何丝滑的使用appium?

1、下载或者更新Homebrew：[homebrew官网](http://brew.sh/index_zh-cn.html) macOS 不可或缺的套件管理器

```shell
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"  #下载
$ brew update #更新
$ brew -v
Homebrew 1.1.5
Homebrew/homebrew-core (git revision e151b; last commit 2016-12-26)
```



2、使用homebrew来安装node.js : [Node.js教程](http://www.runoob.com/nodejs/nodejs-tutorial.html) node采用C++编写，是一个JavaScript的运行环境，因为JS是脚本语言需要一个解析器才能运行，对于写在HTML页面的JS，浏览器本身就是解析器，对于需要单独运行的JS，Node.js就是解析器。

```shell
$ brew install node
$ node -v
v7.3.0
```



3、安装npm： [NPM 使用介绍](http://www.runoob.com/nodejs/nodejs-npm.html) 是一个NodeJS包管理和分发工具，已经成为了非官方的发布Node模块（包）的标准。

```shell
$ brew install npm
```



4、使用npm来安装appium，2017/01/03目前最新的版本是 `1.6.3` ，当然也可以在 [Macos下bitbucket下载appium客户端](https://bitbucket.org/appium/appium.app/downloads/)，但是坑人的是最新版本是 `1.5.3` 支持的还只是 `iOS10` 以下的 `UIAutomation` 框架基础上测试，而 `iOS10` 以后苹果使用的却是基于 `XCUITest` ，所以只能采用npm安装咯。但是因为国内墙太高而使用npm却是从国外服务器下载，有时候使用npm需要非常长的时间，更多时候是失败，推荐使用cnpm来安装：[淘宝 NPM 镜像](https://npm.taobao.org) 10分钟同步npmjs.org镜像，使用方式和npm完全一致

```shell
$ npm install cnpm -g --registry=https://registry.npm.taobao.org
$ npm view appium versions #查看appium所有的版本号
```



5、使用cnpm安装appium和appium-doctor：

```shell
$ cnpm install -g appium --no-cache
$ cnpm install -g appium
$ cnpm install -g appium-doctor
```



6、检查appium环境检测：

```shell
$ appium-doctor
info AppiumDoctor Appium Doctor v.1.2.5
info AppiumDoctor ### Diagnostic starting ###
info AppiumDoctor  ✔ Xcode is installed at: /Applications/Xcode.app/Contents/Developer
WARN AppiumDoctor  ✖ Xcode Command Line Tools are NOT installed!
info AppiumDoctor  ✔ DevToolsSecurity is enabled.
info AppiumDoctor  ✔ The Authorization DB is set up properly.
info AppiumDoctor  ✔ The Node.js binary was found at: /usr/local/bin/node
info AppiumDoctor  ✔ Carthage was found at: /usr/local/bin/carthage
info AppiumDoctor  ✔ HOME is set to: /Users/denglibing
WARN AppiumDoctor  ✖ ANDROID_HOME is NOT set!
info AppiumDoctor  ✔ JAVA_HOME is set to: /Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
WARN AppiumDoctor  ✖ adb could not be found because ANDROID_HOME is NOT set!
WARN AppiumDoctor  ✖ android could not be found because ANDROID_HOME is NOT set!
WARN AppiumDoctor  ✖ emulator could not be found because ANDROID_HOME is NOT set!
WARN AppiumDoctor  ✖ Bin directory for $JAVA_HOME is not set
info AppiumDoctor ### Diagnostic completed, 6 fixes needed. ###
info AppiumDoctor 
info AppiumDoctor ### Manual Fixes Needed ###
info AppiumDoctor The configuration cannot be automatically fixed, please do the following first:
WARN AppiumDoctor - Manually configure ANDROID_HOME.
WARN AppiumDoctor - Manually configure ANDROID_HOME and run appium-doctor again.
WARN AppiumDoctor - Add '$JAVA_HOME/bin' to your PATH environment
info AppiumDoctor ###
info AppiumDoctor 
info AppiumDoctor Bye, run appium-doctor again when all manual fixes have been applied!
info AppiumDoctor 
```

常见问题解决：

```shell
✖ Xcode Command Line Tools are NOT installed!
$ xcode-select --install   #按照提示安装即可

✖ Carthage was NOT found!
$ brew install carthage	   #carthage：类似cocoapods管理第三方代码，自动将工程编译为动态库，所以仅支持iOS8以上

WARN AppiumDoctor  ✖ ANDROID_HOME is NOT set!
但是建议安装安装编译器，可以解决其他Android环境的其他问题 AndroidStudio（https://developer.android.com/studio/index.html?hl=zh-cn）
设置 ANDROID_HOME 路径：
http://stackoverflow.com/questions/19986214/setting-android-home-enviromental-variable-on-mac-os-x
设置 JAVA_HOME 路径：
http://stackoverflow.com/questions/6588390/where-is-java-home-on-osx-sierra-10-12-el-captain-10-11-yosemite-10-10


JAVA_HOME问题：
当前最新JDK版本下载地址 Java SE Development Kit 8u111 (http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

.bash_profile文件：
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
export ANDROID_HOME=/Users/denglibing/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:${JAVA_HOME}/bin
```



7、安装appium客户端，上面的操作只是安装了server端，接下来需要安装测试代码运行所依赖的客户端，我们以python为例：

```shell
git clone https://github.com/appium/python-client
cd python-client-master 
sudo python setup.py install
```



8、对于 `iOS10` 以上的测试，需要下载安装 [appium-xcuitest-driver](https://github.com/appium/appium-xcuitest-driver) 驱动：`Appium iOS driver, backed by Apple XCUITest` , 当然也可以在 `/usr/local/lib/node_modules/appium/node_modules/.2.5.1@appium-xcuitest-driver/WebDriverAgent/` 找到该项目，执行 `carthage update` , 选择 `WebDriverAgentRunner` 编译即可。

其他相关的安装：

```shell
$ brew install libimobiledevice --HEAD  # install from HEAD to get important updates
$ brew install ideviceinstaller         # 只是对iOS9有用

$ npm install -g ios-deploy				# iOS10 以后的版本安装ios-deploy
$ sudo gem install xcpretty  			# 真机需要安装 xcpretty
```





9、下载测试代码并且测试，官网上的测试代码坑很多，这个问题我困扰我好久。一开始还以为是自己那个环境出问题还是流程不对，结果真的只是官网测试代码本身问题，当然可以自己编写代码测试咯，也可以下载我的一个简单的 [erduoniba/appium_ios_sample_code](https://github.com/erduoniba/appium_ios_sample_code)

先执行模拟器的测试

```shell
$ cd sample-code/apps/HHH

#在sample-code/apps/HHH/build/Release-iphonesimulator/得到HHH.app包
$ xcodebuild -sdk iphonesimulator
Build settings from command line:
    SDKROOT = iphonesimulator10.2

=== BUILD TARGET HHH OF PROJECT HHH WITH THE DEFAULT CONFIGURATION (Release) ===
...
...
...
Signing Identity:     "-"

    /usr/bin/codesign --force --sign - --timestamp=none /Users/denglibing/Desktop/sample-code/apps/HHH/build/Release-iphonesimulator/HHH.app

** BUILD SUCCEEDED **


$ cd sample-code/examples/python
$ python ios_simple_simulator.py
test_scroll (__main__.SimpleIOSTests) ... ok

----------------------------------------------------------------------
Ran 1 test in 30.421s

OK
```



```shell
# http://www.jianshu.com/p/3f43370437d2
# xcodebuild -workspace workspacename -scheme schemename [-destination destinationspecifier][-destination-timeout value] [-configuration configurationname][-sdk [sdkfullpath | sdkname]] [buildaction ...] [setting=value ...][-userdefault=value ...]
$ xcodebuild clean
$ xcodebuild -workspace Fangduoduo.xcworkspace -sdk iphonesimulator -scheme Fangduoduo -configuration "Release"
```





对于真机，可能要麻烦些，安装 [appium-xcuitest-driver](https://github.com/appium/appium-xcuitest-driver) 驱动后，生成在真机上运行的包：

参考资料可以看官网的说明 [部署ios-app-到手机上](http://appium.io/slate/cn/master/?ruby#部署ios-app-到手机上)  

```shell
# 在 /Users/denglibing/Desktop/build/Release-iphoneos/HHH.app 得到 release 包
$ xcodebuild clean
$ xcodebuild -sdk iphoneos -target HHH -configuration Release -derivedDataPath "/Users/denglibing/Desktop/build" CODE_SIGN_IDENTITY="iPhone Distribution: Shenzhen XXXX Technology Co., Ltd. (B9FH944VTE)" 
PROVISIONING_PROFILE="08f04032-ca2e-4bb5-b1ba-c32778115f2e"
```

关于怎么设置 `CODE_SIGN_IDENTITY` 和 `PROVISIONING_PROFILE` 如何查看 ，可以选择下图所示的 `Development Team` 一行, `control+c` 然后 复制出去即可

![](http://7xqhx8.com1.z0.glb.clouddn.com/2BC50FE7-1D4D-40CC-9B2C-75D53A5B9F67.png) 

也可以：

```python
CODE_SIGN_IDENTITY 为开发者证书标识，可以在 钥匙串访问 ->证书 -> 选中证书右键弹出菜单 -> 显示简介 -> 常用名称 获取，类似 iPhone Distribution: Company name Co. Ltd (xxxxxxxx9A), 包括括号内的内容。

PROVISIONING_PROFILE: 这个是 mobileprovision 文件的 identifier，获取方式：

Xcode -> Preferences -> 选中申请开发者证书的 Apple ID -> 选中开发者证书 -> View Details… -> 根据 Provisioning Profiles 的名字选中打包所需的 mobileprovision 文件 -> 右键菜单 -> Show in Finder -> 找到该文件后，除了该文件后缀名的字符串就是 PROVISIONING_PROFILE 字段的内容。
```



接下来连上真机 执行测试用例代码：

```shell
$ cd sample-code/examples/python
$ python ios_simple_device.py
test_scroll (__main__.SimpleIOSTests) ... ok

----------------------------------------------------------------------
Ran 1 test in 30.421s

OK
```





10：一些很有帮助的资料

很好的论坛  https://testerhome.com
appium源码地址： https://github.com/appium/appium
appium官网地址：http://appium.io/
appium中文说明文档：http://appium.io/slate/cn/master/?ruby#about-appium



11、心得：

整体的流程其实不多，但是在macOS10.12搭建appium的环境及测试整个过程着实烦躁，有时候一个小问题就卡1-2天也不好说，而且官网上的文档和测试代码已经部分误导我走了非常多的歪路。

比如

11.1、开始使用 appium1.5.3的mac客户端，虽然在macOS10.12上也有些问题，确实毕竟GUI用起来非常方便，可惜是基于 `UIAutomation` 框架来的，然后在iOS10中已经使用了新的框架 `XCUITest` ，只能废弃；

11.2、关于appium如果搭建啊，进行测试啊相关的文章其实不少，有中文的也有英文的，可能是作者在那个时候测试兼容过，但是iOS10上真的很多是误导了，而且搭建的流程我不建议阅读 `appium中文说明文档` ，毕竟年代太久远了。倒是可以在 `appium源码地址` 上多多发现类似的问题或者多多提问，作者及团队人员很热血 😊

11.3、还是环境问题，动不动就会失败，这个时候只能有耐心的多多看看log日志了，log少什么就满足他们，边边角角的一些依赖资源也可以了解了解😄，毕竟程序员需要耐得住寂寞😁，上面的流程基于当前最新的开发环境 `Xcode8.2` 、`iOS10.2` 




