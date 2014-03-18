---
layout: post
title: "采用现代Objective-C"
date: 2014-03-18 11:47:48 +0800
comments: true
published: true
categories:iOS 
---
### 前言
这些年以来，随着Objective-C语言的不断发展和进化。尽管这门语言的核心概念和实践都保持不变，但是这其中的一部分已经有了显著的提升和改变。为了让我们更加容易的写出正确的代码，现代Objective-C语言提升了类型安全、内存管理、性能以及一些其他的方面。因此，在我们现在的以及将来的项目中采用这些新的改变让我们的代码变得更加一致、更加可读、更加可维护。

Xcode提供了一个工具来帮助我们做出这些结构上的改变。但是在使用这个工具之前，我们需要知道Xcode对我们的代码做出了什么样改变，以及它为什么要做出这样的改变。这篇文章会突出介绍如何在我们的代码中采用一些最有效、最有用的现代Objective-C编程方法。

### instancetype
使用关键字`instancetype`作为需要返回一个类(*或者这个类的子类*)实例的方法的返回值类型。这些方法包括`alloc`，`init`，以及类工厂方法。

在合适的场合使用`instancetype`代替`id`类型来提高你的Objective-C代码的类型安全。例如，看一下下面的代码：

```c++
@interface MyObject : NSObject
+ (instancetype)factoryMethodA;
+ (id)factoryMethodB;
@end

@implementation MyObject
+ (instancetype)factoryMethodA { return [[[self class] alloc] init]; }
+ (id)factoryMethodB { return [[[self class] alloc] init]; }
@end

void doSomething() {
    NSUInteger x, y;
 
    x = [[MyObject factoryMethodA] count]; // Return type of +factoryMethodA is taken to be "MyObject *"
    y = [[MyObject factoryMethodB] count]; // Return type of +factoryMethodB is "id"
}
```