
---
title: 创建cocoapods私有库
---

#### 摘要
CocoaPods是iOS开发 下的一个第三方库管理工具。你能并轻松管理三方库版本，避免开发者对三方库的各种各样的配置，简化开发过程中无意义的重复工作

#### 一、从CocoaPods的使用说起
* 创建Podfile
我们在使用CocoaPods是都会在项目目录下创建一个Podfile文件,然后把需要的库加进文件中
> $ pod init

![MacDown logo](https://github.com/yandeguangzy/yandeguangzy.github.io/resources/image/cocoapods文档/init.png)

* 安装三方库

> $ pod install

或

> $ pod install --no-repo-update

安装的过程可以分为以下几个步骤
> * 查看 ~/.cocoapods/repo/master/Specs 是否存在将要安装的库
> * 若存在，从这个本地三方库信息库中获取 Podfile 中对应三方库的下载，若不存在，拉取三方库信息库到 ~/.cocoapods/repo/中，并从远端拉取Podfile中对应的三方库源码至/Users/kw/Library/Caches/CocoaPods/Pods,然后拷贝至当前目录。

#### 二、使用跟踪，CocoaPods干了什么
![MacDown logo](https://github.com/yandeguangzy/yandeguangzy.github.io/resources/image/install.png)

* 阅读Podfile文件,并解决依赖关系
* 与沙盒的文件进行比较
* 下载源码至沙盒，并缓存至对应的Pod文件夹下
* 生成.xcodeproj


#### 三、挑灯细看，CocoaPods核心组件
##### 1、 若干ruby写的gem包

CocoaPods是用ruby写的，并划分成了若干个Gem包。CocoaPods在解析执行过程中最重要的几个包的路径分别是：[CocoaPods/CocoaPods](https://github.com/CocoaPods/CocoaPods)、 [CocoaPods/Core](https://github.com/CocoaPods/Core) 和 [CocoaPods/Xcodeproj](https://github.com/CocoaPods/Xcodeproj)。
	
* CocoaPods / CocoaPod这是面向用户的组件，每当你执行一个pod命令时，这个组件将被激活。它包括了所有使用CocoaPods的功能，并且还能调用其他gem包来执行任务。
* CocoaPods / Core gem提供了与CocoaPods相关的文件（主要是Podfile和podspecs）的处理。
* CocoaPods / Xcodeproj 这个包负责工程文件直接关系的处理。它能创建以及修改.xcodeproj文件和.xcworkspace文件。它也可以作为一个独立的包使用。

##### 2、 [PodSpec](https://github.com/CocoaPods/Specs)

PodSpec是CocoaPods能用起来的另外一个重要的组成部分，它是CocoaPods维护的所有三方库的存储地址，存储的都是.podspec文件。.podspec文件描述了一个三方库所需的源码地址、依赖库、等所有信息。

在本机/Users/kw/.cocoapods/repos/master/Specs缓存了所有的开源的三方库的.podspec文件.

#### 四、实践求真，实现一个自己的私有库

了解了CocoaPods的基本实现思路之后，我们来实现一个自己得私有库。
创建一个文件夹，把想作为私有库的源码放在这个文件夹中，或者可以建一个工程，引用源码。

##### 1、创建私有库.podspec文件，修改并校验
在创建的文件夹下执行

> $pod spec create [name]
![MacDown logo](https://github.com/yandeguangzy/yandeguangzy.github.io/resources/image/podspec.png)

这个文件中包含了私有库的所有配置信息，
包括支持的iOS系统(s.platform)、依赖的系统库(s.frameworks)、依赖的其他CocoaPods库(s.dependency)、暴露的头文件(s.public_header_files)、源文件(s.source)、多重文件夹(s.subspec)等。
![MacDown logo](https://github.com/yandeguangzy/yandeguangzy.github.io/resources/image/podspec注释.png)

修改完成之后用以下命令校验
> $pod lib lint

##### 2、创建自己的spec库

* 在远端创建一个仓库用来维护私有的podspec文件，执行以下命令关联在本地

> $pod repo add Myspecs https://github.com/yandeguangzy/myspec.git

执行成功后，在/Users/kw/.cocoapods/repos路径下能看到Myspecs文件夹。

##### 3、提交源码及podspec

* 提交源代码。文件校验完成之后说明我们的源码及podspec文件没有问题，把需要作为源码的git版本打上一个tag，tag值与podspec文件中的s.source后缀的tag值一致（建议与s.version值也一致），并提交至远端。当然你也可以尝试一下在s.source后缀加上用你提交的commit。
*提交podspec文件。执行以下命令

> $pod repo push Myspecs [name].podspec

执行完成之后在本地/Users/kw/.cocoapods/repos目录下就能看到Myspec的文件夹，里面就是刚提交的podspec文件

##### 4、本地测试、远端测试
* 本地测试

>	pod 'LBKit', :path => '/Users/kw/Desktop/zyr/lbkit' #指定路径
***
>	pod 'LBKit', :podspec => '/Users/kw/.cocoapods/repos/Myspecs/LBKit/0.0.4/LBKit.podspec'  #指定podspec文件
*** 
>  pod 'LBKit', '~> 0.0.1'   #指定版本

***
>注：指定版本时主要标记远端git仓库
![MacDown logo](https://github.com/yandeguangzy/yandeguangzy.github.io/resources/image/source.png)
 
#### 五、黑科技的小确幸
我们了解完了CocoaPods的实现原理，并完成了一个私有库的创建之后，再回过头来看，我们发现，CocoaPods就是通过Podfile中的指定路径或指定podspec文件或指定git或指定版本去分析去下载路径及各方面的信息，也就是我们只要解决了podsepc文件，其他一切问题都可以迎刃而解，既然私有库的podspec文件也是通过自己的仓库去管理，那我们可以想想：

* 1、试试把podsepc文件通过其他的方式比如git，按CocosPads的格式提交至远端，是不是也可以成功pod至项目中？
* 2、甚至我们是不是都不用去维护远端的spec仓库，在使用时直接用指定git的方式是不是也能实现呢？

>	pod 'LBKit', :git =>'https://github.com/yandeguangzy/myspec.git'  #指定git

让我们通过CocoaPods愉快地玩耍吧！

参考文章：
>http://www.cocoachina.com/ios/20150228/11206.html
>
>http://blog.jobbole.com/53365/
>
>https://www.jianshu.com/p/c17cee5e9c7f
