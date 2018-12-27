---
title: Swift 优雅的适配大小
date: 2018-10-24 20:44:50
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

> 在日常开发中常常会对设备进行一定的适配，为了方便在多个项目里统一管理和使用，所以封装并开源了`SwiftyFitsize`这个库，可用于适配视图及字体大小，同时也支持 xib 和 storyboard

GitHub: [SwiftyFitsize](https://github.com/LinXunFeng/SwiftyFitsize)

+<!-- more -->

<The rest of contents | 余下全文>

> 在日常开发中常常会对设备进行一定的适配，为了方便在多个项目里统一管理和使用，所以封装并开源了`SwiftyFitsize`这个库，可用于适配视图及字体大小，同时也支持 xib 和 storyboard

GitHub: [SwiftyFitsize](https://github.com/LinXunFeng/SwiftyFitsize)

最终的效果如下图所示

![效果图](http://linxunfeng.github.io/images/2018/10/Swift-优雅的适配大小/exhibition.png)

## 安装 
使用Cocoapods安装，或手动拖入项目
```
pod 'SwiftyFitsize'
```

## 使用
`SwiftyFitsize`在默认状况下所使用的参照宽度为`iphone6`的`375`
如果设计图所选用设备的宽度与默认值不同，可以在`AppDelegate`下初始化所参照的宽度

```swift
SwiftyFitsize.reference(width: 414)
```
下面列出一些设备对应的分辨率，方便查找

| 设备          | 逻辑分辨率(point) | 设备分辨率(pixel) |
| ------------- | ----------------- | ----------------- |
| SE            | 320x568           | 640x1136          |
| 6(S)／7／8    | 375x667           | 750x1334          |
| 6(S)+／7+／8+ | 414x736           | 1080x1920         |
| X(S)          | 375x812           | 1125x2436         |
| XR            | 414x896           | 828x1792          |
| XS Max        | 414x896           | 1242x2688         |

使用也是非常方便的，只需要在`Number`、`UIFont`、`CGPoint`、`CGSize`、`UIEdgeInsetsMake`这些类型的值后面加上`~`即可
```swift
100~
UIFont.systemFont(ofSize: 14)~
CGPoint(x: 10, y: 10)~
CGSize(width: 100, height: 100)~
CGRect(x: 10, y: 10, width: 100, height: 100)~
UIEdgeInsetsMake(10, 10, 10, 10)~
```
##### xib / storyboard 字体适配

支持控件 `UILabel` `UIButton` `UITextView` `UITextField`



![xib-font](https://github.com/LinXunFeng/SwiftyFitsize/raw/master/Screenshots/xib-font.png)




##### xib / storyboard 约束适配



![xib-font](https://github.com/LinXunFeng/SwiftyFitsize/raw/master/Screenshots/xib-constraint.png)

##### 

注：`~`请不要相互嵌套使用，如

```swift
CGPoint(x: 10~, y: 10~)~
```