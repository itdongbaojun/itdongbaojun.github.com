---
layout: post
title: "采用现代Objective-C"
date: 2014-03-18 14:00:10 +0800
comments: true
categories: iOS
---
### 前言

这些年以来，随着*Objective-C*语言的不断发展和进化。尽管这门语言的核心概念和实践都保持不变，但是这其中的一部分已经有了显著的提升和改变。为了让我们更加容易的写出正确的代码，现代*Objective-C*语言提升了类型安全、内存管理、性能以及一些其他的方面。因此，在我们现在的以及将来的项目中采用这些新的改变让我们的代码变得更加一致、更加可读、更加可维护。

*Xcode*提供了一个工具来帮助我们做出这些结构上的改变。但是在使用这个工具之前，我们需要知道*Xcode*对我们的代码做出了什么样改变，以及它为什么要做出这样的改变。这篇文章会突出介绍如何在我们的代码中采用一些最有效、最有用的现代*Objective-C*编程方法。<!-- more -->

### instancetype

使用关键字`instancetype`作为需要返回一个类(*或者这个类的子类*)实例的方法的返回值类型。这些方法包括`alloc`，`init`，以及类工厂方法。

在合适的场合使用`instancetype`代替`id`类型来提高你的Objective-C代码的类型安全。例如，看一下下面的代码:

```c++
// .h
@interface MyObject : NSObject
+ (instancetype)factoryMethodA;
+ (id)factoryMethodB;
@end
// .m
@implementation MyObject
+ (instancetype)factoryMethodA { return [[[self class] alloc] init]; }
+ (id)factoryMethodB { return [[[self class] alloc] init]; }
@end
// 调用
void doSomething() {
    NSUInteger x, y;
    x = [[MyObject factoryMethodA] count]; // Return type of +factoryMethodA is taken to be "MyObject *"
    y = [[MyObject factoryMethodB] count]; // Return type of +factoryMethodB is "id"
}
```
由于类方法`+factoryMethodA`的返回值类型是`instancetype`，这个类型的消息表现为`MyObject *`。由于`MyObject`没有`-count`方法，编译器报了一个关于`x`行的警告：

```
main.m: ’MyObject’ may not respond to ‘count’
```
然而，类方法`+factoryMethodB`的返回值值类型为`id`类型，编译器就不会报关于`y`行的警告啦。因为一个`id`类型的对象可以是任何类，同时，一个叫做`-count`的方法总是存在于一些类的某个地方，因此，类方法`+factoryMethodB`的返回值来实现这个方法对于编译器来说是可能的。

为了使`instancetype`类工厂方法能够拥有正确的子类化行为，确保我们在分配类时使用`[self class]`而不是直接使用类名。遵循这个约定以确保编译器能够正确的推断出子类的类型。例如：子类化上一个例子中的`MyObject`:

```
@interface MyObjectSubclass : MyObject
@end
void doSomethingElse() {
        NSString *aString = [MyObjectSubclass factoryMethodA];
}
```
对于这段代码编译器给出了如下的警告：

```
main.m: Incompatible pointer types initializing ’NSString *’ with an expression of type ’MyObjectSubclass *’
```
在这个例子中，`+factoryMethodA`消息发送后返回了一个`MyObjectSubclass`类型的对象，是消息接收者类型的对象。编译器因此会相应的认为`+factoryMethodA`方法的返回值应该是`MyObjectSubclass`的子类，而不是其工厂方法中声明的那个父类。

### 如何采用？

在你的代码中，用`instancetype`类型恰当的代替出现`id`返回值类型的地方。比较典型的就是`init`方法和类工厂方法。尽管编译器会自动的将以“alloc”，“init”，“new”开头的方法的返回值类型由`id`转为`instancetype`，但是却不会为其他的方法进行自动转换。Objective-C的约定是为所有的方法明确的使用`instancetype`返回值类型。

请注意，你应该只是对于返回值类型用`instancetype`替换`id`，在其他处的代码是不可以的。与`id`类型不同的是，关键字`instancetype`只能用于方法声明中的返回值类型。

例如：

```
@interface MyObject
- (id)myFactoryMethod;
@end
```
应该转变成：

```
@interface MyObject
- (instancetype)myFactoryMethod;
@end
```

### Properties

Objective-C的*property*是使用`@property`语法声明的公有方法或者私有方法。

```
@property (readonly, getter=isBlue) BOOL blue;
```
`Properties`捕获对象的状态。它能够表达出对象的内在的属性以及和其他对象之间的关系。`Properties`提供了一个安全、方便的方式与这些属性进行交互而不用自己写一组自定义的存取方法（尽管属性允许我们在需要的时候自定义`getters`和`setters`）。

