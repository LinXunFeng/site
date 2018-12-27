---
title: iOS-Swift面向协议编程（二）
date: 2017-09-12 09:32:49
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

上一篇文章[iOS - Swift 面向协议编程（一）
](/2017/09/12/iOS-Swift-面向协议编程（一）/)已经对Swift的面向协议编程做了介绍，接下来该篇文章将使用面向协议开发(POP)来做下实际的应用

+<!-- more -->

<The rest of contents | 余下全文>

> 上一篇文章[iOS - Swift 面向协议编程（一）
> ](/2017/09/12/iOS-Swift-面向协议编程（一）/)已经对Swift的面向协议编程做了介绍，接下来该篇文章将使用面向协议开发(POP)来做下实际的应用

在实际开发中，自定义View基本上是必须的，相信这对我们来说都是比较简单，不过我们还是来回顾一下下~

## 面向对象开发

1 新建一个UIView的FirstTypeView
![FirstTypeView](http://linxunfeng.github.io/images/2017/09/iOS-Swift面向协议编程（二）/1.png)

2 创建一个View的xib文件
![xib](http://linxunfeng.github.io/images/2017/09/iOS-Swift面向协议编程（二）/2.png)
3 设置xib对应的class进行绑定

![xib class](http://linxunfeng.github.io/images/2017/09/iOS-Swift面向协议编程（二）/3.png)

4 在FirstTypeView.swift 中实现一个类方法，方便我们外部用xib来初始化FirstTypeView
```swift
import UIKit

class FirstTypeView: UIView {
    
}

extension FirstTypeView {
    class func loadFromNib() -> FirstTypeView {
        return Bundle.main.loadNibNamed("\(self)", owner: nil, options: nil)?.first as! FirstTypeView
    }
}
```
在外部只要调用FirstTypeView的loadFromNib方法就可以初始化一个View来使用了。好，现在又有一个类SecondTypeView，也是要求使用xib来初始化view。这时我们就会想，一样的加载xib的方法，那我们就把它抽取出来放到父类就可以了。这里的父类以BaseView.swift为例
父类的主要实现代码
```swift
extension BaseView {
    class func loadFromNib() -> BaseView {
        return Bundle.main.loadNibNamed("\(self)", owner: nil, options: nil)?.first as! BaseView
    }
}
```
那我们的FirstTypeView和SecondTypeView只需要直接继承于BaseView就可以了，在其它地方初始化view也很方便
```
let firstView = FirstTypeView.loadFromNib()
view.addSubview(firstView)

let secondView = SecondTypeView.loadFromNib()
view.addSubview(secondView)
```
好，现在FirstTypeView里声明了一个属性name，SecondTypeView声明的属性为age，假如我们现在要使用各自对应的属性，这时是直接点不出来的，需要先进行强转
```swift
let firstView = FirstTypeView.loadFromNib() as! FirstTypeView
firstView.name = "LXF"
view.addSubview(firstView)

let secondView = SecondTypeView.loadFromNib() as! SecondTypeView
secondView.age = 100
view.addSubview(secondView)
```

但是这样觉得不是很方便，还需要进行强转，我们能不能在BaseView里面搞定它呢？如果是Swift 2.x 的话是可以的
```
extension BaseView {
    class func loadFromNib() -> Self { // 注意这里是大写的S
        return Bundle.main.loadNibNamed("\(self)", owner: nil, options: nil)?.first as! Self
    }
}
```
但是现在Swift 3.0已经不支持这种写法了，会报错。这个时候如果使用面向协议的开发就很方便了。

## 面向协议开发
> 将BaseView删除，FirstTypeView和SecondTypeView改回继承于UIView

1 新建一个Swift文件 Nibloadable.swift
![Nibloadable](http://linxunfeng.github.io/images/2017/09/iOS-Swift面向协议编程（二）/4.png)

2 实现协议方法

**协议中不允许定义类方法，需改为静态方法**
```
import UIKit

protocol Nibloadable {
    
}

extension Nibloadable {
    static func loadFromNib() -> Self {
        return Bundle.main.loadNibNamed("\(self)", owner: nil, options: nil)?.first as! Self
    }
}
```

3 遵守协议
```
class SecondTypeView: UIView, Nibloadable {
    var age: Int = 10
}
```

这样就可以了，而且你在调用loadFromNib方法时可以发现，类型是对应上的

![loadFromNib协议方法](http://linxunfeng.github.io/images/2017/09/iOS-Swift面向协议编程（二）/5.png)

好了，面向协议开发的应用就记录到这里，希望能帮助到大家！

[Demo](https://github.com/LinXunFeng/LXFPOP)