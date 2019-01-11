---
title: iOS - 实现UINavigation全屏动返回(二)
date: 2017-09-12 01:05:03
categories: "iOS"
tags:
  - Objective-C
  - Swift
---



<Excerpt in index | 首页摘要> 

在 [iOS - 实现UINavigation全屏滑动返回(一)](/2017/09/11/iOS-实现UINavigation全屏动返回-一) 中我们实现了滑动返回的功能，但不是全屏滑动返回，得在左侧边缘轻扫才能滑动返回~UINavigationController自带的只能在边缘轻扫才能滑动返回，这用户体验是不好的，接下来实现全屏滑动返回!

+<!-- more -->
<The rest of contents | 余下全文>

## 回顾
在 [iOS - 实现UINavigation全屏滑动返回(一)](/2017/09/11/iOS-实现UINavigation全屏动返回-一/) 中我们实现了滑动返回的功能，但不是全屏滑动返回，得在左侧边缘轻扫才能滑动返回~UINavigationController自带的只能在边缘轻扫才能滑动返回，这用户体验是不好的，接下来实现全屏滑动返回!

## 思路
既然自带的滑动返回只能是在边缘，那我们能不能修改使它触摸范围变大甚至全屏呢？先来看下系统手势有没有提供属性或方法供我们使用
```objc
NSLog(@"%@", self.interactivePopGestureRecognizer);
```
打印信息：
```objc
/*
<UIScreenEdgePanGestureRecognizer: 0x7fd542611e20; state = Possible;
 delaysTouchesBegan = YES; view = <UILayoutContainerView 0x7fd542706300>; target= 
<(action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 
0x7fd542611ce0>)>>
*/
```
原来系统手势的类型为 UIScreenEdgePanGestureRecognizer ，转到定义，发现有一个属性
```objc
UIRectEdge edges
```
是个结构体
```objc
typedef NS_OPTIONS(NSUInteger, UIRectEdge) { 
   UIRectEdgeNone = 0,
   UIRectEdgeTop = 1 << 0,
   UIRectEdgeLeft = 1 << 1,
   UIRectEdgeBottom = 1 << 2,
   UIRectEdgeRight = 1 << 3,
   UIRectEdgeAll = UIRectEdgeTop | UIRectEdgeLeft | UIRectEdgeBottom | UIRectEdgeRight
}
```
只提供了这几样，都是边缘的，这样就只好另寻他路了。
既然没有提供方式给我们现实要求，那我们就自己添加一个拖动手势 UIPanGestureRecognizer来替它执行滑动返回功能。
```objc
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:target action:@selector(handleNavigationTransition:)];
[self.view addGestureRecognizer:pan];
```
添加一个拖动手势，让他执行系统手势的操作，调用handleNavigationTransition:方法（刚才打印的信息中可以得知），现在的问题就是target是谁？
我们可以看看UIScreenEdgePanGestureRecognizer中是否有线索呢？
```objc
UIScreenEdgePanGestureRecognizer *gest = self.interactivePopGestureRecognizer;
```
找了半天没找着，但是UIScreenEdgePanGestureRecognizer继承于UIPanGestureRecognizer，而UIPanGestureRecognizer又继承于UIGestureRecognizer，在UIGestureRecognizer提供的方法中我们可以推断出一定有target，而且还是强引用的私有属性！那我们就可以用OC强大的杀手锏KVC来得到这个属性，但是前提是我们得知道target所指属性是什么名字
参照我的另一篇文章：[iOS - 通过runtime获取某个类中所有的变量和方法](/2017/09/12/iOS-通过runtime获取某个类中所有的变量和方法)
```objc
// OC runtime 机制
// 只能动态获取当前类的成员属性，不能获取其子类，或者父类的属性
unsigned int count = 0;// 拷贝出所胡的成员变量列表
Ivar *ivars = class_copyIvarList([UIGestureRecognizer class], &count);
for (int i = 0; i<count; i++) {
   // 取出成员变量
   Ivar ivar = *(ivars + i); 

   // 打印成员变量名字
   NSLog(@"%s", ivar_getName(ivar)); 

   // 打印成员变量的数据类型
   NSLog(@"%s", ivar_getTypeEncoding(ivar));
}

 // 释放
 free(ivars);
```
在打印中我们找到了UIGestureRecognizer的私有属性 _targets，是个数组，而且只有一个元素，元素的类型如图所示
![target](https://linxunfeng.github.io/images/2017/09/iOS-实现UINavigation全屏动返回-二/1.png)
那就好办了，这样我们就可以得到target了
```objc
NSArray *targets = [gest valueForKeyPath:@"_targets"]; // 打印可以发现里面就一个元素
id target = [targets[0] valueForKeyPath:@"_target"];
```
这样我们就差不多实现全屏滑动返回的功能，但是有个bug
![向右滑动，接着点击Button](https://linxunfeng.github.io/images/2017/09/iOS-实现UINavigation全屏动返回-二/2.gif)
如图所示，在最后里回到根控制器界面后我再一次向右滑动，接着点击Button，它没有将FirstVC弹出，这就是传说中的bug，那我们现在在做的，就是在根控制器不让滑动返回生效，即禁用手势。
监听手势，遵守协议UIGestureRecognizerDelegate，实现代理方法

```objc
// 当当前控制器是是根控制器时不让移除当前控制器(换句话说就是禁止手势)
pan.delegate = self;
```
```objc
#pragma mark - UIGestureRecognizerDelegate// 当开始滑动时调用
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer {
   // 当为根控制器是不让移除当前控制器，非根控制器时允许移除
   NSLog(@"%ld", self.viewControllers.count);
   BOOL open = self.viewControllers.count > 1;
   return open;
}
```
## 最后说两句
这样就可以全屏滑动了，不过让我们来看看我们添加手势的习惯
```objc
UIPanGestureRecognizer *myPan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan)];
[self.view addGestureRecognizer:myPan];
myPan.delegate = self;
```
我们在添加手势时设置了target为self，而delegate也为self
那是不是可以推断出系统手势的delegate就是我们刚刚想要的target呢，答案是是的
```objc
id target = self.interactivePopGestureRecognizer.delegate;
```
所以我们的target就可以通过这种方式获得，不用KVC的方式
哦，最后别忘了禁用系统手势
```objc
// 禁止系统的手势
self.interactivePopGestureRecognizer.enabled = NO;
```
这样，我们就实现了全屏滑动返回的功能了~

## 源码
> Objective-C

记得遵守协议： UIGestureRecognizerDelegate
LXFNavigationController.m
```objc
- (void)viewDidLoad {
   [super viewDidLoad];
   // 系统的手势
   UIScreenEdgePanGestureRecognizer *gest = self.interactivePopGestureRecognizer;
   // target
   id target = self.interactivePopGestureRecognizer.delegate;
   // 禁止系统的手势 
   self.interactivePopGestureRecognizer.enabled = NO;
   UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:target action:@selector(handleNavigationTransition:)];
   [self.view addGestureRecognizer:pan]; 
   // 监听代理
   pan.delegate = self;
}
```
```objc
#pragma mark - UIGestureRecognizerDelegate
// 当开始滑动时调用
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer {
   // 当为根控制器是不让移除当前控制器，非根控制器时允许移除
   NSLog(@"%ld", self.viewControllers.count);
   BOOL open = self.viewControllers.count > 1;
   return open;
}
```

> Swift

LXFNavigationController.swift
```
override func viewDidLoad() {
    super.viewDidLoad()
    
    guard let targets = interactivePopGestureRecognizer!.value(forKey: "_targets") as? [NSObject] else { return }
    let targetObjc = targets.first
    let target = targetObjc?.value(forKey: "target")
    let action = Selector(("handleNavigationTransition:"))
    
    let panGes = UIPanGestureRecognizer(target: target, action: action)
    view.addGestureRecognizer(panGes)
}
```