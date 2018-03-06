## CocoaPods是什么

**CocoaPods是一个用来帮助我们管理第三方依赖库的工具**，是iOS平台最常用的第三方类库管理工具，绝大部分有名的开源类库都支持CocoaPods。它可以解决库与库之间的依赖关系，下载库的源代码，同时通过创建一个Xcode的workspace来将这些第三方库和我们的工程连接起来，供我们开发使用。

使用CocoaPods的目的是让我们能自动化的、集中的、直观的管理第三方开源库。



## 为什么使用CocoaPods

在进行iOS开发的时候，总免不了使用第三方的开源库，比如SBJson、AFNetworking、Reachability等等。使用这些库的时候通常需要：

- 下载开源库的源代码并引入工程
- 向工程中添加开源库使用到的framework
- 解决开源库和开源库以及开源库和工程之间的依赖关系、检查重复添加的framework等问题
- 如果开源库有更新的时候，还需要将工程中使用的开源库删除，重新执行前面的三个步骤，顿时头都大了。。。

自从有了CocoaPods以后，这些繁杂的工作就不再需要我们亲力亲为了，只需要我们做好少量的配置工作，CocoaPods会为我们做好一切！



## 安装CocoaPods

### 安装

CocoaPods是用Ruby实现的，要想使用它首先需要有Ruby的环境。幸运的是OS X系统默认的已经可以运行Ruby了，因此我们只需要执行以下命令：

```
sudo gem install cocoapods
```

**升级CocoaPods**

升级很简单，再次执行安装命令即可：

```
sudo gem install cocoapods
```

需要注意的是，如果安装的时候使用了sudo，升级的时候一样需要使用该关键字，不然升级完了以后又会出现路径不匹配问题。



## 使用CocoaPods

**1、 创建Podfile**

使用也比较简单，cd到你的工程的根目录$projectPath

```
cd $projectPath
pod init
```

这个命令就会再工程根目录下面生成一个Podfile文件,podfile用文本编辑就行了。
你也可以使用`touch Podfile`进行创建Podfile

Podfile文件可以放在任意一个目录下，具体请看以下的Podfile配置

**2、 配置Podfile**

在你工程的**Podfile**文件添加

```
pod ‘kerkee’, ’~> 1.0.0’
```

或者你想直接使用最新版可把版本号去掉，使用以下这句

```
pod ‘kerkee’
```

我帖出个podfile文件配置的例子

```
# Uncomment this line to define a global platform for your project
# platform :ios, '8.0'
# Uncomment this line if you're using Swift
# use_frameworks!

target 'TestKerkeePod' do

#or this way，If it is a team work,please do not use this way.
#because it use last version of kerkee,it is a bad way
#pod ‘kerkee’
pod ‘kerkee’, ’~> 1.0.0’

end

target 'TestKerkeePodTests' do
end

target 'TestKerkeePodUITests' do
end
```

Podfile本质上是用来描述Xcode工程中的targets用的。如果我们不显式指定Podfile对应的target，CocoaPods会创建一个名称为default的隐式target，会和我们工程中的第一个target相对应。**换句话说，如果在Podfile中没有指定target，那么只有工程里的第一个target能够使用Podfile中描述的Pods依赖库。**

如果想在同一个工程中的一个Podfile中同时描述project中的多个target，根据需求的不同，可以有不同的实现方式。

① 多个target中使用相同的Pods依赖库

比如，名称为TestKerkeePod的target和SecondTarget的target都需要使用Reachability、SBJson、AFNetworking、kerkee四个Pods依赖库，可以使用`link_with`关键字来实现，将Podfile写成如下方式：

```
link_with 'TestKerkeePod', 'SecondTarget'  
platform :ios    
pod 'Reachability',  '~> 3.0.0'    
pod 'SBJson', '~> 4.0.0'  
pod ‘kerkee’, ’~> 1.0.0’  
    
platform :ios, '7.0'    
pod 'AFNetworking', '~> 2.0'
```

这种写法就实现了TestKerkeePod和SecondTarget两个target共用相同的Pods依赖库。

② 不同的target使用完全不同的Pods依赖库

TestKerkeePod这个target使用的是Reachability、SBJson、AFNetworking、kerkee四个依赖库，但SecondTarget这个target只需要使用OpenUDID这一个依赖库，这时可以使用target关键字，Podfile的描述方式如下：

```
target :'TestKerkeePod' do  
platform :ios    
pod 'Reachability',  '~> 3.0.0'    
pod 'SBJson', '~> 4.0.0'  
pod ‘kerkee’, ’~> 1.0.0’  
pod 'FddPatchSDK', :git => 'ssh://teamcode.fangdd.net:29418/fangdd/customer/hotfix-ios'
    
platform :ios, '7.0'    
pod 'AFNetworking', '~> 2.0'  
end  
  
target :'SecondTarget' do  
pod 'OpenUDID', '~> 1.0.0'  
end
```

其中，do/end作为开始和结束标识符。

**关于于Podfile文件的路径**

事实上Podfile文件可以放在任意一个目录下，需要做的是在Podfile中指定工程的路径，和原来相比，Podfile文件就在最开始的位置增加了一行，具体内容如下：

