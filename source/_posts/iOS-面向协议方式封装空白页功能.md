---
title: iOS-面向协议方式封装空白页功能
date: 2018-04-07 22:21:46
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

为了良好的交互体验，相信大家在对待`scrollView`无数据时的提示页都会使用一些第三方来定制，最典型的就是使用[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet)。但是每个界面都写一堆与`DZNEmptyDataSetDelegate`，` DZNEmptyDataSetSource`相关的代码就不太好，那一般情况下自然的就会采用继承的方式来避免。而Swift除了可以面向对象编程，它还可以面向协议编程。那可不可以也用协议来解决情况呢？嘿嘿，这个可以有，那我们接下来就来试试怎么通过协议的方式来避免上述情况，并且实现一行代码添加空白页功能

+<!-- more -->

<The rest of contents | 余下全文>

> 为了良好的交互体验，相信大家在对待`scrollView`无数据时的提示页都会使用一些第三方来定制，最典型的就是使用[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet)。但是每个界面都写一堆与`DZNEmptyDataSetDelegate`，` DZNEmptyDataSetSource`相关的代码就不太好，那一般情况下自然的就会采用继承的方式来避免。而Swift除了可以面向对象编程，它还可以面向协议编程。那可不可以也用协议来解决情况呢？嘿嘿，这个可以有，那我们接下来就来试试怎么通过协议的方式来避免上述情况，并且实现一行代码添加空白页功能

## 前言

如果对面向协议有疑问的同学可以看下我之前的两篇文章

[iOS - Swift 面向协议编程（一）](/2017/09/12/iOS-Swift-面向协议编程（一）/) 

[iOS - Swift 面向协议编程（二）](/2017/09/12/iOS-Swift-面向协议编程（二）/)

之前的文章中提到了，协议除了起规范作用，还有别一个用处，就是赋予能力。我们现在的目的就是让目标控制器或者目标视图在遵守我们的协议后，就可以有实现空白页的功能。

## 一、基本实现

### 1、创建协议
```swift
// MARK:- 空视图占位协议
public protocol LXFEmptyDataSetable {
    
}
```

### 2、确定面向类
确定我们面向的类，一般`tableView`或者`collectionView`都是写在控制器里，那我们面向的类就规定为`UIViewController`，或许也有人写在`UIView`里，不过这里先按`UIViewController`来写吧
```swift
// MARK:- UIViewController - 空视图占位协议
public extension LXFEmptyDataSetable where Self : UIViewController {
    // 3、的实现的方法写在这里
}
```

### 3、定义功能方法
将`scrollView`传递进来，让我们定义的方法来暗地里做些操作
```swift
func lxf_EmptyDataSet(_ scrollView: UIScrollView) {
    scrollView.emptyDataSetDelegate = self
    scrollView.emptyDataSetSource = self
}
```

### 4、设置数据源和代理
在`3、定义功能方法`中将`delegate`和`source`设置为了`self` ，而协议是无法遵守再次遵守其它协议的，那让什么来遵守对应的协议呢？要明白这里的`self`指的是`UIViewController`,考虑到`UIView`的可能，这里我就让万物对象之父`NSObject`来遵守，并实现对应的数据源方法和代理方法
```swift
extension NSObject : DZNEmptyDataSetDelegate, DZNEmptyDataSetSource {
    public func image(forEmptyDataSet scrollView: UIScrollView!) -> UIImage! {
        // 返回提示图片
    }
    public func title(forEmptyDataSet scrollView: UIScrollView!) -> NSAttributedString! {
        // 设置富文本标题
    }
    public func verticalOffset(forEmptyDataSet scrollView: UIScrollView!) -> CGFloat {
        // 设置纵向偏移
    }
}
```

## 二、定制空白页

通过上述步骤后，只要让`UIViewController`遵守我们的协议，再调用一下`lxf_EmptyDataSet`方法就可以实现数据空白页了。但是，这样直接写死的方式很不好，有时候一些场景是需要我们做出定制的，那怎么实现定制呢？协议又不能有自己的变量来存放我们的定制。

<font color='red'>*这里先做出一个限定，我们要使用重载方法来完成该功能，实现即可高定制，又可使用默认定制。*</font>

回到刚刚的话题，使用UserDefaults来实现可以吗？可以，但是比较麻烦，因为UserDefaults是单例，整个进程共用这一份资源，如果你当前`controller`遵守了我们的协议`LXFEmptyDataSetable`并做出了定制，那么当下一个`controller`在遵守协议后使用了`默认定制`时，那你要怎么办？还要区分`scrollView`，那就得保存当前`scrollView`，在退出当前`controller`后还要把对应的东西置空。好咯好咯，那你说到底要怎么搞才最合适？

