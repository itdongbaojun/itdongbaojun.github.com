---
layout: post
title: "利用Octopress写Blog的常用命令"
date: 2014-11-23 22:01:10 +0800
comments: true
categories: 写作工具
---
### 前言
关于介绍如何搭建基于github的博客的优秀文章有很多，如果你想从零开始搭建一个基于github的博客系统，你可以看一下`唐巧`这篇Blog[象写程序一样写博客：搭建基于github的博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)，介绍的很详细、很清楚。今天在这里只是简单的记录一下搭建博客系统成功后发布博客的简单的流程。

### 创建
Octopress为我们提供了一些task来创建博文和页面。博文必须存储在`source/_posts`目录下，并且需要按照Jekyll的命名规范对文章进行命名：`YYYY-MM-DD-post-title.markdown`。文章的名字会被当做url的一部分，而其中的日期用于对博文的区分和排序。<!-- more -->

通过Octopress提供的task可以正确的按照命名规范创建一个博文，并且在博文中会附带常用的一些yaml元数据。命令如下：

```c++
$ rake new_post["title"]
```

其中title为博文的文件名，创建出来的文件默认是markdown格式。这个文件在`source/_posts/xxx`路径下，打开文件可以看到如下的内容:
```
---
layout: post
title: "title"
date: 2014-11-23 22:01:10 +0800
comments: true
categories: 
---
```
### 内容
接下来就可以在上面生成的那个文件中写博客啦！这里有一点需要说明的就是，你可以在文章合适的地方插入`<!-- more -->`，这样就可以实现在文章的预览首页不显示全部内容，会有一个“Continue →”按钮来查查看整篇文章。

### 生成和预览
可以使用下面的命令生成静态文件

```
$ rake generate
```

可以使用下面的命令检测文件变化，实时生成新内容

```
$ rake watch
```
可以使用下面的命令生成在本机4000端口访问的内容

```
$ rake preview		# 访问http://localhost:4000就可以
```
### 发布
当你把文章内容写好了，同时本地预览效果也没有问题了，这个时候就可以发布文章了，如下：

```
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
$ rake deploy				#发布
```
### 总结
是不是很简单，这样就搞定了，哈哈!
