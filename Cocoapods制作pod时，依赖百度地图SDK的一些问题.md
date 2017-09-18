### Cocoapods制作pod时，依赖百度地图SDK的一些问题

#### 1、制作一个pod时，依赖了百度地图sdk（静态库），这个时候，Cocoapods会在百度地图sdk这个pod中的podspec中为项目配置百度地图sdk需要的系统动态库及静态库。编译新做的pod，得到framework，分析它的二进制，没有多余的代码，但是二进制却有6.9M，使用 otool 分析得到：

```shell
denglibing$ otool -L /Users/denglibing/Library/Developer/Xcode/DerivedData/HDPodBMKSDK-hbkvrvffwquplafimjwrnqklfktr/Build/Products/Debug-iphonesimulator/HDPodBMKSDK.framework/HDPodBMKSDK 
/Users/denglibing/Library/Developer/Xcode/DerivedData/HDPodBMKSDK-hbkvrvffwquplafimjwrnqklfktr/Build/Products/Debug-iphonesimulator/HDPodBMKSDK.framework/HDPodBMKSDK:
@rpath/HDPodBMKSDK.framework/HDPodBMKSDK (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libsqlite3.dylib (compatibility version 9.0.0, current version 254.6.0)
/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
/System/Library/Frameworks/CoreGraphics.framework/CoreGraphics (compatibility version 64.0.0, current version 1070.22.0)
/System/Library/Frameworks/CoreLocation.framework/CoreLocation (compatibility version 1.0.0, current version 2101.0.62)
/System/Library/Frameworks/CoreTelephony.framework/CoreTelephony (compatibility version 1.0.0, current version 0.0.0)
/System/Library/Frameworks/OpenGLES.framework/OpenGLES (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/QuartzCore.framework/QuartzCore (compatibility version 1.2.0, current version 1.11.0)
/System/Library/Frameworks/Security.framework/Security (compatibility version 1.0.0, current version 0.0.0)
/System/Library/Frameworks/SystemConfiguration.framework/SystemConfiguration (compatibility version 1.0.0, current version 888.50.20)
/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1349.54.0)
/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
/usr/lib/libSystem.dylib (compatibility version 1.0.0, current version 1238.50.2)
/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1349.55.0)
/System/Library/Frameworks/UIKit.framework/UIKit (compatibility version 1.0.0, current version 3600.7.47)
/usr/lib/libc++abi.dylib (compatibility version 1.0.0, current version 307.2.0)
```

其中 libsqlite3.dylib 我找了一下发现有4.4M

而libstdc++.6.dylib则是libstdc++.6.0.9.dylib的替身，libstdc++.6.0.9.dylib是1.5M

libobjc.A.dylib有14.2M

libSystem.dylib则是libSystem.B.dylib的替身，libSystem.B.dylib大小是61KB

libc++abi.dylib有440KB

/System/Library/Frameworks/下面的一些系统的framework都很大，比如/System/Library/Frameworks/CoreGraphics.framework/CoreGraphics就有23.8M

显然，这些 /usr/lib/ 和 /System/Library/Frameworks/ 下面的库肯定不是在我刚刚制作的二进制中，它们应该只是依赖关系，用到的时候通过地址去找。

**那么这6.9M的体积是怎么来的呢？** 下面回给出答案。

#### 2、在主工程使用刚刚制作好的framework：

制作的pod的podspec

