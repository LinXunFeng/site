---
title: iOS - 实现UINavigation全屏动返回(一)
date: 2017-09-11 21:53:59
categories: "iOS"
tags:
  - iOS
  - Objective-C
  - Swift
---
<Excerpt in index | 首页摘要> 
interactivePopGestureRecognizer 是UINavigationController自带手势，当我们自定义了导航条的返回按钮后，这个手势就自动失效了，也就是说无法滑动返回。
+<!-- more -->
<The rest of contents | 余下全文>

## 要点
interactivePopGestureRecognizer 是UINavigationController自带手势，当我们自定义了导航条的返回按钮后，这个手势就自动失效了，也就是说无法滑动返回。

## 条件
很多情况下我们不得不自定义导航条的返回按钮，但是我们也要滑动返回上一级的效果。

## 思路
既然自动失效，那我们就告诉它什么时候生效。
- 在非根控制器下生效(用于滑动返回上一级)
- 在根控制器下失效(防止根控制器被移除，当然系统不会让我们把它移除，只是会出现bug)
  ![苹果官方文档说明](https://linxunfeng.github.io/images/2017/09/iOS-实现UINavigation全屏动返回-一/1.png)翻译：第一个被添加的控制器成为永远不会被出栈的根控制器

## 步骤
自定义一个 UINavigationController ，即继承于 UINavigationController ，名字为 LXFNavigationController ，将代理设为自己，遵守协议 UINavigationControllerDelegate ，实现代理方法 navigationController:didShowViewController:animated:

## 代码
LXFNavigationController.m

```objc
/** 系统手势代理 */
@property(nonatomic, strong) id popGesture;
```
```objc
- (void)viewDidLoad {
 [super viewDidLoad];

  // 记录系统手势代理
  self.popGesture = self.interactivePopGestureRecognizer;
  self.delegate = self;
}
```
```objc
#pragma mark - UINavigationControllerDelegate
// 当控制器显示完毕的时候调用
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated {
    // 根据 栈 先进后出
    if (self.viewControllers[0] == viewController) { // 根控制器
        // 还原代理
        self.interactivePopGestureRecognizer.delegate = self.popGesture;
    } else { // 非控制器
        // 清空手势代理就能实现滑动返回，iOS6不支持
        self.interactivePopGestureRecognizer.delegate = nil;
    }
    // 如果当前控制器为根控制器，则使手势失效，不然手势会将根控制器移除
    if (self.viewControllers.count == 1) {
        self.interactivePopGestureRecognizer.enabled = NO;
    } else {
        self.interactivePopGestureRecognizer.enabled = YES;
    }
}
```
![只有左侧边缘滑动才有效](https://linxunfeng.github.io/images/2017/09/iOS-实现UINavigation全屏动返回-一/2.gif)



[附上Demo](https://github.com/LinXunFeng/LXFNavigationControllerDemo)

### 最后说两句
这样就可以了，但是注意了，现在实现的是滑动返回功能，并没有全屏滑动返回~~接下来看下一篇吧
 [iOS - 实现UINavigation全屏滑动返回(二)](/2017/09/12/iOS-实现UINavigation全屏动返回-二/)



<div class="github-widget" data-repo="LinXunFeng/LXFNavigationControllerDemo"></div>