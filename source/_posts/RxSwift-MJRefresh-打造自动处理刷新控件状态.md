---
title: RxSwift + MJRefresh 打造自动处理刷新控件状态
date: 2017-10-24 08:51:20
categories: "iOS"
tags:
  - iOS
  - RxSwift
  - Swift
---

<Excerpt in index | 首页摘要> 

MVVM的模式中，多出了ViewModel这个角色，将逻辑处理、网络请求等繁杂操作中ViewController中抽离出来，ViewController得以瘦身。
结合RxSwift架构，我们一般就会在ViewModel中定义一个input收集繁杂操作所需的信息，通过一个transform方法将input作为参数传入，进而得到一个output供controller使用。

在使用RxSwift开发时会大量的使用到这种形式，其中就包括我们的网络请求。

+<!-- more -->

<The rest of contents | 余下全文>

> 本文是基于 [iOS-RxSwift项目实战记录](http://linxunfeng.top/2017/09/12/iOS-RxSwift-%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98%E8%AE%B0%E5%BD%95/) 所述，如果你还未阅读过，建议你最好还先阅读一遍，并下载Demo熟悉一下 : )

## 前言

MVVM的模式中，多出了ViewModel这个角色，将逻辑处理、网络请求等繁杂操作中ViewController中抽离出来，ViewController得以瘦身。
结合RxSwift架构，我们一般就会在ViewModel中定义一个input收集繁杂操作所需的信息，通过一个transform方法将input作为参数传入，进而得到一个output供controller使用。

在使用RxSwift开发时会大量的使用到这种形式，其中就包括我们的网络请求。
结合 [iOS-RxSwift项目实战记录](http://linxunfeng.top/2017/09/12/iOS-RxSwift-%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98%E8%AE%B0%E5%BD%95/) 中所述的“MJRefresh在RxSwift中的使用”，在output中定义了一个变量
```swift
let refreshStatus = Variable<LXFRefreshStatus>(.none)
```
controller通过output将其进行监听，从而当值发生变化时，controller就能实时获取当前应所处的刷新状态
```swift
vmOutput.refreshStatus.asObservable().subscribe(onNext: {[weak self] status in
    switch status {
    case .beingHeaderRefresh:
        self?.tableView.mj_header.beginRefreshing()
    case .endHeaderRefresh:
        self?.tableView.mj_header.endRefreshing()
    case .beingFooterRefresh:
        self?.tableView.mj_footer.beginRefreshing()
    case .endFooterRefresh:
        self?.tableView.mj_footer.endRefreshing()
    case .noMoreData:
        self?.tableView.mj_footer.endRefreshingWithNoMoreData()
    default:
        break
    }
}).addDisposableTo(rx_disposeBag)
```

如果在一个项目多处使用到了这种方式，我们就可以看到弊端——重复代码，过于冗余。

难道我们每次都要在controller中进行如此操作吗？

## 面向协议
关于协议的内容可以看下我之前的这两篇文章
[iOS - Swift 面向协议编程（一）](http://linxunfeng.top/2017/09/12/iOS-Swift-%E9%9D%A2%E5%90%91%E5%8D%8F%E8%AE%AE%E7%BC%96%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/)

 [iOS - Swift 面向协议编程（二）](http://linxunfeng.top/2017/09/12/iOS-Swift-%E9%9D%A2%E5%90%91%E5%8D%8F%E8%AE%AE%E7%BC%96%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/)

总结协议的两大作用：1、规范  2、定制能力


定义协议 Refreshable 

```swift
/* ============================ Refreshable ================================ */
// 需要使用 MJExtension 的控制器使用

protocol Refreshable {
    
}
extension Refreshable where Self : UIViewController {
    func initRefreshHeader(_ scrollView: UIScrollView, _ action: @escaping () -> Void) -> MJRefreshHeader {
        scrollView.mj_header = MJRefreshNormalHeader(refreshingBlock: { action() })
        return scrollView.mj_header
    }
    
    func initRefreshFooter(_ scrollView: UIScrollView, _ action: @escaping () -> Void) -> MJRefreshFooter {
        scrollView.mj_footer = MJRefreshAutoNormalFooter(refreshingBlock: { action() })
        return scrollView.mj_footer
    }
}
```
在controller中遵循 Refreshable 协议，通过initRefreshHeader方法或者initRefreshFooter方法给tableView或者collectionView赋予头部或尾部刷新的能力，并且书写下拉刷新时需要执行的代码
```swift
// 以下拉刷新为例
let refreshHeader = initRefreshHeader(liveCollectionView) { [weak self] in
    // 下拉后需要执行的操作 
    self?.vmOutput?.requestCommand.onNext(())
}
```

接下来再讲讲output，只要有网络请求的地方，就会需要需要监听请求状态，既然这样，那么可以为output定义一个协议OutputRefreshProtocol，专门用来规范必需声明的属性
```swift
/* ============================ OutputRefreshProtocol ================================ */
// viewModel 中 output使用

protocol OutputRefreshProtocol {
    // 告诉外界的tableView当前的刷新状态
    var refreshStatus : Variable<LXFRefreshStatus> {get}
}
```
接着让output去遵循该协议，并进行初始化刷新状态的值为.none
```
struct LXFLiveOutput: OutputRefreshProtocol {
    var refreshStatus: Variable<LXFRefreshStatus>
    
    let sections: Driver<[LXFLiveSection]>
    init(sections: Driver<[LXFLiveSection]>) {
        self.sections = sections
        refreshStatus = Variable<LXFRefreshStatus>(.none)
    }
}
```
到此为止，其实跟之前没啥两样，只是使controller更方便初始化刷新控件而已。接下来才是本文的重点。
## 重点
刷新的状态无非也就那么几种，下拉重载数据，上拉加载更多，请求完成时结束下拉或上拉等等。。。那我们何必要在每个controller中再去管理这等琐事？？
而至此，刷新控件的状态是由变量 refreshStatus 来决定，此时 refreshStatus 又声明在 OutputRefreshProtocol 协议中，我们何不再定义一个方法，将刷新控件的状态交给refreshStatus自己来帮我们处理呢～


```swift
extension OutputRefreshProtocol {
    func autoSetRefreshHeaderStatus(header: MJRefreshHeader?, footer: MJRefreshFooter?) -> Disposable {
        return refreshStatus.asObservable().subscribe(onNext: { (status) in
            switch status {
            case .beingHeaderRefresh:
                header?.beginRefreshing()
            case .endHeaderRefresh:
                header?.endRefreshing()
            case .beingFooterRefresh:
                footer?.beginRefreshing()
            case .endFooterRefresh:
                footer?.endRefreshing()
            case .noMoreData:
                footer?.endRefreshingWithNoMoreData()
            default:
                break
            }
        })
    }
}
```
这时需要我们将刷新控件的对象 header / footer 传入到方法中，实现自动控制刷新控件状态。

## 总结使用

一、output中遵守协议 OutputRefreshProtocol， 并初始化 refreshStatus 的值为 none

```swift
struct LXFLiveOutput: OutputRefreshProtocol {
    var refreshStatus: Variable<LXFRefreshStatus>
    
    let sections: Driver<[LXFLiveSection]>
    init(sections: Driver<[LXFLiveSection]>) {
        self.sections = sections
        refreshStatus = Variable<LXFRefreshStatus>(.none)
    }
}
```
二、controller 遵守协议 Refreshable，通过协议中的方法初始化刷新控件及对应的操作，并将刷新控件对象作为参数传入到自动处理状态方法中
```swift
extension LXFLiveViewController: Refreshable 
```
```swift
let refreshHeader = initRefreshHeader(liveCollectionView) { [weak self] in
    self?.vmOutput?.requestCommand.onNext(())
}
vmOutput?.autoSetRefreshHeaderStatus(header: refreshHeader, footer: nil).disposed(by: rx.disposeBag)
```

三、viewModel中根据实际情况实时更新 refreshStatus 的刷新状态

![image.png](http://linxunfeng.github.io/images/2017/10/RxSwift-MJExtension-打造自动处理刷新控件状态/1.png)

## 案例
协议：[Refreshable.swift](https://github.com/LinXunFeng/LXFBiliBili/blob/master/LXFBiliBili/LXFBiliBili/Classes/Common/Protocol/Lib/Refreshable.swift)
ViewModel：[LXFLiveViewModel](https://github.com/LinXunFeng/LXFBiliBili/blob/master/LXFBiliBili/LXFBiliBili/Classes/Main/Home/Controller/Live/ViewModel/LXFLiveViewModel.swift)
Controller：[LXFLiveViewController](https://github.com/LinXunFeng/LXFBiliBili/blob/master/LXFBiliBili/LXFBiliBili/Classes/Main/Home/Controller/Live/LXFLiveViewController.swift)

[LXFBiliBili](https://github.com/LinXunFeng/LXFBiliBili)


![LXFBiliBili](http://linxunfeng.github.io/images/2017/10/RxSwift-MJExtension-打造自动处理刷新控件状态/2.gif)