```ruby
# HDPodBMKSDK.podspec 
Pod::Spec.new do |s|
    s.name         = "HDPodBMKSDK"
    s.module_name  = "HDPodBMKSDK"
    s.version      = "0.0.1"
    s.summary      = "HDPodBMKSDK"
    s.description  = "这个是用到百度地图sdk的一个pod工程"
    s.homepage     = "https://github.com/erduoniba"
    s.author       = { "denglibing" => "13049862397@163.com" }
    s.platform     = :ios, "8.0"
    s.requires_arc = true
    s.license      = { :type => 'MIT'}
    s.source       =  { :git => 'https://github.com/erduoniba/HDPodBMKSDK.git', :tag => "#{s.version}"}

    s.subspec 'HDPodBMKSDK' do |hdBMK|
        hdBMK.source_files = 'HDPodBMKSDK/*.{h,m,mm}'
    end

    s.dependency 'BaiduMapKit'

    #-undefined dynamic_lookup 这个表明了当主工程和framework都包含同一个库时，会优先使用主工程的库。
    s.pod_target_xcconfig = {
        'FRAMEWORK_SEARCH_PATHS'   => '$(inherited) $(PODS_ROOT)/BaiduMapKit/BaiduMapKit',
        'LIBRARY_SEARCH_PATHS'     => '$(inherited) $(PODS_ROOT)/BaiduMapKit/BaiduMapKit/thirdlibs',
        'OTHER_LDFLAGS'            => '$(inherited) -undefined dynamic_lookup -ObjC',
        'ENABLE_BITCODE'           => 'NO'
    }
end
```

主工程的Podfile：

```ruby
platform :ios,'8.0'

#cocoapods使用framework模式，意思就是.a的静态库不支持使用cocoapods管理，在包含swift的项目中，需要使用这样的模式，因为swift不支持静态库，(原因：静态库在编译的时候就打包加入到项目的二进制中，但是目前的iPhone设备中还没有支持对swift的解析，所以不能载静态库中编写swift代码，动态库则可以，因为项目使用swift后会在项目中添加swift解析的动态库到工程中，也就是如此所以使用swift的项目打包出来的ipa体积会变大，如下图，当引用的swift需要的依赖库越多，占用的体积越大)
use_frameworks!
inhibit_all_warnings!

target 
'HDPodBMKDemo' do
pod 'HDPodBMKSDK', :path => ‘../HDPodBMKSDK’
end

#百度地图SDK现在使用的7个framework，为了支持ssl所以还添加了两个.a的静态库，这个时候需要使用如下命令来让cocoapods对静态库支持

# pod1.3.0之前需要这样
pre_install do |installer|
# workaround for https://github.com/CocoaPods/CocoaPods/issues/3289
def installer.verify_no_static_framework_transitive_dependencies; end
end

# pod1.3.0之后需要这样
pre_install do |installer|
    # workaround for https://github.com/CocoaPods/CocoaPods/issues/3289
    Pod::Installer::Xcode::TargetValidator.send(:define_method, :verify_no_static_framework_transitive_dependencies) {}
end
```

使用swift的项目打包出来的ipa体积会变大：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk1.jpg) 



编译主工程得到ipa包，打开后：

```shell
denglibing$ tree -L 3
├── Base.lproj
│   ├── LaunchScreen.storyboardc
│   │   ├── 01J-lp-oVM-view-Ze5-6b-2t3.nib
│   │   ├── Info.plist
│   │   └── UIViewController-01J-lp-oVM.nib
│   └── Main.storyboardc
│       ├── BYZ-38-t0r-view-8bC-Xf-vdC.nib
│       ├── Info.plist
│       └── UIViewController-BYZ-38-t0r.nib
├── Frameworks
│   └── HDPodBMKSDK.framework
│       ├── HDPodBMKSDK
│       ├── Info.plist
│       └── _CodeSignature
├── HDPodBMKDemo
├── Info.plist
├── PkgInfo
├── _CodeSignature
│   └── CodeResources
└── mapapi.bundle
    ├── files
    │   └── cfg
    └── images
        ├── baidumap_logo.png
```

果然发现了刚刚我们所做的framework，查看大小，惊人的发现它只有了53KB

继续使用otool分析：

```shell
denglibing$ otool -L HDPodBMKSDK 
HDPodBMKSDK:
@rpath/HDPodBMKSDK.framework/HDPodBMKSDK (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1349.54.0)
/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
/usr/lib/libSystem.dylib (compatibility version 1.0.0, current version 1238.50.2)
/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1349.55.0)
/System/Library/Frameworks/CoreGraphics.framework/CoreGraphics (compatibility version 64.0.0, current version 1070.22.0)
/System/Library/Frameworks/UIKit.framework/UIKit (compatibility version 1.0.0, current version 3600.7.47)
```

和上面的对比，现在少了这些：

