---
title: iOS - RxSwift 项目实战记录
date: 2017-09-12 10:02:27
categories: "iOS"
tags:
  - iOS
  - RxSwift
  - Swift
---

<Excerpt in index | 首页摘要> 

最近刚刚把接手的OC项目搞定，经过深思熟虑后，本人决定下个项目起就使用Swift(学了这么久的Swift还没真正用到实际项目里。。。)，而恰巧RxSwift已经出来有一些时间了，语法也基本上稳定，遂只身前来试探试探这RxSwift，接着就做了个小Demo，有兴趣的同学可以瞧一瞧~

+<!-- more -->

<The rest of contents | 余下全文>

![ReactiveX](linxunfeng.github.io/images/2017/09/iOS - RxSwift 项目实战记录/1.png)


> 最近刚刚把接手的OC项目搞定，经过深思熟虑后，本人决定下个项目起就使用Swift(学了这么久的Swift还没真正用到实际项目里。。。)，而恰巧RxSwift已经出来有一些时间了，语法也基本上稳定，遂只身前来试探试探这RxSwift，接着就做了个小Demo，有兴趣的同学可以瞧一瞧~

![Exhibition](linxunfeng.github.io/images/2017/09/iOS - RxSwift 项目实战记录/2.gif)


## 结构
```
.
├── Controller
│   └── LXFViewController.swift		// 主视图控制器
├── Extension
│   └── Response+ObjectMapper.swift	// Response分类，Moya请求完进行Json转模型或模型数组
├── Model
│   └── LXFModel.swift				// 模型
├── Protocol
│   └── LXFViewModelType.swift		// 定义了模型协议
├── Tool
│   ├── LXFNetworkTool.swift		// 封装Moya请求
│   └── LXFProgressHUD.swift		// 封装的HUD
├── View
│   ├── LXFViewCell.swift			// 自定义cell
│   └── LXFViewCell.xib				// cell的xib文件
└── ViewModel
    └── LXFViewModel.swift			// 视图模型
```

## 第三方库
```
RxSwift			// 想玩RxSwift的必备库
RxCocoa 		// 对 UIKit Foundation 进行 Rx 化
NSObject+Rx		// 为我们提供 rx_disposeBag 
Moya/RxSwift 	// 为RxSwift专用提供，对Alamofire进行封装的一个网络请求库
ObjectMapper 	// Json转模型之必备良品
RxDataSources	// 帮助我们优雅的使用tableView的数据源方法
Then 			// 提供快速初始化的语法糖
Kingfisher		// 图片加载库
SnapKit			// 视图约束库
Reusable 		// 帮助我们优雅的使用自定义cell和view,不再出现Optional
MJRefresh 		// 上拉加载、下拉刷新的库
SVProgressHUD 	// 简单易用的HUD
```

## 敲黑板
### Moya的使用 
Moya是基于Alamofire的网络请求库，这里我使用了Moya/Swift，它在Moya的基础上添加了对RxSwift的接口支持。接下来我们来说下Moya的使用

一、创建一个枚举，用来存放请求类型，这里我顺便设置相应的路径，等下统一取出来直接赋值即可
```swift
enum LXFNetworkTool {
    enum LXFNetworkCategory: String {
        case all     = "all"
        case android = "Android"
        case ios     = "iOS"
        case welfare = "福利"
    }
    case data(type: LXFNetworkCategory, size:Int, index:Int)
}
```
二、为这个枚举写一个扩展，并遵循塄 TargetType，这个协议的Moya这个库规定的协议，可以按住Commond键+单击左键进入相应的文件进行查看
```swift
extension LXFNetworkTool: TargetType {
    /// baseURL 统一基本的URL
    var baseURL: URL {
        return URL(string: "http://gank.io/api/data/")!
    }
    
    /// path字段会追加至baseURL后面
    var path: String {
        switch self {
        case .data(let type, let size, let index):
            return "\(type.rawValue)/\(size)/\(index)"
        }
    }
    
    /// HTTP的请求方式
    var method: Moya.Method {
        return .get
    }
    
    /// 请求参数(会在请求时进行编码)
    var parameters: [String: Any]? {
        return nil
    }
    
    /// 参数编码方式(这里使用URL的默认方式)
    var parameterEncoding: ParameterEncoding {
        return URLEncoding.default
    }
    
    /// 这里用于单元测试，不需要的就像我一样随便写写
    var sampleData: Data {
        return "LinXunFeng".data(using: .utf8)!
    }
    
    /// 将要被执行的任务(请求：request 下载：upload 上传：download)
    var task: Task {
        return .request
    }
    
    /// 是否执行Alamofire验证，默认值为false
    var validate: Bool {
        return false
    }
}
```
三、定义一个全局变量用于整个项目的网络请求
```swift
let lxfNetTool = RxMoyaProvider<LXFNetworkTool>()
```
至此，我们就可以使用这个全局变量来请求数据了
### RxDataSources
如果你想用传统的方式也行，不过这就失去了使用RxSwift的意义。好吧，我们接下来说说如何优雅的来实现tableView的数据源。其实[RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources)官网上已经有很明确的使用说明，不过我还是总结一下整个过程吧。

