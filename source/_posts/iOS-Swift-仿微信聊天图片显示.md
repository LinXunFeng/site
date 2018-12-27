---
title: iOS - Swift 仿微信聊天图片显示
date: 2017-09-12 08:57:35
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

其实原理比较简单，准备一张图片MaskImgae，先对其进行拉伸，然后按照其轮廓对图片进行裁剪就行了

+<!-- more -->

<The rest of contents | 余下全文>

## 效果图

如图所示，图片左侧有个小箭头
![效果图](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/1.png)

## 原理
其实原理比较简单，准备一张图片MaskImgae，先对其进行拉伸，然后按照其轮廓对图片进行裁剪就行了
![MaskImgae](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/2.png)

## 步骤
这里摘重点说，布局什么的按自己意愿去弄吧。我固定了图片的显示大小为 102 * 152
### 1、对MaskImgae进行拉伸
```swift
// 设置拉伸范围
let stretchInsets = UIEdgeInsetsMake(30, 28, 23, 28)
// 待拉伸的图片
let stretchImage = UIImage(named: "SenderImageNodeMask")
// 进行拉伸
let bubbleMaskImage = stretchImage.resizableImage(withCapInsets: stretchInsets, resizingMode: .stretch)
```
拉伸的效果如图
![拉伸效果](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/3.png)

### 2、对imageView设置裁剪区域
这里我的 imageView 叫   chatImgView
上面的拉伸效果图是临时把拉伸好的图片赋值给了chatImgView，只是为了给大家看到效果而已，各位看官如果有赋值请记得改回来~~

**好，下面进行裁剪**
```swift
// 新建一个图层
let layer = CALayer()
// 设置图层显示的内容为拉伸过的MaskImgae
layer.contents = bubbleMaskImage.cgImage
// 设置拉伸范围(注意：这里contentsCenter的CGRect是比例（不是绝对坐标）)
layer.contentsCenter = self.CGRectCenterRectForResizableImage(bubbleMaskImage)
// 设置图层大小与chatImgView相同
layer.frame = CGRect(x: 0, y: 0, width: 102, height: 152)
// 设置比例
layer.contentsScale = UIScreen.main.scale
// 设置不透明度
layer.opacity = 1
// 设置裁剪范围
self.chatImgView.layer.mask = layer
// 设置裁剪掉超出的区域
self.chatImgView.layer.masksToBounds = true
```
```swift
func CGRectCenterRectForResizableImage(_ image: UIImage) -> CGRect {
    // LXFLog("\(image.capInsets)")
    // 这里的image.capInsets就是UIEdgeInsetsMake(30, 28, 23, 28)
    return CGRect(
        x: image.capInsets.left / image.size.width,
        y: image.capInsets.top / image.size.height,
        width: (image.size.width - image.capInsets.right - image.capInsets.left) / image.size.width,
        height: (image.size.height - image.capInsets.bottom - image.capInsets.top) / image.size.height
    )
}
```
这样就完成了
## 解释一下下
### UIEdgeInsetsMake
MaskImgae 的大小为 56 * 50
```swift
// UIEdgeInsetsMake(top: CGFloat, left: CGFloat, bottom: CGFloat, right: CGFloat)
UIEdgeInsetsMake(30, 28, 23, 28)
```
红色范围就是要拉伸的范围(随手一扣，不太准确，意思意思下就好了~~)
![拉伸区域](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/4.png)

### contentsCenter
这是对某个区域进行全面拉伸，如果不设置的话默认值为
```swift
CGRect(x: 0, y: 0, width: 1, height: 1)
```
就是直接进行缩放
那我们先来看看，如果不对contentsCenter这个值进行设置会是什么效果
![直接拉伸](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/5.png)
我们来看下官方解释

```swift
var contentsCenter: CGRect { get set }
Description	
The rectangle that defines how the layer contents are scaled
if the layer’s contents are resized. Animatable.
```
翻译：如果图层的内容是重新设置了尺寸的，那定义的这个矩形(contentsCenter)是为了告诉图层，图层的内容是如何被缩放的

那明了，我们的图片是被拉伸后再绘制到layer上的，为了正确显示我们的图片，我们得告诉layer它是怎么被进行拉伸的。是的，就是下面代码所指定的范围
```swift
UIEdgeInsetsMake(30, 28, 23, 28)
```
但是，正如上面提到过的，contentsCenter所要赋值的CGRect是比例，不是绝对坐标，所以现在我们得通过(30, 28, 23, 28)获取比例值，转换方法已经在上面给出了，就是CGRectCenterRectForResizableImage
我们来打印下 image.capInsets的内容
```swift
LXFLog("\(image.capInsets)")
LXFLog("\(image.capInsets.top)")
LXFLog("\(image.capInsets.bottom)")
LXFLog("\(image.capInsets.left)")
LXFLog("\(image.capInsets.right)")
```
打印结果
```
UIEdgeInsets(top: 30.0, left: 28.0, bottom: 23.0, right: 28.0)
30.0
23.0
28.0
28.0
```
好，现在结合 下面的图 与 CGRectCenterRectForResizableImage 方法中的代码就很明确比例是怎么取到的了
![拉伸区域](linxunfeng.github.io/images/2017/09/iOS - Swift 仿微信聊天图片显示/6.png)

附上相关项目：[Swift 3.0 高仿微信](https://github.com/LinXunFeng/LXFWeChat)



<div class="github-widget" data-repo="LinXunFeng/LXFWeChat"></div>