在越来越多的场合下使用属性代替实例变量有以下的益处：

* **自动合成`getters`和`setters`。**当你声明一个`property`，会默认自动为你创建`getter`和`setter`方法。
* **意图表达更加清楚的一组方法。**由于存取方法的命名规范，能够非常准确的表达`getter`和`setter`方法正在做什么。
* **`Property`关键字表达表达了一些关于行为的额外信息。**属性为声明像`assign`(vs `copy`)，`weak`，`atomic`（vs `nonatomic`）等行为提供了可能。

属性方法遵循一个简单的命名规范。*getter*方法名为属性的名字（例如：`date`），*setter*方法名位属性名带上*set*前缀，采用驼峰写法（例如：`setDate`）。布尔值属性的命名约定是的*getter*方法是以“is”开头的：

```
@property (readonly, getter=isBlue) BOOL blue;
```
因此，下面的每一个方法都是能够工作的：

```
if (color.blue) { }
if (color.isBlue) { }
if ([color isBlue]) { }
```
当想要决定什么可以是`property`时，一定要记住，下面的这些情况不能为属性：

* `init`方法
* `copy`方法，`mutableCopy`方法
* 类工厂方法
* 方法启动了一个操作，同时返回一个`Bool`值
* 方法明确改变了内部状态作为一个getter的副作用。

此外，在你的代码中识别潜在的属性时，需要考虑下面的这些原则：

* 一个read/write property 拥有两个存取方法。setter方法拥有一个参数，无返回值，getter方法没有参数，有一个返回值。如果你想转变这组方法称为property，需要标注为`readwrite`关键字。
* 一个read-only property只有一个存取方法，就是getter方法，无参数，有一个返回值。如果想要转变这个方法为property时，需要标注`readonly`关键字。
* getter方法应该是*idempotent*（如果getter方法被调用两次，第二次调用的结果和第一次调用的结果应该是相同的）。然而，对于getter方法是合理的去计算它每一次被调用的值。

### 如何采用？

识别出一组方法，使其有资格转化成一个property，比如：

```
- (NSColor *)backgroundColor;
- (void)setBackgroundColor:(NSColor *)color;
```
使用`@property`语法选择合理的关键字来声明成属性：

```
@property (copy) NSColor *backgroundColor;
```

### Enumeration Macros

`NS_ENUM`和`NS_OPTIONS`宏提供了一个方便、简单的方式来定义基于**C**语言的枚举和选择。这些宏在Xcode中改善了代码实现，明确指定枚举、选择的类型以及大小。除此之外，这种声明枚举的方式可以很好的兼容旧的的编译器，新的声明方法可以解释基础类型的信息。

使用`NS_ENUM`宏来定义*enumerations*，一组不同的值的集合：

```
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
};
```
`NS_ENUM`宏帮助定义枚举的名字和类型，`UITableViewCellStyle`宏的类型为`NSInteger`。枚举的类型应该是`NSInteger`。

使用`NS_OPTIONS`宏来定义*options*，一组结合在一起的位掩码：

```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
像枚举一样，`NS_OPTIONS`宏定义了名字和类型。然而，*options*的类型通常是`NSUInteger`。

### 如何采用？

替代你的`enum`声明，比如：

```
enum {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
};
typedef NSInteger UITableViewCellStyle;
```
使用`NS_ENUM`语法：

```
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
};
```
但是，当你使用`enum`来定义一个位掩码(bitmask)，比如：

```
enum {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
typedef NSUInteger UIViewAutoresizing;
```
使用`NS_OPTIONS`宏：

```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

### 自动引用计数（ARC）

自动引用引用计数（ARC）是编译器特性，为Objective-C对象提供内存管理，不再需要记住什么时候应该使用`retain`，`release`，和`autorelease`。ARC帮你管理对象的生存周期，在编译器帮你自动插入适当的内存管理代码，编译器也会在合适的时机帮你生成`dealloc`方法。

### 如何采用？

Xcode提供了ARC自动转换工具。选择ARC迁移工具：选择：**Edit** > **Refactor** > **Convert to Objective-C ARC**。更多信息请查看：[Transitioning to ARC Release Notes](https://developer.apple.com/library/ios/releasenotes/objectivec/rn-transitioningtoarc/introduction/introduction.html)

### 相关链接

[Adopting Modern Objective-C](https://developer.apple.com/library/ios/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html?utm_campaign=iOS_Dev_Weekly_Issue_137&utm_medium=email&utm_source=iOS%2BDev%2BWeekly#//apple_ref/doc/uid/TP40014150)