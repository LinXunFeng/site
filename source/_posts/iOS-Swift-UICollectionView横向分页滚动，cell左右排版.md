---
title: iOS - Swift UICollectionView横向分页滚动，cell左右排版
date: 2017-09-12 08:49:37
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

最近在做表情键盘时遇到一个问题，我用UICollectionView来布局表情，使用横向分页滚动，但在最后一页出现了如图所示的情况

+<!-- more -->

<The rest of contents | 余下全文>

## 情况
最近在做表情键盘时遇到一个问题，我用UICollectionView来布局表情，使用横向分页滚动，但在最后一页出现了如图所示的情况
![只显示一半](/images/2017/09/iOS-Swift-UICollectionView横向分页滚动，cell左右排版/1.png)

### 情况分析图
是的，现在的item分布就是这个鬼样子
![从上到下，从左到右](/images/2017/09/iOS-Swift-UICollectionView横向分页滚动，cell左右排版/2.jpeg)
现在想要做的，就是将现在这个鬼样子变成另外一种样子，如图
![从左到右，从上到下](/images/2017/09/iOS-Swift-UICollectionView横向分页滚动，cell左右排版/3.jpeg)
那怎么办？只好重新布局item了

## 解决方案
我是自定了一个Layout(LXFChatEmotionCollectionLayout)，让UICollectionView在创建的时候使用了它

在 LXFChatEmotionCollectionLayout.swift 中
### 添加一个属性来保存所有item的attributes
```swift
// 保存所有item的attributes
fileprivate var attributesArr: [UICollectionViewLayoutAttributes] = []
```
### 重新布局
```swift
// MARK:- 重新布局
override func prepare() {
    super.prepare()
    
    let itemWH: CGFloat = kScreenW / CGFloat(kEmotionCellNumberOfOneRow)
    
    // 设置itemSize
    itemSize = CGSize(width: itemWH, height: itemWH)
    minimumLineSpacing = 0
    minimumInteritemSpacing = 0
    scrollDirection = .horizontal
    
    // 设置collectionView属性
    collectionView?.isPagingEnabled = true
    collectionView?.showsHorizontalScrollIndicator = false
    collectionView?.showsVerticalScrollIndicator = true
    let insertMargin = (collectionView!.bounds.height - 3 * itemWH) * 0.5
    collectionView?.contentInset = UIEdgeInsetsMake(insertMargin, 0, insertMargin, 0)
    
    
    /// 重点在这里
    var page = 0
    let itemsCount = collectionView?.numberOfItems(inSection: 0) ?? 0
    for itemIndex in 0..<itemsCount {
        let indexPath = IndexPath(item: itemIndex, section: 0)
        let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
        
        page = itemIndex / (kEmotionCellNumberOfOneRow * kEmotionCellRow)
        // 通过一系列计算, 得到x, y值
        let x = itemSize.width * CGFloat(itemIndex % Int(kEmotionCellNumberOfOneRow)) + (CGFloat(page) * kScreenW)
        let y = itemSize.height * CGFloat((itemIndex - page * kEmotionCellRow * kEmotionCellNumberOfOneRow) / kEmotionCellNumberOfOneRow)
        
        attributes.frame = CGRect(x: x, y: y, width: itemSize.width, height: itemSize.height)
        // 把每一个新的属性保存起来
        attributesArr.append(attributes)
    }
}
```
### 返回所有当前可见的Attributes
```swift
override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
    var rectAttributes: [UICollectionViewLayoutAttributes] = []
    _ = attributesArr.map({
        if rect.contains($0.frame) {
            rectAttributes.append($0)
        }
    })
    return rectAttributes
}
```
## 大功告成
![](/images/2017/09/iOS-Swift-UICollectionView横向分页滚动，cell左右排版/4.gif)

## 完整代码
```swift
import UIKit

let kEmotionCellNumberOfOneRow = 8
let kEmotionCellRow = 3

class LXFChatEmotionCollectionLayout: UICollectionViewFlowLayout {
    // 保存所有item
    fileprivate var attributesArr: [UICollectionViewLayoutAttributes] = []
    
    // MARK:- 重新布局
    override func prepare() {
        super.prepare()
        
        let itemWH: CGFloat = kScreenW / CGFloat(kEmotionCellNumberOfOneRow)
        
        // 设置itemSize
        itemSize = CGSize(width: itemWH, height: itemWH)
        minimumLineSpacing = 0
        minimumInteritemSpacing = 0
        scrollDirection = .horizontal
        
        // 设置collectionView属性
        collectionView?.isPagingEnabled = true
        collectionView?.showsHorizontalScrollIndicator = false
        collectionView?.showsVerticalScrollIndicator = true
        let insertMargin = (collectionView!.bounds.height - 3 * itemWH) * 0.5
        collectionView?.contentInset = UIEdgeInsetsMake(insertMargin, 0, insertMargin, 0)
        
        var page = 0
        let itemsCount = collectionView?.numberOfItems(inSection: 0) ?? 0
        for itemIndex in 0..<itemsCount {
            let indexPath = IndexPath(item: itemIndex, section: 0)
            let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
            
            page = itemIndex / (kEmotionCellNumberOfOneRow * kEmotionCellRow)
            // 通过一系列计算, 得到x, y值
            let x = itemSize.width * CGFloat(itemIndex % Int(kEmotionCellNumberOfOneRow)) + (CGFloat(page) * kScreenW)
            let y = itemSize.height * CGFloat((itemIndex - page * kEmotionCellRow * kEmotionCellNumberOfOneRow) / kEmotionCellNumberOfOneRow)
            
            attributes.frame = CGRect(x: x, y: y, width: itemSize.width, height: itemSize.height)
            // 把每一个新的属性保存起来
            attributesArr.append(attributes)
        }
        
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        var rectAttributes: [UICollectionViewLayoutAttributes] = []
        _ = attributesArr.map({
            if rect.contains($0.frame) {
                rectAttributes.append($0)
            }
        })
        return rectAttributes
    }
    
}
```

附上相关项目：[Swift 3.0 高仿微信](https://github.com/LinXunFeng/LXFWeChat)

<div class="github-widget" data-repo="LinXunFeng/LXFWeChat"></div>