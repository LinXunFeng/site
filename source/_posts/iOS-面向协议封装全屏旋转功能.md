---
title: iOS 面向协议封装全屏旋转功能
date: 2018-09-15 18:05:45
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

> 关于使用面向协议来封装功能的实战可以参考我上篇文章 [【iOS 面向协议方式封装空白页功能】](/2018/04/07/iOS-面向协议方式封装空白页功能/)，这里就不再赘述，我们直接进入使用阶段吧。
> 本篇文章只有一个目的，那就是只要遵守协议，一行代码随意切换全屏～

如果对面向协议有疑问的同学可以看下我之前的两篇文章

[iOS - Swift 面向协议编程（一）](/2017/09/12/iOS-Swift-面向协议编程（一）/) 

[iOS - Swift 面向协议编程（二）](/2017/09/12/iOS-Swift-面向协议编程（二）/)

+<!-- more -->

<The rest of contents | 余下全文>

> 关于使用面向协议来封装功能的实战可以参考我上篇文章 [【iOS 面向协议方式封装空白页功能】](/2018/04/07/iOS-面向协议方式封装空白页功能/)，这里就不再赘述，我们直接进入使用阶段吧。
> 本篇文章只有一个目的，那就是只要遵守协议，一行代码随意切换全屏～

如果对面向协议有疑问的同学可以看下我之前的两篇文章

[iOS - Swift 面向协议编程（一）](/2017/09/12/iOS-Swift-面向协议编程（一）/) 

[iOS - Swift 面向协议编程（二）](/2017/09/12/iOS-Swift-面向协议编程（二）/)



