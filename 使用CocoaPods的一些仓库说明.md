#### 使用CocoaPods的一些仓库说明：



[CocoaPods官网](https://cocoapods.org/)  

#### 1、pod使用官网的仓库的关联代码（这些代码需要cocoapods审核通过才能被其他人使用，而且每次稳定的代码版本需要打上tag方便使用者选择对应的tag代码）

上传至cocoapods的公开的 [仓库](https://github.com/CocoaPods/Specs.git) 使用 `pod setup` 或者 `pod install` 时会从仓库中下载到本地，存放在电脑的 `.cocoapods/repo/master` 中，这个仓库是cocoapods团队维护，大部分开源代码都放在这里，当然本地放的只是项目的一些相关信息：

**CAIStatusBar.podspec.json** 

```json
{
  "name": "CAIStatusBar",
  "version": "0.0.1",
  "summary": "A simple indicator",
  "homepage": "https://github.com/apple5566/CAIStatusBar.git",
  "license": "MIT",
  "authors": {
    "apple5566": "zaijiank110@sohu.com"
  },
  "platforms": {
    "ios": "6.0"
  },
  "source": {
    "git": "https://github.com/apple5566/CAIStatusBar.git",
    "tag": "0.0.1"
  },
  "source_files": "CAIStatusBar/**/*.{h,m}",
  "resources": "CAIStatusBar/CAIStatusBar.bundle",
  "requires_arc": true
}
```

需要使用cocoapods的项目的[Podfile文件说明](https://guides.cocoapods.org/syntax/podfile.html)

**Podfile:**

```ruby
platform :ios,'8.0'
use_frameworks!
inhibit_all_warnings!

target 'HDDemo' do
	pod 'AFNetworking'
	pod 'MJRefresh'
end
```



#### 2、pod使用本地路径代码 （不需要经过类似cocoapods审核流程即可使用，本地代码一改，pod update即可获取最新代码，但是一不小心代码地址移动或者删除时，会有问题）

**Podfile:** 

```ruby
platform :ios,'8.0'
use_frameworks!
inhibit_all_warnings!
target 'HDDemo' do
	pod 'AFNetworking', :path => '~/Documents/AFNetworking'
end
```



#### 3、pod使用源码地址的代码（每次pod update即可获取给定地址的最新代码，也可以选择指定的tag，源码每次push到git地址后，其他项目即可使用，无审核流程，这种模式属于比较灵活的方式）

**Podfile:** 

```ruby
platform :ios,'8.0'
use_frameworks!
inhibit_all_warnings!
target 'HDDemo' do
  # 只拉取FDDUITableViewDemoSwift/FDDBaseRepo这个下面的代码, FDDUITableViewDemoSwift.podspec见下面
	pod 'FDDUITableViewDemoSwift/FDDBaseRepo', :git => 'https://github.com/erduoniba/FDDUITableViewDemoSwift.git' ,:tag => '0.1.0'
 	pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :branch => 'dev'
  	pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :commit => '082f8319af'
end
```

pod update --verbose --no-repo-update 即可拉取最新代码



#### 4、pod使用podspec文件来拉取代码：

**Podfile：**

```ruby
platform :ios,'8.0'
use_frameworks!
inhibit_all_warnings!
target 'HDDemo' do
  	# 远程podspec地址
	pod 'FDDUITableViewDemoSwift', :podspec => 'https://raw.githubusercontent.com/erduoniba/FDDUITableViewDemoSwift/master/FDDUITableViewDemoSwift.podspec'
    # 本地podspec地址
  	pod 'FDDUITableViewDemoSwift', :podspec => '/Users/denglibing/project/harryProject/FDDUITableViewDemoSwift/FDDUITableViewDemoSwift.podspec'
end
```

**FDDUITableViewDemoSwift.podspec**

[podspec说明](https://guides.cocoapods.org/syntax/podspec.html) 

```ruby
Pod::Spec.new do |s|
    s.name         = 'FDDUITableViewDemoSwift'
    s.version      = "0.1.1"
    s.license      = { :type => 'MIT', :file => 'LICENSE' }
    s.author       = { 'denglibing' => 'denglibing@fangdd.com' }
    s.summary      = 'FDDUITableViewDemoSwift'

    s.platform     =  :ios, '8.0'
    s.homepage     = "https://github.com/erduoniba/FDDUITableViewDemoSwift"

    s.source       =  { :git => 'https://github.com/erduoniba/FDDUITableViewDemoSwift.git', :tag => "#{s.version}"}
    s.module_name  = 'FDDUITableViewDemoSwift'
    s.framework    = 'UIKit'
    s.requires_arc = true

    # Pod Dependencies

    s.subspec 'FDDBaseRepo' do |ss|
        ss.source_files = 'FDDUITableViewDemoSwift/FDDBaseRepo/*'
        ss.resources = ["FDDUITableViewDemoSwift/FDDBaseRepo/Resources/*"]
        ss.dependency 'PullToRefresher'
    end
end
```



#### 5、pod使用自己的私有仓库来替换cocoapods的仓库，这样同样的不需要审核流程，自己管理所有的 podsepc文件，需要加上tag来拉取指定的代码



[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/) 

1. 创建并设置一个私有的`Spec Repo`。这个仓库你可以创建私有的也可以创建公开的，不过既然私有的`Spec Repo`，还是创建私有的仓库吧。创建完成之后在`Terminal`中执行如下命令

   ```shell
   # pod repo add [Private Repo Name] [GitHub HTTPS clone URL]
   $ pod repo add HDPodRepo https://github.com/erduoniba/HDPodRepo.git
   ```

   此时如果成功的话进入到`~/.cocoapods/repos`目录下就可以看到 `HDPodRepo`  这个目录了。至此第一步创建私有`Spec Repo`完成。

2. 创建`Pod`的所需要的项目工程文件，并且有可访问的项目版本控制地址。

3. 创建`Pod`所对应的`podspec`文件。

4. 本地测试配置好的`podspec`文件是否可用。（不合格问题也不大）

5. 向私有的`Spec Repo`中提交`podspec`。

   ```shell
   $ cd path/FDDUITableViewDemoSwift.podspec
   $ pod repo push HDPodRepo FDDUITableViewDemoSwift.podspec  #前面是本地Repo名字 后面是podspec名字

   # 成功之后
   $ cd /Users/denglibing/.cocoapods/repos/HDPodRepo 
   $ tree
   .
   ├── FDDUITableViewDemoSwift
   │   ├── 0.1.1
   │   │   └── FDDUITableViewDemoSwift.podspec
   │   └── 0.1.2
   │       └── FDDUITableViewDemoSwift.podspec
   └── README.md
   # 私有库自动生成了最新的代码
   ```

6. 在个人项目中的`Podfile`中增加刚刚制作的好的`Pod`并使用。

   **Podfile:** 

   ```ruby
   source 'https://github.com/CocoaPods/Specs.git'			# 官方库地址
   source 'https://github.com/erduoniba/HDPodRepo.git'     # 私有库地址

   platform :ios,'8.0'
   use_frameworks!
   inhibit_all_warnings!
   target 'HDDemo' do
     	pod 'FDDUITableViewDemoSwift'	# 私有库地址里的FDDUITableViewDemoSwift项目
   end
   ```

   ​

7. 更新维护`podspec`。

   ```shell
   $ cd anyPath
   $ pod repo remove HDPodRepo #删除本地的私有库
   $ pod repo add HDPodRepo https://github.com/erduoniba/HDPodRepo.git #重新添加私有库地址
   ```

   ​





一些错误：

```shell
$ pod lib lint --verbose --no-clean --allow-warnings
We get the error below:

** BUILD FAILED **

The following build commands failed:
CompileSwift normal x86_64 /var/folders/yg/dlxwsn292j108t5qtlmgbtfh0000gn/T/CocoaPods/Lint/Pods/ReachabilitySwift/Reachability/Reachability.swift
CompileSwiftSources normal x86_64 com.apple.xcode.tools.swift.compiler
(2 failures)
...
...
...

Pods workspace available at /var/folders/yg/dlxwsn292j108t5qtlmgbtfh0000gn/T/CocoaPods/Lint/App.xcworkspace for inspection.

[!] Teste did not pass validation, due to 69 errors.

```

https://github.com/ashleymills/Reachability.swift/issues/146

解决：

```html
在仓库主目录建立 .swift-version 文件，文件添加 "3.0" 即可
同时在对应的 FDDUITableViewDemoSwift.podspec 目录下也添加 .swift-version 文件
```