概念点
RxDataSources是以section来做为数据结构来传输，这点很重要，可能很多同学会比较疑惑这句话吧，我在此举个例子，在传统的数据源实现的方法中有一个numberOfSection，我们在很多情况下只需要一个section，所以这个方法可实现，也可以不实现，默认返回的就是1，这给我们带来的一个迷惑点：【tableView是由row来组成的】，不知道在坐的各位中有没有是这么想的呢？？有的话那从今天开始就要认清楚这一点，【tableView其实是由section组成的】，所以在使用RxDataSources的过程中，即使你的setion只有一个，那你也得返回一个section的数组出去！！！

一、自定义Section
在我们自定义的Model中创建一个Section的结构体，并且创建一个扩展，遵循SectionModelType协议，实现相应的协议方法。约定俗成的写法呢请参考如下方式
```swift
LXFModel.swift

struct LXFSection {
    // items就是rows
    var items: [Item]
  	// 你也可以这里加你需要的东西，比如 headerView 的 title
}

extension LXFSection: SectionModelType {
  
  	// 重定义 Item 的类型为 LXFModel
    typealias Item = LXFModel
  
    // 实现协议中的方式
    init(original: LXFSection, items: [LXFSection.Item]) {
        self = original
        self.items = items
    }
}
```
二、在控制器下创建一个数据源属性

以下代码均在 LXFViewController.swift 文件中
```swift
// 创建一个数据源属性，类型为自定义的Section类型
let dataSource = RxTableViewSectionedReloadDataSource<LXFSection>()
```
使用数据源属性绑定我们的cell
```swift
// 绑定cell
dataSource.configureCell = { ds, tv, ip, item in
    // 这个地方使用了Reusable这个库，在LXFViewCell中遵守了相应的协议
	// 使其方便转换cell为非可选型的相应的cell类型
    let cell = tv.dequeueReusableCell(for: ip) as LXFViewCell
    cell.picView.kf.setImage(with: URL(string: item.url))
    cell.descLabel.text = "描述: \(item.desc)"
    cell.sourceLabel.text = "来源: \(item.source)"
    return cell
}
```
三、将sections序列绑定给我们的rows
```swift
output.sections.asDriver().drive(tableView.rx.items(dataSource:dataSource)).addDisposableTo(rx_disposeBag)
```
大功告成，接下来说说section序列的产生
### ViewModel的规范
我们知道MVVM思想就是将原本在ViewController的视图显示逻辑、验证逻辑、网络请求等代码存放于ViewModel中，让我们手中的ViewController瘦身。这些逻辑由ViewModel负责，外界不需要关心，外界只需要结果，ViewModel也只需要将结果给到外界，基于此，我们定义了一个协议LXFViewModelType

一、创建一个LXFViewModelType.swift
```swift
LXFViewModelType.swift

// associatedtype 关键字 用来声明一个类型的占位符作为协议定义的一部分
protocol LXFViewModelType {
    associatedtype Input
    associatedtype Output
    
    func transform(input: Input) -> Output
}
```
二、viewModel遵守LXFViewModelType协议

1. 我们可以为XFViewModelType的Input和Output定义别名，以示区分，如：你这个viewModel的用于请求首页模块相关联的，则可以命名为：HomeInput 和 HomeOutput
2. 我们可以丰富我们的 Input 和 Output 。可以看到我为Output添加了一个序列，类型为我们自定义的LXFSection数组，在Input里面添加了一个请求类型(即要请求什么数据，比如首页的数据)
3. 我们通过 transform 方法将input携带的数据进行处理，生成了一个Output

**注意： 以下代码为了方便阅读，进行了部分删减**