```shell
/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
/System/Library/Frameworks/CoreLocation.framework/CoreLocation (compatibility version 1.0.0, current version 2101.0.62)
/System/Library/Frameworks/CoreTelephony.framework/CoreTelephony (compatibility version 1.0.0, current version 0.0.0)
/System/Library/Frameworks/OpenGLES.framework/OpenGLES (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/QuartzCore.framework/QuartzCore (compatibility version 1.2.0, current version 1.11.0)
/System/Library/Frameworks/Security.framework/Security (compatibility version 1.0.0, current version 0.0.0)
/System/Library/Frameworks/SystemConfiguration.framework/SystemConfiguration (compatibility version 1.0.0, current version 888.50.20)
/usr/lib/libc++abi.dylib (compatibility version 1.0.0, current version 307.2.0)
```



查看百度地图官网的集成：

```html
http://lbsyun.baidu.com/index.php?title=iossdk/guide/buildproject
第二步、引入所需的系统库
百度地图SDK中提供了定位功能和动画效果，v2.0.0版本开始使用OpenGL渲染，因此您需要在您的Xcode工程中引入CoreLocation.framework和QuartzCore.framework、OpenGLES.framework、SystemConfiguration.framework、CoreGraphics.framework、Security.framework、libsqlite3.0.tbd（xcode7以前为 libsqlite3.0.dylib）、CoreTelephony.framework 、libstdc++.6.0.9.tbd（xcode7以前为libstdc++.6.0.9.dylib）。

（注：红色标识的系统库为v2.9.0新增的系统库，使用v2.9.0及以上版本的地图SDK，务必增加导入这3个系统库。）

添加方式： 在Xcode的Project -> Active Target ->Build Phases ->Link Binary With Libraries，添加这几个系统库即可。
```

由此可见，动态库独立出去的时候，包含了动态库能独立在运行时所需要的环境，但是没有想到这些需要环境的说明竟然占居了这么大的体积，虽然最后在加入到项目中后会把这部分体积清空掉，这也让我想到，不要一味的看第三方framework的体积了，毕竟它需要依赖的系统动态库也需要很多体积进行说明。

而现在，注意观察ipa，一共是13M，其中主项目的二进制占了6.9M，百度地图的资源占用了6M，说明百度需要系统库从刚刚我们自己做的framework转移给了 主项目的二进制，**但是最终这 6.9M的体积何去何从在我这里还是一个谜**

otool分析这个二进制，发现和打包出来的framework一致。

和同事讨论，他的意思这多出来的6.9M肯定是百度地图SDK产生的，而不是上面所说的对依赖的系统动态库说明产生的。这个可能是Cocoapods做了很多处理：

单独打包framework的时候，因为百度地图sdk有.a静态库，所以Cocoapods默认将其全部做成了静态库，加入到自己做的framework中，当主工程也使用这个百度的时候，Cocoapods给我们的sdk做了一次引用，但是同样将它们做成静态库加入到 项目的二进制中，**也就是说这6.9M是这样转移了的。**

但是这个需要证实。

**这里有一个大问题：百度地图相关的代码嵌入到了主工程的二进制中，在自己制作的framework中是没有相关百度地图代码的，这样明显跑不起来的。**

这里会遇到一下问题：

**问题:**  为什么使用pod，debug可以跑起来，而release却失败？

```html
# http://www.tuicool.com/articles/Ybq6Rf3
release打包出来安装，一点击运行，变crash了，View Device Logs：

Exception Type:  EXC_CRASH (SIGABRT)
Exception Codes: 0x0000000000000000, 0x0000000000000000
Exception Note:  EXC_CORPSE_NOTIFY

Termination Description: DYLD, Symbol not found: OBJC_CLASS$_BMKMapView | Referenced from: /private/var/containers/Bundle/Application/50F3B444-F54E-4AF1-8C13-D60FBEC0F669/HDPodBMKDemo.app/Frameworks/HDPodBMKSDK.framework/HDPodBMKSDK | Expected in: flat namespace | in /private/var/containers/Bundle/Application/50F3B444-F54E-4AF1-8C13-D60FBEC0F669/HDPodBMKDemo.app/Frameworks/HDPodBMKSDK.framework/HDPodBMKSDK

Triggered by Thread:  0
```

