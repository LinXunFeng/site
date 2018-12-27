---
title: iOS - ARC与MRC的单例设计模式
date: 2017-09-12 09:25:47
categories: "iOS"
tags:
  - iOS
  - Objective-C
---

<Excerpt in index | 首页摘要> 

单例设计模式(Singleton) 就是保证某个类创建出来的对象从始到终只有一个的一种方案

+<!-- more -->

<The rest of contents | 余下全文>

## 单例设计模式(Singleton)
### 定义
就是保证某个类创建出来的对象从始到终只有一个的一种方案
### 作用
- 节省内存开销
- 保证整个程序中使用同一份资源

### 实现
首先将我们的环境设置为非ARC环境，即MRC，如图
![MRC环境](/images/2017/09/iOS - ARC与MRC的单例设计模式/1.png)

> 在MRC模式下，我们得自己手动释放资源，所以得重写一些与资源创建与释放相关的方法，以保证单例对象的唯一。

新建一个继承于NSObject的类 LXFFileTool，我直接上代码，并写上注释
LXFFileTool.h
```objc
@interface LXFFileTool : NSObject
+ (instancetype)sharedFileTool;
@end
```
LXFFileTool.m
```objc
#import "LXFFileTool.h"
@implementation LXFFileTool
static LXFFileTool *_fileTools = nil;
/**
 *  alloc方法内部会调用allocWithZone:
 *  @param zone 系统分配给app的内存
 */
+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    if (_fileTools == nil) {
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{    // 安全(这个代码只会被调用一次)
            _fileTools = [super allocWithZone:zone];
        });
    }
    return _fileTools;
}
- (oneway void)release {
    // 在allocWithZone中使用了GCD令创建对象的代码只执行一次，如果_fileTools被释放则无法再创建
    // 重写release方法，防止_fileTools被释放
}
// 重写retain方法
- (instancetype)retain {
    return self;
}
// 重写retainCount锁定引用计数
- (NSUInteger)retainCount {
    return 1;
}
// 重写init方法，防止单例所拥有的属性值被重置
// 让初始化的方法只能执行一次，自然属性值就没有机会被重置
- (instancetype)init {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _fileTools = [super init];  // init会先调用alloc方法
    });
    return _fileTools;
}
// 仿造系统的单例创建方式，提供类方法
+ (instancetype)sharedFileTool {
    // 由于我们已经重写了init方法保证了单例对象的唯一了，所以这里直接调用init方法即可。
    return [[self alloc] init];
}
@end
```
> MRC下就是这样，我们的目的就是只能创建和初始化一次对象，不给机会释放，也不给机会重新初始化，从而保证了该对象的唯一。

那现在来看看ARC下是如何实现单例的吧。其实ARC下与MRC的区别就是ARC下我们不用自己再手动去释放资源了，从而使代码上大同小异，如下所示。
```objc
#import "LXFFileTool.h"
@implementation LXFFileTool

static LXFFileTool *_fileTools = nil;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    if (_fileTools == nil) {
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            _fileTools = [super allocWithZone:zone];
        });
    }
    return _fileTools;
}
- (instancetype)init {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _fileTools = [super init];
    });
    return _fileTools;
}
+ (instancetype)sharedFileTool {
    return [[self alloc] init];
}
```
## 批量创建单例
> 现在我们已经知道了ARC与MRC下分别是如何创建单例的了，但是如果我们一个项目里需要多个单例，那我们只能把代码复制粘贴再改改就完事吗？这未免也太麻烦了吧。那我们能不能做到快速且方便的创建单例对象呢？可以的，利用宏

首先先说下一些关于宏的知识吧
- 使用 #define 关键字来定义宏
- 宏定义只能是单行的，不能换行

那现在来讨论下一些疑惑吧，你说宏只能单行，可是创建单例的代码可是有很多行呀！还有我们如何做到自定义类方法名(就是 sharedXXX )？好，我们来介绍下宏下的两个特殊符号

### 宏的特殊符号

|     |            作用            |
| :--: | :-----------------------: |
|  \   |      用来转译换行符，即屏蔽换行符       |
|  ##  | 将两个相邻的标记(token)连接为一个单独的标记 |

想了解其它关于宏的预处理命令可以自行百度参考"C语言的预处理命令"

> 简单来说，\用于取消换行，##用来连接，而我们就用##来实现自定义类方法名

创建一个头文件Singleton.h用来存放宏定义
先来看看定义.h中 sharedXXX 是如何通过宏来定义的
```objc
// .h文件的实现
#define SingletonH(methodName) + (instancetype)shared##methodName;
```
现在回到LXFFileTool.h中，直接一行定义sharedFileTool这个类方法
```objc
#import "Singleton.h"
@interface LXFFileTool : NSObject
SingletonH(FileTool)
@end
```
我们只需要将方法名FileTool传入SingletonH()中就可以拼接为sharedFileTool

那现在再来看看定义.m中创建单例的方式，以ARC为例
```objc
#define SingletonM(methodName) \
static id _instance = nil; \
+ (instancetype)allocWithZone:(struct _NSZone *)zAone { \
    if (_instance == nil) { \
        static dispatch_once_t onceToken; \
        dispatch_once(&onceToken, ^{ \
            _instance = [super allocWithZone:zone]; \
        }); \
    } \
    return _instance; \
} \
\
- (instancetype)init { \
    static dispatch_once_t onceToken; \
    dispatch_once(&onceToken, ^{ \
        _instance = [super init]; \
    }); \
    return _instance; \
} \
\
+ (instancetype)shared##methodName { \
    return [[self alloc] init]; \
}
```
> 在每一个行后面加上\(反斜杠)取消换行，使用##来拼接传入的方法名，但还有一点需要注意：最后一行不能加反斜杠

回到LXFFileTool.m中，一行实现创建单例
```objc
#import "LXFFileTool.h"
@implementation LXFFileTool
SingletonM(FileTool)
@end
```
好，现在还有一个问题，就是如果我的项目中有个别文件是需要MRC环境的，那我该怎么办才能让创建单例也是如此简单呢？很简单，加个判断就好了，大致判断如下，详情看文章最后附上的Demo
```objc
#if __has_feature(objc_arc) // ARC
// 写上ARC下的定义代码
#else   // 非ARC
// 写上MRC下的定义代码
#endif
```
>好了，现在用起来是不是方便多了？我们只要创建一个类，然后在.h文件中写SingletonH(XXX)，再在.m文件中写SingletonM(XXX)就可以实现单例了~

### 指定环境
顺便提下如何在MRC下指定某个类文件使用的环境为ARC
![指定环境](/images/2017/09/iOS - ARC与MRC的单例设计模式/2.png)
如图，可以在 Build Phases -> Compile Sources 中双击某个需要ARC环境的类文件，然后写上

```
-fobjc-arc
```
如果是指定MRC，则写上
```
-fno-objc-arc
```
## Demo
最后，附上Demo: [LXFSingleton](https://github.com/LinXunFeng/LXFSingleton)



<div class="github-widget" data-repo="LinXunFeng/LXFSingleton"></div>