```
xcodeproj "/Users/zihong/Desktop/test/TestKerkeePod/TestKerkeePod.xcodeproj"  
  
platform :ios    
pod 'Reachability',  '~> 3.0.0'    
pod 'SBJson', '~> 4.0.0' 
pod ‘kerkee’, ’~> 1.0.0’    

platform :ios, '7.0'    
pod 'AFNetworking', '~> 2.0'
```

指定路径使用的是xcodeproj关键字。

此后，进入Podfile文件所在路径，执行pod install命令（往下文看）就会和之前一样下载这些Pods依赖库，而且生成的相关文件都放在了Podfile所在目录下面。

**3、执行导入命令**

使用终端cd 到你的Podfile所在的目录，然后再使用`pod install`就可以把这些开源库导入，CocoaPods就开始为我们做下载源码、配置依赖关系、引入需要的framework等一些列工作。

```
cd $projectPath
pod install --verbose
```

可以看到，工程的根目录下多了三个东西：yourproject.xcworkspace、Podfile.lock文件和Pods目录

安装完，这些库都放在一个Pods的工程中，然后会用xcode的workspace来管理这个工程和你自己的工程。以后就打开`yourproject.xcworkspace`就行了。



## 说明

对于工程发生的变化，有几点需要说明：

- *第三方库会被编译成静态库供我们正真的工程使用*

CocoaPods会将所有的第三方库以target的方式组成一个名为Pods的工程，该工程就放在刚才新生成的Pods目录下。整个第三方库工程会生成一个名称为libPods.a的静态库提供给我们自己的CocoaPodsTest工程使用。

- *我们的工程和第三方库所在的工程会由一个新生成的workspace管理*

为了方便我们直观的管理工程和第三方库，我们的工程和Pods工程会被以workspace的形式组织和管理，也就是我们刚才看到的`yourproject.xcworkspace`文件。

原来的工程设置已经被更改了，这时候我们直接打开原来的工程文件去编译就会报错，只能使用新生成的workspace来进行项目管理。



- 关于Podfile.lock文件

上文讲过，在开始使用CocoaPods，执行完pod install之后，会生成一个Podfile.lock文件。这个文件看起来跟我们关系不大，*实际上绝对不应该忽略它*。

该文件用于保存已经安装的Pods依赖库的版本，通过CocoaPods安装了SBJson、AFNetworking、Reachability三个POds依赖库以后对应的Podfile.lock文件内容为：

```
PODS:  
  - AFNetworking (2.1.0):  
    - AFNetworking/NSURLConnection  
    - AFNetworking/NSURLSession  
    - AFNetworking/Reachability  
    - AFNetworking/Security  
    - AFNetworking/Serialization  
    - AFNetworking/UIKit  
  - AFNetworking/NSURLConnection (2.1.0):  
    - AFNetworking/Reachability  
    - AFNetworking/Security  
    - AFNetworking/Serialization  
  - AFNetworking/NSURLSession (2.1.0):  
    - AFNetworking/NSURLConnection  
  - AFNetworking/Reachability (2.1.0)  
  - AFNetworking/Security (2.1.0)  
  - AFNetworking/Serialization (2.1.0)  
  - AFNetworking/UIKit (2.1.0):  
    - AFNetworking/NSURLConnection  
  - Reachability (3.0.0)  
  - SBJson (4.0.0)  
  
DEPENDENCIES:  
  - AFNetworking (~> 2.0)  
  - Reachability (~> 3.0.0)  
  - SBJson (~> 4.0.0)  
  
SPEC CHECKSUMS:  
  AFNetworking: c7d7901a83f631414c7eda1737261f696101a5cd  
  Reachability: 500bd76bf6cd8ff2c6fb715fc5f44ef6e4c024f2  
  SBJson: f3c686806e8e36ab89e020189ac582ba26ec4220  
  
COCOAPODS: 0.29.0
```

**Podfile.lock文件最大得用处在于多人开发。**对于没有在Podfile中指定Pods依赖库版本的写法，如下：

```
pod 'SBJson'
```

该句话用于获取当前SBJson这个Pods依赖库的最新版本。

当团队中的某个人执行完pod install命令后，生成的Podfile.lock文件就记录下了当时最新Pods依赖库的版本，这时团队中的其它人check下来这份包含Podfile.lock文件的工程以后，再去执行pod install命令时，获取下来的Pods依赖库的版本就和最开始用户获取到的版本一致。

如果没有Podfile.lock文件，后续所有用户执行pod install命令都会获取最新版本的SBJson，这就有可能造成同一个团队使用的依赖库版本不一致，这对团队协作来说绝对是个**灾难**！

在这种情况下，如果团队想使用当前最新版本的SBJson依赖库，有两种方案：

> 更改Podfile，使其指向最新版本的SBJson依赖库；
>
> 执行pod update命令；

**鉴于Podfile.lock文件对团队协作如此重要，我们需要将它添加到版本管理中。**



对`.spec`文件的理解： 在[Specs仓库](https://github.com/CocoaPods/Specs.git)中包含了所有`Pod项目`，随便点击一个`Pod项目`进去可以发现里面只有一个`json`文件：(里面内容其实就是开发者上传的`.spec`文件转`json`后的文件) 同样的，我们在安装或者更新`CocoaPods`的时候,实际上就是在更新`Specs仓库`，本地存放地址是：`.cocoapods/repos/master/Specs`