原因解答：因为 Debug 版本暴露了所有自定义类的符号以便于调试，因此你的 framework 可以找到相应的符号，而 Release 版本则不会。

#### 3、既然这样，怎么处理呢？

**3.1、主工程和framework同时包含百度地图好了。**

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk2.jpg) 

像这样子，就可以了，但是值得注意的地方是，两边都有百度地图，需要在两边进入地图实现百度的一些代码，也就是下面的代码需要写两份：

```objective-c
    BMKMapManager* _mapManager = [[BMKMapManager alloc]init];
    // 如果要关注网络及授权验证事件，请设定     generalDelegate参数
    BOOL ret = [_mapManager start:@"qbfQUpMvsAkqgwFZL8YVy89dK9ZXZE3h" generalDelegate:nil];
    if (!ret) {
        NSLog(@"manager start failed!");
    }
```

这样无论是debug还是release，都是正常的，缺点很明显百度在两边都存在了,项目体积也大了：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk3.jpg) 



![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk4.jpg) 



同时在运行的时候Xcode的log日志：

```html
objc[10502]: Class BMSDKKeychainItemWrapper is implemented in both /private/var/containers/Bundle/Application/B5A38F9D-E271-491C-B95D-6C05671D2332/HDPodBMKDemo.app/Frameworks/HDPodBMKSDK.framework/HDPodBMKSDK (0x100f951d8) and /var/containers/Bundle/Application/B5A38F9D-E271-491C-B95D-6C05671D2332/HDPodBMKDemo.app/HDPodBMKDemo (0x1004ad708). One of the two will be used. Which one is undefined.

objc[10502]: Class BMSDKUDID is implemented in both /private/var/containers/Bundle/Application/B5A38F9D-E271-491C-B95D-6C05671D2332/HDPodBMKDemo.app/Frameworks/HDPodBMKSDK.framework/HDPodBMKSDK (0x100f95228) and /var/containers/Bundle/Application/B5A38F9D-E271-491C-B95D-6C05671D2332/HDPodBMKDemo.app/HDPodBMKDemo (0x1004ad758). One of the two will be used. Which one is undefined.
```

确实也在说明两边都有百度地图这个鬼东西。

当然不能每次在pod update HDPodBMKSDK 重新勾选百度地图到 HDPodBMKSDK中，脚本来解决(主工程中的Podfile中)：

```ruby
# http://www.rubydoc.info/gems/cocoapods/Pod/Project

post_install do |installer|
    project_location = './Pods/Pods.xcodeproj'
    # 设置使用#{framework_names}对应的target
    target_names = ['HDPodBMKSDK']
    framework_names = [ 'BaiduMapKit' ]

    project = installer.pods_project

    framework_names.each do |framework_name|
        frameworks = project.pod_group(framework_name)
        .children
        .find { |group| group.name == 'Frameworks' }
        .children

        target_names.each do |target_name|
            target = project.targets.find { |target| target.to_s == target_name }
            frameworks_group = project.groups.find { |group| group.display_name == 'Frameworks' }
            frameworks_build_phase = target.build_phases.find { |build_phase| build_phase.to_s == 'FrameworksBuildPhase' }

            frameworks.each do |file_ref|
                frameworks_build_phase.add_file_reference(file_ref)
            end
        end
    end
end
```

或者在工程的 Pods.xcodeproj 同级路径下，添加一个 ruby 脚本：podTaget.rb :