> 解决方案：拓展`UIScrollView`！！！有没有发现？，非常地恰巧，我们定义的方法`lxf_EmptyDataSet`需要外界将`UIScrollView`传递进来，在`DZNEmptyDataSet`的数据源方法和代理方法也有`scrollView`。那让`UIScrollView`来携带我们的定制就好啦。

### 1、定义定制相关的枚举
这里我定义了常用的定制相关的枚举
```swift
public enum LXFEmptyDataSetAttributeKeyType {
    /// 纵向偏移(-50)  CGFloat
    case verticalOffset
    /// 提示语(暂无数据)  String
    case tipStr
    /// 提示语的font(system15)  UIFont
    case tipFont
    /// 提示语颜色(D2D2D2)  UIColor
    case tipColor
    /// 提示图(LXFEmptyDataPic) UIImage
    case tipImage
    /// 允许滚动(true) Bool
    case allowScroll
}
```

### 2、拓展UIScrollView
为`UIScrollView`定义一个定制相关的属性字典
```swift
extension UIScrollView {
    private struct AssociatedKeys {
        static var lxf_emptyAttributeDict:[LXFEmptyDataSetAttributeKeyType : Any]?
    }
    /// 属性字典
    var lxf_emptyAttributeDict: [LXFEmptyDataSetAttributeKeyType : Any]? {
        get {
            return objc_getAssociatedObject(self, &AssociatedKeys.lxf_emptyAttributeDict) as? [LXFEmptyDataSetAttributeKeyType : Any]
        }
        set {
            objc_setAssociatedObject(self, &AssociatedKeys.lxf_emptyAttributeDict, newValue as [LXFEmptyDataSetAttributeKeyType : Any]?, objc_AssociationPolicy.OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}
```

### 3、完善lxf_EmptyDataSet方法
这里我们让外界通过闭包的方式来定制自己的空白页
```swift
// MARK:- UIViewController - 空视图占位协议
public extension LXFEmptyDataSetable where Self : UIViewController {
    func lxf_EmptyDataSet(_ scrollView: UIScrollView, attributeBlock: (()->([LXFEmptyDataSetAttributeKeyType : Any]))? = nil) {
        scrollView.lxf_emptyAttributeDict = attributeBlock != nil ? attributeBlock!() : nil
        scrollView.emptyDataSetDelegate = self
        scrollView.emptyDataSetSource = self
    }
}
```

### 4、使用定制属性字典
这里以返回提示图片的方法为例吧
```
public func image(forEmptyDataSet scrollView: UIScrollView!) -> UIImage! {
    guard let tipImg = scrollView.lxf_emptyAttributeDict?[.tipImage] as? UIImage else {
        return UIImage(named: "LXFEmptyDataPic")
    }
    return tipImg
}
```

### 5、外界的使用姿势

```swift
class LXFEmptyDemoController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        initUI()
    }
}

extension LXFEmptyDemoController: LXFEmptyDataSetable {
    fileprivate func initUI() {
        let tableView = UITableView()
        // ...
        
        // 高定制
        self.lxf_EmptyDataSet(tableView) { () -> ([LXFEmptyDataSetAttributeKeyType : Any]) in
            return [
                .tipStr:"哟哟哟",
                .verticalOffset:-150,
                .allowScroll: false
            ]
        }
        
        // 默认定制
        // self.lxf_EmptyDataSet(tableView)
    }
}
```

![大功告成](/images/2018/04/iOS-面向协议方式封装空白页功能/lxf_EmptyDataSet.png)

## 三、开源库

我对这个过程进行一次整理，并做成一个名为 [LXFProtocolTool](https://github.com/LinXunFeng/LXFProtocolTool) 的库并上传至gitHub。可以使用`Cocoapods`的方式来安装使用

```
pod 'LXFProtocolTool'
```

我也将 [iOS - Swift 面向协议编程（二）](http://linxunfeng.top/2017/09/12/iOS-Swift-面向协议编程（二）/) 中提及的通过协议便捷加载xib的功能也集成了进来。大家可以根据自己的需要在Podfile写明要安装的功能

- Xib加载

```
pod 'LXFProtocolTool/LXFNibloadable'
```

- 空白视图

```
pod 'LXFProtocolTool/LXFEmptyDataSetable'
```

创建这个库的目的是为了通过协议的方式来方便快捷地实现一些的实用功能，目前功能不多，不过往后会逐渐增加，或许你有什么想实现的功能也可以提出来，喜欢的就给个Star鼓励下我吧 🚀 🚀 🚀 
