---
title: 打造Moya便捷解析库，提供RxSwift拓展
date: 2018-05-24 20:25:04
categories: "iOS"
tags:
  - iOS
  - Swift
  - RxSwift
---

<Excerpt in index | 首页摘要> 

MoyaMapper是基于Moya和SwiftyJSON封装的工具，以Moya的plugin的方式来实现间接解析，支持RxSwift

+<!-- more -->

<The rest of contents | 余下全文>

## 一、概述
1、相信大家在使用Swift开发时，[Moya](https://github.com/Moya/Moya)是首选的网络工具，在模型解析这一块，Swift版模型解析的相关第三方库有很多，本人最习惯用的就是[SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON)。

2、下面会开始讲解整个主要的开发功能与思想。

3、以下内容是基于大家会使用Moya和SwiftJSON的前提下所著，还不会的同学可以先简单了解后再来阅读本篇文章哦～

## 二、功能开发与思想讲解


### 1、尝试模型解析
Moya请求服务器返回的数据以Response类返回给我们，那我们就给Response类做一个扩展，这里以解析模型为例
```swift
// 需要传入一个参数，告知我们要转换出什么模型
public func mapObject<T: Modelable>(_ type: T.Type) -> T {
    // 模型解析过程
    。。。
  
    return T
}
```

Q: 那中间的解析过程该怎么写呢？

A: **可以让开发者遵守某个协议，实现指定的转换方法并描述转换关系。其转换过程我们不需要知道，交给开发者即可。**

那接着我们来定义一个协议Modelable，并声明转换方法
```swift
public protocol Modelable {
    mutating func mapping(_ json: JSON)
}
```

开发者创建一个`MyMoel`的结构体，遵守协议`Modelable`,并实现`mapping`，书写转换关系
```swift
struct MyModel: Modelable {
    var _id = ""
    
    mutating func mapping(_ json: JSON) {
        self._id = json["_id"].stringValue
    }
}
```

以目前的现状来分析一下：`mapObject`可以让开发者传入`模型类型`，而我们的协议方法却并非是个类方法。那我们需要先得到这个`模型类型`的对象，再来调用`mapping`方法

### 2、模型解析的驱动开发

Q: 怎么得到这个对象？

A: **可以在协议中声明一个初始化方法来创建对象。是的，我们在mapObject中创建对应模型类型的对象，调用mapping方法来转换数据，再把模型对象传出去即可。**

那我们在`Modelable`中声明一个init方法，并传入一个参数，区别于其它初始化方法
```swift
public protocol Modelable {
    mutating func mapping(_ json: JSON)
    init(_ json: JSON)
}
```

OK，现在把`mapObject`方法补齐模型解析过程

```swift
public func mapObject<T: Modelable>(_ type: T.Type) -> T {
    let modelJson = JSON(data)["modelKey"]
    // 模型解析过程
    var obj = T.init(modelJson)
    obj.mapping(modelJson)
    return obj
}
```

### 3、自定义解析键名

Q: 这样是搞定解析了，但是网络请求回来的json格式错综复杂，有什么办法可以让开发者来自行指定model对应的键名呢？

A: **嗯嗯，既然解析过程是在 Response 扩展里操作的，那我们可以通过协议定义键名属性，并且使用 Runtime 给Response动态添加一个属性，来记录遵守协议后的相应类名**

```swift
public protocol ModelableParameterType {
    /// 请求成功时状态码对应的值
    static var successValue: String { get }
    /// 状态码对应的键
    static var statusCodeKey: String { get }
    /// 请求后的提示语对应的键
    static var tipStrKey: String { get }
    /// 请求后的主要模型数据的键
    static var modelKey: String { get }
}
```

```swift
// MARK:- runtime
extension Response {
    private struct AssociatedKeys {
        static var lxf_modelableParameterKey = "lxf_modelableParameterKey"
    }
    var lxf_modelableParameter: ModelableParameterType.Type {
        get {
            let value = objc_getAssociatedObject(self, &AssociatedKeys.lxf_modelableParameterKey) as AnyObject
            guard let type = value as? ModelableParameterType.Type else { return NullParameter.self }
            return type
        } set {
            objc_setAssociatedObject(self, &AssociatedKeys.lxf_modelableParameterKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}
```

> **这里有个坑：_SwiftValue问题** (献上 [参考链接](https://stackoverflow.com/questions/42033735/failing-cast-in-swift-from-any-to-protocol/42034523#42034523))
> 如果我们存储的不是OC对象，那么`objc_getAssociatedObject`取出来的值的类型统统为`_SwiftValue`，直接`as? ModelableParameterType.Type`绝对是nil，需要在取出来后`as AnyObject`再转换为其它类型才会成功～～ 


现在开发者就可以创建一个类来遵守`ModelableParameterType`协议，并自定义解析键名
```swift
struct NetParameter : ModelableParameterType {
    static var successValue: String { return "false" }
    static var statusCodeKey: String { return "error" }
    static var tipStrKey: String { return "errMsg" }
    static var modelKey: String { return "results" }
}
```

### 4、插件注入
Q: 厉害了，不过要在什么时机下存储这个自定义键名的`NetParameter`？

A: 额，这个～～～ 哦，对了，可以通过Moya提供的插件机制！

翻出Moya中的Plugin.Swift，找到这个`process`方法，看看方法说明。
```swift
/// 在结束之前，可以被用来修改请求结果
/// Called to modify a result before completion.
func process(_ result: Result<Moya.Response, MoyaError>, target: TargetType) -> Result<Moya.Response, MoyaError>
```

那好，我们也做一个插件`MoyaMapperPlugin`给开发者使用，在创建`MoyaMapperPlugin`时把自定义解析键名的类型传进来
```swift
public struct MoyaMapperPlugin: PluginType {
    var parameter: ModelableParameterType.Type
    
    public init<T: ModelableParameterType>(_ type: T.Type) {
        parameter = type
    }
    
    // modify response
    public func process(_ result: Result<Response, MoyaError>, target: TargetType) -> Result<Response, MoyaError> {
        _ = result.map { (response) -> Response in
            // 趁机添加相关数据 
            response.lxf_modelableParameter = parameter
            return response
        }
        return result
    }
}
```

使用：开发者在创建`MoyaProvider`对象时，顺便注入插件。(OS: 这一步堪称“注入灵魂”)
```swift
MoyaProvider<LXFNetworkTool>(plugins: [MoyaMapperPlugin(NetParameter.self)])
```

### 5、总结
> 以上就是主要的踩坑过程了。模型数组解析和指定解析也跟这些差不多的，这里就不再赘述。本人已经将其封装成一个开源库 [MoyaMapper](https://github.com/LinXunFeng/MoyaMapper)，包含了上述已经和未曾说明的功能，下面会讲解如何去使用。以上部分可以称为开胃菜，目的就是平滑过渡到下面MoyaMapper的具体使用。

可能单单使用`MoyaMapper`的默认子库`Core`，作用体会上并不会很深。但是，如果你也是使用RxSwift来开发项目的话，请安装`'MoyaMapper/Rx'`吧，绝对一个字：「爽」


## 二、MoyaMapper的使用

![MoyaMapper](http://linxunfeng.github.io/images/2018/05/打造Moya便捷解析库，提供RxSwift拓展/MoyaMapper.png)

MoyaMapper是基于Moya和SwiftyJSON封装的工具，以Moya的plugin的方式来实现间接解析，支持RxSwift



![JSON数据对照](/images/2018/05/打造Moya便捷解析库，提供RxSwift拓展/JSON数据对照.png)

### 1、定义并注入自定义键名类
1. 定义一个遵守ModelableParameterType协议的结构体

```swift
// 各参数返回的内容请参考上面JSON数据对照图
struct NetParameter : ModelableParameterType {
    static var successValue: String { return "false" }
    static var statusCodeKey: String { return "error" }
    static var tipStrKey: String { return "" }
    static var modelKey: String { return "results" }
}
```
**此外，这里还可以做简单的路径处理，以应付各种情况，以'>'隔开**

```json
// 假设返回的json数据关于请求状态的相关数据如下所示，
error: {
    'errorStatus':false
    'errMsg':'error Argument type'
}

```

```swift
// 我们指明解析路径：error对象下的errMsg字段，一层层表示下去即可
static var tipStrKey: String { return "error>errMsg" }
```

2. 以plugin的方式传递给MoyaProvider

```swift
// MoyaMapperPlugin这里只需要传入类型
MoyaProvider<LXFNetworkTool>(plugins: [MoyaMapperPlugin(NetParameter.self)])
```

### 2、定义解析模型

创建一个遵守Modelable协议的结构体

```swift
struct MyModel: Modelable {
    
    var _id = ""
    ...
    
    init(_ json: JSON) { }
    
    mutating func mapping(_ json: JSON) {
        self._id = json["_id"].stringValue
        ...
    }
}

遵守Modelable协议，实现协议的两个方法，在`mapping`方法中描述模型字段的具体解析

```

### 3、解析数据

#### 0x00 请求结果与模型解析

```swift
// Result
public func mapResult(params: ModelableParamsBlock? = nil) -> MoyaMapperResult

// Model
public func mapObject<T: Modelable>(_ type: T.Type, modelKey: String? = nil) -> T
// Result+Model
public func mapObjResult<T: Modelable>(_ type: T.Type, params: ModelableParamsBlock? = nil) -> (MoyaMapperResult, T)

// Models
public func mapArray<T: Modelable>(_ type: T.Type, modelKey: String? = nil) -> [T]
// Result+Models
public func mapArrayResult<T: Modelable>(_ type: T.Type, params: ModelableParamsBlock? = nil) -> (MoyaMapperResult, [T])
```

上面的五个方法，观其名，知其意，这里就不过多解释了，主要注意两点：
  - result
```swift
// 元祖类型
// 参数1：根据statusCodeKey取出的值与successValue是否相等
// 参数2：根据tipStrKey取出的值
result：(Bool, String)
```

  - params
```swift
// params: ModelableParamsBlock? = nil
// 这里只有在特殊场景下才需要使用到。如：项目中需要在某处使用特定接口，但是返回的json格式跟自己项目的不一样，并且只有这么一两处用得着该额外接口，那就需要我们这个参数了，以Block的方式返回解析参数类型。
```

#### 0x01、特定解析

```swift
// Model
public func toJSON(modelKey: String? = nil) -> JSON
// 获取指定路径的值
public func fetchJSONString(path: String? = nil, keys: [JSONSubscriptType]) -> String
```

这两个方法，如果没有指定路径，默认都是针对modelKey的
```swift
// fetchJSONString(keys: <[JSONSubscriptType]>)
1、通过 keys 传递数组, 该数组可传入的类型为 Int 和 String
2、默认是以 modelKey 所示路径，来获取相应的数值。如果modelKey并非是你所想要使用的解析路径，可以使用下方的重载方法重新指定路径即可

// response.fetchJSONString(path: <String?>, keys: <[JSONSubscriptType]>)
```

**MoyaMapper也提供了Rx子库，为方便RxSwift的流式编程下便捷解析数据**

```ruby
MoyaMapper默认只安装Core下的文件
pod 'MoyaMapper'

RxSwift拓展
pod 'MoyaMapper/Rx'
```

具体使用还不是很明白的同学可以下载并运行`Example`看看

**如果[MoyaMapper](https://github.com/LinXunFeng/MoyaMapper)有什么不足的地方，欢迎提出issues，感谢大家的支持**