```swift
LXFViewModel.swift

extension LXFViewModel: LXFViewModelType {
   // 存放着解析完成的模型数组
   let models = Variable<[LXFModel]>([])
  
    // 为LXFViewModelType的Input和Output定义别名
    typealias Input = LXFInput
    typealias Output = LXFOutput

  	// 丰富我们的Input和Output
    struct LXFInput {
        // 网络请求类型
        let category: LXFNetworkTool.LXFNetworkCategory
        
        init(category: LXFNetworkTool.LXFNetworkCategory) {
            self.category = category
        }
    }

    struct LXFOutput {
        // tableView的sections数据
        let sections: Driver<[LXFSection]>
        
        init(sections: Driver<[LXFSection]>) {
            self.sections = sections
        }
    }
    
    func transform(input: LXFViewModel.LXFInput) -> LXFViewModel.LXFOutput {
        let sections = models.asObservable().map { (models) -> [LXFSection] in
            // 当models的值被改变时会调用，这是Variable的特性
            return [LXFSection(items: models)] // 返回section数组
        }.asDriver(onErrorJustReturn: [])
        
        let output = LXFOutput(sections: sections)
        
      	// 接下来的代码是网络请求，请结合项目查看，不然会不方便阅读和理解
    }
}
```
接着我们在ViewController中初始化我们的input，通过transform得到output，然后将我们output中的sections序列绑定tableView的items
```swift
LXFViewController.swift

// 初始化input
let vmInput = LXFViewModel.LXFInput(category: .welfare)
// 通过transform得到output
let vmOutput = viewModel.transform(input: vmInput)

vmOutput.sections.asDriver().drive(tableView.rx.items(dataSource: dataSource)).addDisposableTo(rx_disposeBag)
```

### RxSwift中使用MJRefresh
一、定义一个枚举LXFRefreshStatus，用于标志当前刷新状态
```swift
enum LXFRefreshStatus {
    case none
    case beingHeaderRefresh
    case endHeaderRefresh
    case beingFooterRefresh
    case endFooterRefresh
    case noMoreData
}
```
二、在LXFOutput添加一个refreshStatus序列，类型为LXFRefreshStatus
```swift
// 给外界订阅，告诉外界的tableView当前的刷新状态
let refreshStatus = Variable<LXFRefreshStatus>(.none)
```
我们在进行网络请求并得到结果之后，修改refreshStatus的value为相应的LXFRefreshStatus项

三、外界订阅output的refreshStatus

外界订阅output的refreshStatus，并且根据接收到的值进行相应的操作
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
四、output提供一个requestCommond用于请求数据

PublishSubject 的特点：即可以作为Observable，也可以作为Observer，说白了就是可以发送信号，也可以订阅信号
```swift
// 外界通过该属性告诉viewModel加载数据（传入的值是为了标志是否重新加载）
let requestCommond = PublishSubject<Bool>()
```
在transform中，我们对生成的output的requestCommond进行订阅
```swift
output.requestCommond.subscribe(onNext: {[unowned self] isReloadData in
    self.index = isReloadData ? 1 : self.index+1
    lxfNetTool.request(.data(type: input.category, size: 10, index: self.index)).mapArray(LXFModel.self).subscribe({ [weak self] (event) in
        switch event {
        case let .next(modelArr):
            self?.models.value = isReloadData ? modelArr : (self?.models.value ?? []) + modelArr
            LXFProgressHUD.showSuccess("加载成功")
        case let .error(error):
            LXFProgressHUD.showError(error.localizedDescription)
        case .completed:
            output.refreshStatus.value = isReloadData ? .endHeaderRefresh : .endFooterRefresh
        }
    }).addDisposableTo(self.rx_disposeBag)
}).addDisposableTo(rx_disposeBag)
```
五、在ViewController中初始化刷新控件

为tableView设置刷新控件，并且在创建刷新控件的回调中使用output的requestCommond发射信号
```swift
tableView.mj_header = MJRefreshNormalHeader(refreshingBlock: { 
    vmOutput.requestCommond.onNext(true)
})
tableView.mj_footer = MJRefreshAutoNormalFooter(refreshingBlock: { 
    vmOutput.requestCommond.onNext(false)
})
```
总结流程：



1. ViewController已经拿到output，当下拉加载数据的时候，使用output的requestCommond发射信息，告诉viewModel我们要加载数据

2. viewModel请求数据，在处理完json转模型或模型数组后修改models，当models的值被修改的时候会发信号给sections，sections在ViewController已经绑定到tableView的items了，所以此时tableView的数据会被更新。接着我们根据请求结果，修改output的refreshStatus属性的值

3. 当output的refreshStatus属性的值改变后，会发射信号，由于外界之前已经订阅了output的refreshStatus，此时就会根据refreshStatus的新值来处理刷新控件的状态

好了，附上[RxSwiftDemo](https://github.com/LinXunFeng/RxSwiftDemo)。完结撒花



<div class="github-widget" data-repo="LinXunFeng/RxSwiftDemo"></div>