## 开源库
| Name      | Link                                                         |
| --------- | ------------------------------------------------------------ |
| GitHub    | [LXFProtocolTool](https://github.com/LinXunFeng/LXFProtocolTool) |
| Wiki      | [Wiki首页](https://github.com/LinXunFeng/LXFProtocolTool/wiki) |
| 本文 Demo | [LXFFullScreenable](https://github.com/LinXunFeng/LXFProtocolTool/tree/master/Example/LXFProtocolTool/Demo/LXFFullScreenable) |

使用Cocoapods的方式来安装即可

```ruby
pod 'LXFProtocolTool/FullScreenable'
```

## 一、配置

在AppDelegate中实现如下方法

```swift
func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
    return UIApplication.shared.lxf.currentVcOrientationMask
}
```

## 二、使用案例

> 方法与属性的调用都需要命名空间加上 `lxf`，如`isFullScreen` -> `lxf.isFullScreen`

```swift
isFullScreen : 获取当前遵守协议者是否为全屏状态
```

```swift
func switchFullScreen(
    isEnter: Bool? = nil,
    specifiedView: UIView? = nil,
    superView: UIView? = nil,
    config: FullScreenableConfig? = nil,
    completed: ((_ isFullScreen: Bool)->Void)? = nil
)
```
| Name          | Type                              | Desc                                |
| ------------- | --------------------------------- | ----------------------------------- |
| isEnter       | `Bool?`                           | 是否进入全屏                        |
| specifiedView | `UIView?`                         | 指定即将全屏的视图                  |
| superView     | `UIView?`                         | 作为退出全屏后specifiedView的父视图 |
| config        | `FullScreenableConfig?`           | 配置                                |
| completed     | `((_ isFullScreen: Bool)->Void)?` | 进入/退出 全屏后的回调              |

> 当`switchFullScreen`的调用者为`UIView`时，如果`specifiedView`为`nil`会自动填写，`superView`也是如此

> `switchFullScreen`方法不推荐直接使用，不过当遵守协议者为`UIViewController`时，可以通过使用默认参数来切换屏幕方向`lxf.switchFullScreen()`



![lxf_FullScreenable_1](/images/2018/09/iOS 面向协议封装全屏旋转功能/lxf_FullScreenable_1.gif)




以下分两种情况说明

### UIViewController

```swift
func enterFullScreen(
    specifiedView: UIView,
    config: FullScreenableConfig? = nil,
    completed: FullScreenableCompleteType? = nil
)
```

```swift
func exitFullScreen(
    superView: UIView,
    config: FullScreenableConfig? = nil,
    completed: FullScreenableCompleteType? = nil
)
```

> 以上两个方法是对`switchFullScreen`的抽离，使调用时对参数的传递更加清晰

1、遵守协议 `FullScreenable`

```swift
class LXFFullScreenableController: UIViewController, FullScreenable { }
```

2、指定视图进入全屏
```swift
lxf.enterFullScreen(specifiedView: cyanView)
```

3、指定视图退出全屏，并添加到当前控制器的`view`上
```swift
lxf.exitFullScreen(superView: self.view)
```

#### 🔥自动进入|退出全屏

```swift
func autoFullScreen(
    specifiedView: UIView,
    superView: UIView,
    config: FullScreenableConfig? = nil
) 
```
- 控制器可以调用该方法来注册自动进入或退出全屏，各控制器之间互不影响。
- `view`手动进入全屏会屏蔽当前控制器的自动全屏功能，退出方可恢复



### UIView

```swift
func enterFullScreen(
    specifiedView: UIView? = nil,
    config: FullScreenableConfig? = nil,
    completed: FullScreenableCompleteType? = nil
)
```

```swift
func exitFullScreen(
    superView: UIView? = nil,
    config: FullScreenableConfig? = nil,
    completed: FullScreenableCompleteType? = nil
)
```
> 以上两个方法是对`switchFullScreen`的抽离，使调用时对参数的传递更加清晰

1、遵守协议 `FullScreenable`

```swift
class LXFFullScreenView: UIButton, FullScreenable { }
```

```swift
let cyanView = LXFFullScreenView()
```

2、进入全屏

```swift
cyanView.lxf.enterFullScreen()
```

3、退出全屏
```swift
cyanView.lxf.exitFullScreen()
```

> 这里是对遵守了`FullScreenable`协议的视图进入全屏切换，由于代码内部已经经过自动视图填写，所以直接调用相应的方法即可，当然也可以自己指定`specifiedView`和`superView`



![lxf_FullScreenable_2](/images/2018/09/iOS 面向协议封装全屏旋转功能/lxf_FullScreenable_2.gif)



## 三、FullScreenableConfig说明
> 上述的方法都有一个`config`参数，默认为nil，即为默认配置

相关属性说明
| Name                       | Type                     | Desc                           | Default        |
| -------------------------- | ------------------------ | ------------------------------ | -------------- |
| animateDuration            | `Double`                 | 进入/退出 全屏时的旋转动画时间 | 0.25           |
| enterFullScreenOrientation | `UIInterfaceOrientation` | 进入全屏时的初始方向           | landscapeRight |

这里我们把动画时间设置为`1s`，初始方向为`左`后来看看效果
```swift
FullScreenableConfig(
    animateDuration: 1,
    enterFullScreenOrientation : .landscapeLeft
)
```
```swift
cyanView.lxf.enterFullScreen(config: diyConfig)
cyanView.lxf.exitFullScreen(config: diyConfig)
```



![lxf_FullScreenable_3](/images/2018/09/iOS 面向协议封装全屏旋转功能/lxf_FullScreenable_3.gif)



## 结语
到这里相关的说明已罗列完毕，有什么不清楚的可以下载[Demo](https://github.com/LinXunFeng/LXFProtocolTool/tree/master/Example/LXFProtocolTool/Demo/LXFFullScreenable)看看，或者在文章下方留言提问

[LXFProtocolTool](https://github.com/LinXunFeng/LXFProtocolTool) 主要是通过协议的方式来方便快捷地实现一些的实用功能，除了本文提及的全屏旋转功能外还有其它实用功能的封装，具体内容可以到 [Wiki首页](https://github.com/LinXunFeng/LXFProtocolTool/wiki)  查找。如果你有什么想实现的功能也可以提出来，喜欢的就给个Star鼓励下我吧 🚀 🚀 🚀，感谢支持！