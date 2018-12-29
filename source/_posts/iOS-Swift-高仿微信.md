---
title: iOS - Swift 高仿微信
date: 2017-09-12 09:29:00
categories: "iOS"
tags:
  - iOS项目
  - Swift
---

<Excerpt in index | 首页摘要> 

从2016年12月份开始做到 2017年2月16，虽说这个项目看起来不大，但是锻炼人的地方还真是很多的，微信通讯录联系人的按拼音排序，自定义表情键盘中遇到的分页滚动，微信聊天界面的图片显示、语音信息的动画等。挑战着各种各样的需求，本人也将遇到的问题的解决方案做了记录并整理了一下，希望能给iOS程序员同胞们带来帮助。目前基本的聊天已完成，实现了一部分个人信息的修改，功能我会尽快完善的！

+<!-- more -->

<The rest of contents | 余下全文>

# LXFWeChat
Swift 3.0 高仿微信

> 两个测试账号： lxf lqr  密码都是123456 

## 源码地址
码云
http://git.oschina.net/LinXunFeng/LXFWeChat

GitHub
https://github.com/LinXunFeng/LXFWeChat

在此提供存放于百度云的完整项目[【高仿微信】- 百度云](https://pan.baidu.com/s/1bpB55Bx)
希望各位能在我的项目上献出一个宝贵的Star
谢谢


> 从2016年12月份开始做到现在，虽说这个项目看起来不大，但是锻炼人的地方还真是很多的，微信通讯录联系人的按拼音排序，自定义表情键盘中遇到的分页滚动，微信聊天界面的图片显示、语音信息的动画等。挑战着各种各样的需求，本人也将遇到的问题的解决方案做了记录并整理了一下，希望能给iOS程序员同胞们带来帮助。目前基本的聊天已完成，实现了一部分个人信息的修改，功能我会尽快完善的！

## 模仿微信的导航栏
在navigationBar底部添加一个添加了渐变层的view
```swift
let blurBackView = UIView()
blurBackView.frame = CGRect(x: 0, y: -20, width: kScreenW, height: 64)
let gradintLayer = CAGradientLayer()
gradintLayer.frame = CGRect(x: 0, y: 0, width: kScreenW, height: 64)
gradintLayer.colors = [
    UIColor.hexInt(0x040012).withAlphaComponent(0.76).cgColor,
    UIColor.hexInt(0x040012).withAlphaComponent(0.28).cgColor
]
gradintLayer.startPoint = CGPoint(x: 0, y: 0)
gradintLayer.endPoint = CGPoint(x: 0, y: 1.0)
blurBackView.layer.addSublayer(gradintLayer)
blurBackView.isUserInteractionEnabled = false
blurBackView.alpha = 0.5

// 设置导航栏样式
navigationBar.barStyle = .black
navigationBar.insertSubview(blurBackView, at: 0)
```

## 表情面板和更多面板
遇到的问题总结了一下，可以参考下以下总结的文章

[《iOS - Swift UICollectionView横向分页滚动，cell左右排版》](http://www.jianshu.com/p/18d7d0f5e3e2)

[《iOS-Swift-UICollectionView横向分页的问题》](http://www.jianshu.com/p/60da3b52d64c)

## 聊天界面
[《iOS-Swift 仿微信聊天图片显示》](http://www.jianshu.com/p/4c570cd79bd3)

[《iOS-Swift-UITableView的scrollToRow的"坑"》](http://www.jianshu.com/p/aa139463eb4b)

[《iOS-Swift-UIButton中ImageView的animationImages动画执行完毕后，图标变暗》](http://www.jianshu.com/p/412a2e23b5b6)


## 首页
[《iOS - Swift 仿微信小红点(无数字)》](http://www.jianshu.com/p/807cddad469a)


## 目前完成的功能
### 微信界面
1. 显示右上角的菜单
2. 显示最近联系人
3. 最近联系人信息未读数的显示

### 通讯录界面
1. 联系人的排序
2. 联系人总数显示

### 发现界面
1. 动态小红点的显示
2. 购物选项的链接跳转

### 我界面
1. 个人头像的上传与设置，及头像的保存
2. 我的二维码界面的显示及二维码的保存

### 聊天界面
1. 小视频的录制与发送
2. 小视频的播放
3. 聊天时间
4. 图片的发送与显示
5. 未发送成功的重发功能
6. 语音的录制与发送
7. 语音的播放动态效果

## 已知BUG
- 小视频和图片发送出去后不能立即更新显示缩略图
- 更换头像模拟器测试正常，真机无效。。

## 2017-07-24 更新
抽空出来添加了一个简单的直播功能（对方需要先进入到对应的聊天界面）
相关博文：[iOS-给高仿微信添加直播聊天功能](http://www.jianshu.com/p/022b9044decc)
需要用到[编译好的B站开源库ijkplayer](https://github.com/LinXunFeng/IJKFramework) ，由于打包好的文件太大，传不上来，所以需要各位去自己编译集成进去。


## 效果图
### 动态图
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/1.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/2.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/3.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/4.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/5.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/6.gif)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/7.gif)

### 静态图
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170206_1.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_1.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_2.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_3.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_4.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_5.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_6.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_7.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_8.png)
![](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/Snip20170214_9.png)

<div class="github-widget" data-repo="LinXunFeng/LXFWeChat"></div>