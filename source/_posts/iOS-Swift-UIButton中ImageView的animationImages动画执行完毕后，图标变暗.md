---
title: iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗
date: 2017-09-12 09:22:32
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

查看官方文档的说明 当该属性设置为true时，按钮在高亮状态下，图片会绘制成高亮（前提是没有手动设置高亮图片）。并且该值的默认值为true
所以我们也可以将属性adjustsImageWhenHighlighted设置为false，不让系统自动帮我们设置高亮状态下显示的图片即可。

+<!-- more -->

<The rest of contents | 余下全文>

## 情况
![变暗](http://linxunfeng.github.io/images/2017/09/iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗/1.gif)
贴出重要代码

```swift
// 设置按钮图片动画数组
voiceButton.imageView?.animationImages = [
    #imageLiteral(resourceName: "message_voice_sender_playing_1"),
    #imageLiteral(resourceName: "message_voice_sender_playing_2"),
    #imageLiteral(resourceName: "message_voice_sender_playing_3")
]
```
```swift
// 开始动画
voiceButton.imageView?.startAnimating()
```
```swift
// 停止动画
voiceButton.imageView?.stopAnimating()
```
## 原因
这个按钮在结束动画之后之所以会变暗，是因为它在动画结束之后自动显示为高亮图片，不信？那只好上证据了~
```swift
// 设置语音按钮的高亮图片
voiceButton.setImage(#imageLiteral(resourceName: "message_voice_sender_normal"), for: .highlighted)
```
![](http://linxunfeng.github.io/images/2017/09/iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗/2.gif)

那知道原因之后就很好解决了
## 解决方案
### 方案一：设置按钮的高亮图片
将按钮的高亮图片与普通状态下的一致即可。这里就再赘述了
### 方案二：adjustsImageWhenHighlighted = false
在UIButton中有这么一个属性
```swift
adjustsImageWhenHighlighted
```
查看官方文档的说明
![](http://linxunfeng.github.io/images/2017/09/iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗/3.png)
当该属性设置为true时，按钮在高亮状态下，图片会绘制成高亮（前提是没有手动设置高亮图片）。并且该值的默认值为true
所以我们也可以将属性adjustsImageWhenHighlighted设置为false，不让系统自动帮我们设置高亮状态下显示的图片即可。

## 效果
![完美](http://linxunfeng.github.io/images/2017/09/iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗/4.gif)

附上相关项目：[Swift 3.0 高仿微信](https://github.com/LinXunFeng/LXFWeChat)

<div class="github-widget" data-repo="LinXunFeng/LXFWeChat"></div>