```ruby
# https://github.com/CocoaPods/Xcodeproj/issues/408

require 'xcodeproj'

#project related valus
project_path = "/Users/denglibing/project/harryProject/HDPodBMKDemo/Pods/Pods.xcodeproj"
target_name = "HDPodBMKSDK"
project = Xcodeproj::Project.open(project_path)
target = project.targets.find { |target| target.to_s == target_name }
frameworks_build_phase = target.build_phases.find { |build_phase| build_phase.to_s == 'FrameworksBuildPhase' }
frameworks_group = project.groups.find { |group| group.display_name == 'Frameworks' }

baidusdk_framework_path = "/Users/denglibing/project/harryProject/HDPodBMKDemo/Pods/BaiduMapKit/BaiduMapKit/thirdlibs/"
framework_names = [
'BaiduMapAPI_Base.framework',
'BaiduMapAPI_Cloud.framework',
'BaiduMapAPI_Location.framework',
'BaiduMapAPI_Map.framework',
'BaiduMapAPI_Radar.framework',
'BaiduMapAPI_Search.framework',
'BaiduMapAPI_Utils.framework',
]


# Add framework to target as "Linked Frameworks"
framework_names.each do |framework_name|
    framework_ref = frameworks_group.new_file("#{baidusdk_framework_path}/#{framework_name}")
    frameworks_build_phase.add_file_reference(framework_ref)
end

# Save the project
project.save()
```

或者简单的来，在包含百度地图的pod的 podspec中，写好如下的代码（关键代码）：

```ruby
s.frameworks   = "CoreLocation", "CoreTelephony", "OpenGLES", "QuartzCore", "Security", "SystemConfiguration", "BaiduMapAPI_Base", "BaiduMapAPI_Cloud", "BaiduMapAPI_Location", "BaiduMapAPI_Map", "BaiduMapAPI_Radar", "BaiduMapAPI_Search", "BaiduMapAPI_Utils"
s.libraries    = "z", "sqlite3.0", "stdc++.6.0.9", "crypto", “ssl"
s.pod_target_xcconfig = {
        'FRAMEWORK_SEARCH_PATHS'   => '$(inherited) $(PODS_ROOT)/BaiduMapKit/BaiduMapKit',
        'LIBRARY_SEARCH_PATHS'     => '$(inherited) $(PODS_ROOT)/BaiduMapKit/BaiduMapKit/thirdlibs',
        'OTHER_LDFLAGS'            => '$(inherited) -undefined dynamic_lookup -lObjC',
        'ENABLE_BITCODE'           => 'NO'
}
```

**3.2、framework包含百度地图，帮助主工程完成百度地图相关代码，这个肯定可行，但是感觉不怎么通用。毕竟framework不能实现主工程的代码逻辑吧**

**3.3、放弃Cocoapods管理framework，自己打包加入到主工程中。这个可行，从打包后的结构看出主项目的二进制并没有百度地图相关代码，它们存在framework中**

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk41.jpg) 



![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk5.jpg) 

方案可行。



#### 4、下面是测试的代码地址：

主项目和framework都是通过Cocoapods来依赖百度地图sdk：

制作的framework工程地址： [https://github.com/erduoniba/HDPodBMKSDK](https://github.com/erduoniba/HDPodBMKSDK)

项目主工程地址：[https://github.com/erduoniba/HDPodBMKDemo](https://github.com/erduoniba/HDPodBMKDemo)

这样其实两边都是有百度地图sdk的代码的，造成项目体积变大，虽然我测试过程中Xcode只是提示 One of the two will be used. Which one is undefined 运行没有问题，但是是否存在一些隐性的问题还待发现。



#### 5、其他

同样的，经过我的实验（不用Cocoapods管理），动态库HDFramework2是可以嵌入HDFramework3（二进制未嵌入，HDFramework3二进制还是存在HDFramework3.framework中）的，在主工程HDFrameworkDemo中只需要有HDFramework2即可; 项目打包后结构：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk6.jpg) 

HDFramework2如何嵌入HDFramework3：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk7.jpg) 

同样的，经过我的实验，静态库HDFramework2是可以嵌入HDFramework3（二进制未嵌入）的，在主工程HDFrameworkDemo中需要有HDFramework2，同时link HDFramework3; 项目打包后结构：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk8.jpg) 

主工程HDFrameworkDemo需要 link  HDFramework3：

![](http://7xqhx8.com1.z0.glb.clouddn.com/baidusdk9.jpg) 



ruby库xcodeproj使用心得

[http://www.jianshu.com/p/cca701e1d87c](http://www.jianshu.com/p/cca701e1d87c)