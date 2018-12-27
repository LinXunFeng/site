---
title: iOS - Swift UICollectionView横向分页的问题
date: 2017-09-12 08:53:55
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

有两种方式可以解决，数据只有11个，要分两页需要16个，那我们可以直接添加数据到16个，然后在dataSource中返回cell时进行判断及处理即可。不过对于现在来说太小题大做了，我选第二种方式~

+<!-- more -->

<The rest of contents | 余下全文>

# UICollectionView横向分页的问题
## 情况
直接看图
![滚前](/images/2017/09/iOS - Swift UICollectionView横向分页的问题/1.png)
![滚后](/images/2017/09/iOS - Swift UICollectionView横向分页的问题/2.png)
已经设置collectionView的isPagingEnabled为true了，可是出现了这种情况，原因就是collectionView的contentSize不够。

```
<UICollectionView: 0x7fc565076000; 
frame = (0 0; 375 197); 
clipsToBounds = YES; 
gestureRecognizers = <NSArray: 0x6180000557e0>; 
layer = <CALayer: 0x61000022a5a0>; 
contentOffset: {187.5, 0}; 
contentSize: {562.5, 192.25}
>
```
## 解决方案
有两种方式可以解决，数据只有11个，要分两页需要16个，那我们可以直接添加数据到16个，然后在dataSource中返回cell时进行判断及处理即可。不过对于现在来说太小题大做了，我选第二种方式~
### 直接修改contentSize
我自定义了一个继承于UICollectionViewFlowLayout的Layout(LXFChatMoreCollectionLayout)，让UICollectionView在创建的时候使用了它

在 LXFChatMoreCollectionLayout.swift 中我们需要重写父类的collectionViewContentSize，将contentSize取出来修改为我们自己创建的newSize就可以了代码如下
```swift
override var collectionViewContentSize: CGSize {
    let size: CGSize = super.collectionViewContentSize
    let collectionViewWidth: CGFloat = self.collectionView!.frame.size.width
    let nbOfScreen: Int = Int(ceil(size.width / collectionViewWidth))
    let newSize: CGSize = CGSize(width: collectionViewWidth * CGFloat(nbOfScreen), height: size.height)
    return newSize
}
```
注：ceil函数的作用是求不小于给定实数的最小整数。ceil(2)=ceil(1.2)=cei(1.5)=2.00
### 效果
![](/images/2017/09/iOS - Swift UICollectionView横向分页的问题/3.png)

至于如何让item水平布局，请参考[《iOS - Swift UICollectionView横向分页滚动，cell左右排版》](/2017/09/12/[iOS - Swift UICollectionView横向分页滚动，cell左右排版](http://linxunfeng.top/2017/09/12/iOS-Swift-UICollectionView横向分页滚动，cell左右排版/)

附上相关项目：[Swift 3.0 高仿微信](https://github.com/LinXunFeng/LXFWeChat)