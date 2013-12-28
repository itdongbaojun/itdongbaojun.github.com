---
layout: post
title: "使用CocoaPods来管理iOS项目的依赖库"
date: 2013-12-28 17:17:25 +0800
comments: true
categories: iOS开发工具
---
![CocoaPods Logo](/images/blogImgs/cocoapods-logo.png)

### 前言
细细算来，我接触iOS已经有*1.5f*年的时间了，虽然其中有差不多一年的时间是在大四经历*自学和实习*的这个阶段。抛去那段时间不算，毕业后在现在的公司工作差不多半年了...    

在经历过的几个项目上基本上每一个都会用到第三方开源库，比如`SDWebImage`、`AFNetworking`、`MBProgressHUD`等。然而，每次把这些第三方库导入到我们的项目中要配置一些选项以及添加第三方库本身依赖的系统框架，这个工作是重复的而且非常没有技术含量。有没有什么工具能替代我们做这些工作呢？         

一直到最近几天才知道，业界早已有了为iOS项目提供依赖管理的工具(*我深深的感觉到我还没入行就先落伍了*)，这个工具就是：[CocoaPods](http://beta.cocoapods.org)。 
 
### CocoaPods简介

CocoaPods是一个负责管理iOS项目中第三方开源库的工具。CocoaPods的[项目源码](https://github.com/CocoaPods/CocoaPods)在Github上管理。该项目开始于2011年8月12日，在这两年多的时间里，它持续保持活跃更新。开发iOS项目不可避免地要使用第三方开源库，CocoaPods的出现使得我们可以节省设置和更新第三方开源库的时间 

在我们有了CocoaPods这个工具之后，只需要将用到的第三方开源库放到一个名为Podfile的文件中，然后在命令行执行`$ pod install`命令。CocoaPods就会自动将这些第三方开源库的源码下载下来，并且为我的工程设置好相应的系统依赖和编译参数     

### CocoaPods的安装及使用

### 安装

安装的方式非常简单，Mac下已经自带了ruby，只要使用ruby的gem命令就可以安装了。打开的Mac的终端，在终端运行下面的命令： 

```c++
$ [sudo] gem install cocoapods
$ pod setup
``` 
说明：执行`$ pod setup`这步可能比较慢，需要多等待一段时间，也可能是我网络的问题 

### 更新

当然我们也可以更新我们的CocoaPods，同样也是使用ruby的gem命令： 

```
$ [sudo] gem update cocoapods
``` 
然而你也可以更新CocoaPods的预览版，执行下面的命令：

```
$ [sudo] gem update cocoapods --pre
```  
### 查找第三方库

如果我们不知道cocoaPods管理的库中，是否有你想要的库，那么你可以通过`$ pod search xxx`命令进行查找，以下是我用`$ pod search sdwebimage`查找到的所有可用的库： 

```
-> SDWebImage (3.5.1)
   Asynchronous image downloader with cache support with an UIImageView
   category.
   pod 'SDWebImage', '~> 3.5.1'
   - Homepage: https://github.com/rs/SDWebImage
   - Source:   https://github.com/rs/SDWebImage.git
   - Versions: 3.5.1, 3.5, 3.4, 3.3, 3.2, 3.1, 3.0, 2.7.4, 2.7, 2.6, 2.5, 2.4
   [master repo]
   - Sub specs:
     - SDWebImage/Core (3.5.1)
     - SDWebImage/MapKit (3.5.1)
     - SDWebImage/WebP (3.5.1)
``` 
注：我省略了两个库，没有全列出。

### 使用

假设我的Desktop上有一个已经存在的一个项目名称叫做：*CocoaPodsTest*，首先，进入项目的根目录，并在根目录下创建一个名叫Podfile的文件（没有任何后缀）： 
 
```
$ cd Desktop/CocoaPodsTest/		'进入项目根目录，根据自己项目实际目录'
$ vim Podfile	 '创建Podfile文件，你可以选择你自己喜欢的编辑器'
```  
注：vim的简单用法，`$ vim fileName`创建文件fileName，并打开；按`i`进入插入模式，输入文本；按`esc`进入命令模式后，按`:wq`或`ZZ`退出并保存。

然后，在Podfile文件中按以下的格式将依赖库的名字列出： 

```
platform :ios, '6.0'				'平台、版本'
pod 'SDWebImage', '~> 3.5.1'		'开源库名称、版本'
pod 'AFNetworking', '~> 2.0.3'		'开源库名称、版本' 
``` 
保存Podfile文件后，执行如下安装的命令: 

```
$ pod install
``` 
当安装命令执行成功后，会输出： 

```
Analyzing dependencies
Downloading dependencies
Installing AFNetworking (2.0.3)
Installing SDWebImage (3.5.1)
Generating Pods project
Integrating client project
[!] From now on use `CocoaPodsTest.xcworkspace`.
``` 
哈哈，看到类似这样的输出就是成功了。你所需要的第三方开源库都下载好了，并且设置好了相应的依赖以及编译参数。在我们以后用的时候一定要记住以下两点： 

**1. 最后一行是一个警告，提醒我们需要注意：从现在开始，需要通过`xxx.xcworkspace`打开的我们的项目。而不是之前我们一直用的`xxx.xcodeproj`** 

**2. 当我们每次修改了`Podfile`这个文件后，一定要记得执行命令：`$ pod install`，还可以执行`$ pod update`来更新类库** 

### 总结

用CocoaPods给我们的iOS项目添加依赖库真的太方便了，几个命令就搞定了，我个人建议像我一样还不会使用CocoaPods进行项目依赖的初级开发者，尤其是像我这样刚毕业的本科生，这个工具有必要学会，不能被鄙视，更能提高效率

有很多iOS大牛早已写了关于cocoaPods的相关教程，我个人又参考各大牛的博客写了一遍，只为能增加使用CocoaPods的熟练度。如有造成侵权行为，请联系本人

### 参考资料 

[唐巧的技术博客](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)
 
[CocoaPods Blog](http://blog.cocoapods.org)
 
[CocoaPods安装和使用教程](http://code4app.com/article/cocoapods-install-usage)