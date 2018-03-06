### xcodebuild相关知识点

1、xcodebuild的常用命令：

xcodebuild 是苹果发布自动构建的工具，通过该命令可以生成app文件，它的一些命令可以通过 `man xcodebuild` 来查看对应的一些命令:

```shell
$ man xcodebuild
	xcodebuild [-project name.xcodeproj] -scheme schemename
                [[-destination destinationspecifier] ...] [-destination-timeout value]
                [-configuration configurationname] [-sdk [sdkfullpath | sdkname]]
                [action ...] [buildsetting=value ...] [-userdefault=value ...]

     xcodebuild -workspace name.xcworkspace -scheme schemename
                [[-destination destinationspecifier] ...] [-destination-timeout value]
                [-configuration configurationname] [-sdk [sdkfullpath | sdkname]]
                [action ...] [buildsetting=value ...] [-userdefault=value ...]

     xcodebuild -version [-sdk [sdkfullpath | sdkname]] [infoitem]

     xcodebuild -showsdks

     xcodebuild -showBuildSettings
                [-project name.xcodeproj | [-workspace name.xcworkspace -scheme schemename]]

     xcodebuild -list [-project name.xcodeproj | -workspace name.xcworkspace]

     xcodebuild -exportArchive -archivePath xcarchivepath -exportPath destinationpath
                -exportOptionsPlist path

     xcodebuild -exportLocalizations -project name.xcodeproj -localizationPath path
                [[-exportLanguage language] ...]
     xcodebuild -importLocalizations -project name.xcodeproj -localizationPath path
...
...
...
```

值得注意的是 `-showBuildSettings`: 显示工程的配置

```shell
$ xcodebuild -showBuildSettings
Build settings for action build and target HHH:
    ACTION = build
    AD_HOC_CODE_SIGNING_ALLOWED = NO
...
...
...
BUILD_DIR = /Users/denglibing/Library/Developer/Xcode/DerivedData/HHH-fumljkmvateusnbfpueycefmfgrb/Build/Products
    BUILD_ROOT = /Users/denglibing/Library/Developer/Xcode/DerivedData/HHH-fumljkmvateusnbfpueycefmfgrb/Build/Products
    BUILD_STYLE = 
    BUILD_VARIANTS = normal
...
...
...
	XCODE_APP_SUPPORT_DIR = /Applications/Xcode.app/Contents/Developer/Library/Xcode
    XCODE_PRODUCT_BUILD_VERSION = 8C1002
    XCODE_VERSION_ACTUAL = 0821
    XCODE_VERSION_MAJOR = 0800
    XCODE_VERSION_MINOR = 0820
    XPCSERVICES_FOLDER_PATH = HHH.app/XPCServices
    YACC = yacc
    arch = arm64
    diagnostic_message_length = 130
    variant = normal
```

比较通用生成app文件的写法如下:

```shell
# xcodebuild -workspace workspacename -scheme schemename [-destination destinationspecifier] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [buildaction ...] [setting=value ...] [-userdefault=value ...]

# 构建 Fangduoduo.xcworkspace 中的 Fangduoduo scheme,生成Release的模拟器包
$ xcodebuild -workspace Fangduoduo.xcworkspace -sdk iphonesimulator -scheme Fangduoduo -configuration "Release"
```

生成ipa文件，原本以为这个是xcrun的工作，但是在最新的macOS12和Xcode8上，发现xcrun命令已经废弃，所以还是使用xcodebuild来实现ipa文件的打包：

1.1 生成 `.xcarchive` 文件 :

```shell
$ cd Fangduoduo.xcworkspace path
$ xcodebuild archive -workspace Fangduoduo.xcworkspace -scheme Fangduoduo -archivePath Fangduoduo.xcarchive
```

生成的 Fangduoduo.xcarchive 里面包括三个文件，Fangduoduo.app.dsym文件(可用于bugly等监控bug的平台)，info.plist(保存打包的一些信息)，还有我们的 Fangduoduo.app 文件。

1.2 通过 `.xcarchive` 文件生成 `.ipa` 文件：

```shell
$ xcodebuild -exportArchive -archivePath Fangduoduo.xcarchive -exportPath Fangduoduo -exportFormat ipa
```



2、xcodebuild和Xcode中的操作有何区别？

Xcode的Build ( `command+B` ) 其实就是基于Xcode的工具 `Command Line Tools` 来实现，它不是Xcode安装自带的，需要执行 

```shell
$ xcode-select --install 
```

进行安装。当然也可以使用 `xcode-select --install` 来判断是否已经安装。而 `xcodebuild` 和 `xcrun` 则是  `Command Line Tools` 中的一部分。

当使用Xcode执行 Build 时，其实就是调用了 `xcodebuild` 命令

举个例子说明：

```shell
$ xcodebuild -workspace Fangduoduo.xcworkspace -sdk iphonesimulator -scheme Fangduoduo -configuration "Release" 
```

对应Xcode的设置如下：

![](http://7xqhx8.com1.z0.glb.clouddn.com/4F30A43F-2AF2-4538-B6B7-A8B00F637F9A.png) 

`command+B` 后，在Xcode的Products下找到 `Fangduoduo.app`，其实就是和xcodebuild生成的包是同一个地址，也就是说效果一致的。

