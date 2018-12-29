---
title: iOS-给高仿微信添加直播聊天功能
date: 2017-09-12 09:47:36
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

抽空给之前做的开源项目[【高仿微信】](https://github.com/LinXunFeng/LXFWeChat)添加直播功能，由于时间有限，做得不是很完美，有空再去完善吧，能用就好~~

+<!-- more -->

<The rest of contents | 余下全文>

> 抽空给之前做的开源项目[【高仿微信】](https://github.com/LinXunFeng/LXFWeChat)添加直播功能，由于时间有限，做得不是很完美，有空再去完善吧，能用就好~~

在此提供存放于百度云的完整项目[【高仿微信】- 百度云](https://pan.baidu.com/s/1bpB55Bx)
希望各位能在我的GitHub上献出一个宝贵的Star [【高仿微信】- GitHub](https://github.com/LinXunFeng/LXFWeChat)
谢谢


注意：直播功能的使用（对方需要先进入到对应的聊天界面）

> 两个测试账号： lxf lqr  密码都是123456

![直播聊天](https://github.com/LinXunFeng/LXFWeChat/raw/master/Screenshots/8.gif)

## 推流
首先第一件事当然就是搭建一个推流服务器，这里请跳转参考我之前写好的文章吧[【Ubuntu安装nginx来搭建推流服务器】](/2017/09/12/Ubuntu-安装nginx-来搭建推流服务器/)，这里我的服务器的ip地址是：192.168.123.191

APP上推流我使用的是第三方的库 [LFLiveKit](https://github.com/LaiFengiOS/LFLiveKit)，这个第三方库已经帮我们处理了很多事情，而且还包括美颜~~。当然，有时间我们还是要去了解一下底层的东西，这里就先不赘述，过几天抽空再做总结。

关键代码如下

```swift
// 初始化配置
let audioConfiguration = LFLiveAudioConfiguration.default()
let videoConfiguration = LFLiveVideoConfiguration.defaultConfiguration(for: .low2, outputImageOrientation: .portrait)
// 初始化session
let session = LFLiveSession(audioConfiguration: audioConfiguration, videoConfiguration: videoConfiguration)
// 设置代理
// session?.delegate = self
// 设置展示的View
session?.preView = self.view
```

```swift
let stream = LFLiveStreamInfo()
stream.url = "rtmp://192.168.123.191:1935/rtmplive/lxf"; // 服务器地址
session.startLive(stream)
// 开始推流
session.running = true
```

## 拉流
这里我使用的是B站的开源库 [ijkplayer](https://github.com/Bilibili/ijkplayer) 

为了方便可以用这个 [编译好的B站开源库](https://github.com/LinXunFeng/IJKFramework)

需要注意的是：IJKPlayer默认使用的是软解码(FFMpeng)，如果需要使用硬解码需要我们进行相应的设置
```swift
// 设置"videotoolbox"的值为0为软解码(默认)，设置为1则是硬解码
let options = IJKFFOptions.byDefault()
options?.setOptionIntValue(1, forKey: "videotoolbox", of: kIJKFFOptionCategoryPlayer)

let ijkPlayer = IJKFFMoviePlayerController(contentURLString: "rtmp://192.168.123.191:1935/rtmplive/lxf", with: options)
// 需保存起来
self.ijkPlayer = ijkPlayer
ijkPlayer?.view.frame = view.bounds
view.addSubview(ijkPlayer!.view)

// 准备播放，当视频准备好的时候会自动进行播放
ijkPlayer?.prepareToPlay()
```

## 将IJKPlayer打包

从B站的gitHub上下载的 [ijkplayer](https://github.com/Bilibili/ijkplayer) 需要手动编译出来，跟着说明走就可以了，这里就不赘述咯，接下来我们将它打包，方便使用

如果你不跟着说明走的话会提示找不到 avformat.h 这个头文件
![avformat.h](http://http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/1.png)

这时你需要在终端cd到ijkplayer这个目录，然后执行 init-ios.sh文件，如图

![目录](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/2.png)

![init-ios.sh](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/3.png)

然后经过一段漫长的时间之后，在ios目录下就多出了这些ffmpeg相关的目录

![ffmpeg相关目录](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/4.png)

这个操作是在下载ffmpeg源码，然缺失的avformat.h就在里面

![avformat.h](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/5.png)


打开项目 IJKMediaPlayer
![打开项目](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/6.png)

设置为 release，这样打出来的包会小些
![Edit Scheme](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/7.png)

![release](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/8.png)

选择真机和模拟器，各Command+B编译一次
![真机](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/9.png)
![模拟器](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/10.png)

右击，Show in Finder
![](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/11.png)

如图，就有两个文件夹，里面存放着的就是我们编译出来的库
![Paste_Image.png](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/12.png)

可以使用如下命令查看信息
```shell 
lipo -info IJKMediaFramework
```
![查看所支持的处理器](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/13.png)
默认模拟器编译出来的包是不支持i386，如果希望支持的话
进入项目的 Build Settings，将 Build Active Architecture Only 设置为NO
![Build Active Architecture Only](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/14.png)

好，现在对编译出来的包进行合并，这样就即支持真机，也支持模拟器
```shell
// 格式
// lipo -create  path1  path2  -output  frameName

lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
```

![合并](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/15.png)

将合并出来的IJKMediaFramework替换 IJKMediaFramework.framework中的IJKMediaFramework，最后将替换好的 IJKMediaFramework.framework 拖入到项目中使用即可。
![替换](http://linxunfeng.github.io/images/2017/09/iOS-给高仿微信添加直播聊天功能/16.png)

最后，附上编译好的IJKMediaFramework
链接:https://pan.baidu.com/s/1eRYlJ7W 密码:9iaw



<div class="github-widget" data-repo="LinXunFeng/LXFWeChat"></div>