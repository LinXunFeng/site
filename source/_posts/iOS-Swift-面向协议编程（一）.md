---
title: iOS - Swift 面向协议编程（一）
date: 2017-09-12 09:30:29
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

传统的面向对象开发思维方式是将类中实现的相似方法抽取出来，接着放入一个Base类，然后继承于Base类后各个类即可找拥有相同的方法，不用再一个个手动实现。
比如：一个Person类，一个Dog类，它们都拥有方法eat，那么就可以新建一个Animal类，将eat方法抽取出来放入其中，然后将Person类和Dog类都继承于Animal。
但是，如果现在又有一个Robot类，也需要拥有eat方法，而此时也将其继承于Animal的话显然是不合理的，于是我们就需要转换思维，面向协议开发~

+<!-- more -->

<The rest of contents | 余下全文>

> OC无法做到面向协议开发，而Swift可以，因为Swift可以做到协议方法的具体实现，而OC不行

## 面向对象开发

传统的面向对象开发思维方式是将类中实现的相似方法抽取出来，接着放入一个Base类，然后继承于Base类后各个类即可找拥有相同的方法，不用再一个个手动实现。
比如：一个Person类，一个Dog类，它们都拥有方法eat，那么就可以新建一个Animal类，将eat方法抽取出来放入其中，然后将Person类和Dog类都继承于Animal。
但是，如果现在又有一个Robot类，也需要拥有eat方法，而此时也将其继承于Animal的话显然是不合理的，于是我们就需要转换思维，面向协议开发~

## 面向协议开发

面向协议开发的核心是：** 模块化（组件化） **
我们先来回顾下协议的一般使用，新建一个Swift文件LXFProtocol.swift
```
import Foundation

protocol LXFProtocol {
    func eat()
}
```
我们的Person类遵守协议LXFProtocol，需要我们实现协议中的方法，如：
```
class Person: NSObject, LXFProtocol {
    func eat() {
        //
    }
}
```
那我们每个类都这样做的话跟直接复制粘贴代码并没什么不同~~
而开头已经提到一点：
> Swift可以做到协议方法的具体实现

那么现在，我们新建一个Swift文件Eatable.swift，以区分LXFProtocol.swift
Eatable.swift中的代码实现如下：
```
import Foundation

protocol Eatable {
    // 可声明变量
}

extension Eatable {
    func eat() {
        // 实现具体的功能
    }
}
```
有2个注意点
- protocol中可以声明变量，方便在协议方法中使用
- 协议方法的具体实现需要在extension中来实现

使Dog类遵守Eatable
```
class Dog: NSObject, Eatable {

}
```
这样我们就可以在其它地方轻松调用dog的eat方法，Person类与Robot类也是如法炮制
![](https://linxunfeng.github.io/images/2017/09/iOS-Swift-面向协议编程（一）/1.png)

至此，我们就可以通过面向协议的方式给类定制不同的功能，也就是模块化。可以发现Swift的面向协议编程跟c++的多继承很相似

## 约束
现在的这个Eatable协议是可以被任意遵守的，如果我们有这么个需求，我们创建的协议只是被UIViewController遵守，那我们该怎么做呢？
【当然，Eatable协议只能被UIViewController遵守很扯淡，这里只是举例，不要太在意咯~~】
> 在 extension 后面加上约束关键字【where】，并注明该协议只能被UIViewController这个类（包括子类）所遵守，而且此时我们还可以拿到遵守该协议的控制器的view

```
//import Foundation
import UIKit

protocol Eatable {
    
}

extension Eatable where Self : UIViewController {
    func eat() {
        view.backgroundColor = UIColor.red
    }
}
```

[Demo](https://github.com/LinXunFeng/LXFPOP)
接下来以一个实际应用来巩固下吧 [iOS - Swift 面向协议编程（二）](/2017/09/12/iOS-Swift-面向协议编程（二）/)