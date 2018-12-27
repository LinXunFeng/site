---
title: Swift 掌控Moya的网络请求、数据解析与缓存
date: 2018-10-27 23:41:37
categories: "iOS"
tags:
  - iOS
  - Swift
---

<Excerpt in index | 首页摘要> 

- `Moya` 在Swift开发中起着重要的网络交互作用，但是还有不如之处，比如网络不可用时，返回的 `Response` 为 `nil`，这时还得去解析相应的 `Error`
- `Codable` 可以帮助我们快速的解析数据，但是一旦声明的属性类型与json中的不一致，将无法正常解析; 而且对于模型中自定义属性名的处理也十分繁琐

+<!-- more -->

<The rest of contents | 余下全文>



- `Moya` 在Swift开发中起着重要的网络交互作用，但是还有不如之处，比如网络不可用时，返回的 `Response` 为 `nil`，这时还得去解析相应的 `Error`
- `Codable` 可以帮助我们快速的解析数据，但是一旦声明的属性类型与json中的不一致，将无法正常解析; 而且对于模型中自定义属性名的处理也十分繁琐

解决的方案有很多，不过我比较习惯使用 `MoyaMapper` ，不仅可以解决上述问题，还提供了多种`模型转换`、`数据互转`、`多种数据类型任意存储`的便捷方法。掌控Moya的网络请求、数据解析与缓存简直易如反掌。

> `MoyaMapper`是基于Moya和SwiftyJSON封装的工具，以Moya的plugin的方式来实现间接解析，支持RxSwift
>
>  GitHub: [MoyaMapper](https://github.com/MoyaMapper/MoyaMapper)
>
> 📖 详细的使用请查看手册 [https://MoyaMapper.github.io](https://moyamapper.github.io/)

## 特点

-  支持`json` 转 `Model` 自动映射 与 自定义映射
-  无视 `json` 中值的类型，`Model` 中属性声明的是什么类型，它就是什么类型
-  支持 `Data` `字典` `JSON` `json字符串` `Model` 互转
-  插件方式，全方位保障`Moya.Response`，拒绝各种网络问题导致 `Response` 为 `nil`，将各式各样的原因导致的数据加载失败进行统一处理，开发者只需要关注 `Response`
-  可选 - 支持数据随意缓存( `JSON` 、 `Number` 、`String`、 `Bool`、 `Moya.Response` )
-  可选 - 支持网络请求缓存

## 数据解析

##### 一、插件注入

附：[插件 MoyaMapperPlugin 的详细使用](https://moyamapper.github.io/plugin/)

![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/success-obj.png)

1、定义适用于项目接口的 `ModelableParameterType`

```swift
// statusCodeKey、tipStrKey、 modelKey 可以任意指定级别的路径，如： "error>used"
struct NetParameter : ModelableParameterType {
    var successValue = "000"
    var statusCodeKey = "retStatus"
    var tipStrKey = "retMsg"
    var modelKey = "retBody"
}
```

2、在 `MoyaProvider` 中使用 `MoyaMapperPlugin` 插件，并指定 `ModelableParameterType`

```swift
let lxfNetTool = MoyaProvider<LXFNetworkTool>(plugins: [MoyaMapperPlugin(NetParameter())])
```

❗ 使用 `MoyaMapperPlugin` 插件是整个 `MoyaMapper`  的核心所在！

##### 二、Model声明

> `Model` 需遵守 `Modelable ` 协议
> - `MoyaMapper` 支持模型自动映射 和 自定义映射
> - 不需要考虑源json数据的真实类型，这里统一按 `Model` 中属性声明的类型进行转换


1、一般情况下如下写法即可

```swift
struct CompanyModel: Modelable {
    
    var name : String = ""
    var catchPhrase : String = ""
    
    init() { }
}
```

2、如果自定义映射，则可以实现方法 `mutating func mapping(_ json: JSON)`

```swift
struct CompanyModel: Modelable {
    
    var name : String = ""
    var catchPhrase : String = ""
    
    init() { }
    mutating func mapping(_ json: JSON) {
        self.name = json["nickname"].stringValue
    }
}
```
3、支持模型嵌套

```swift
struct UserModel: Modelable {
    
    var id : String = ""
    var name : String = ""
    var company : CompanyModel = CompanyModel()
    
    init() { }
}
```

##### 三、Response 解析

> 1、以下示例皆使用了 `MoyaMapperPlugin` ，所以不需要指定 `解析路径`
>
> 2、如果没有使用 `MoyaMapperPlugin` 则需要指定 `解析路径`，否则无法正常解析
>
> ps:  `解析路径` 可以使用 `a>b` 这种形式来解决多级路径的问题



解析方法如下列表所示

|      方法       | 描述 (支持RxSwift)                                           |
| :-------------: | :----------------------------------------------------------- |
|     toJSON      | Response 转 JSON ( [toJSON](https://moyamapper.github.io/core/toJSON/)   [rx.toJSON](https://moyamapper.github.io/rx/toJSON/)) |
|   fetchString   | 获取指定路径的字符串( [fetchString](https://moyamapper.github.io/core/fetchString/)   [rx.fetchString](https://moyamapper.github.io/rx/fetchString/)) |
| fetchJSONString | 获取指定路径的原始json字符串 ( [fetchJSONString](https://moyamapper.github.io/core/fetchJSONString/)   [rx.fetchJSONString](https://moyamapper.github.io/rx/fetchJSONString/) ) |
|    mapResult    | Response -> MoyaMapperResult   `(Bool, String)` ( [mapResult](https://moyamapper.github.io/core/mapResult/)   [rx.mapResult](https://moyamapper.github.io/rx/mapResult/) ) |
|    mapObject    | Response -> Model ( [mapObject](https://moyamapper.github.io/core/mapObject/)   [rx.mapObject](https://moyamapper.github.io/rx/mapObject/)) |
|  mapObjResult   | Response -> (MoyaMapperResult, Model) ( [mapObjResult](https://moyamapper.github.io/core/mapObjResult/)   [rx.mapObjResult](https://moyamapper.github.io/rx/mapObjResult/)) |
|    mapArray     | Response -> [Model] ( [mapArray](https://moyamapper.github.io/core/mapArray/)  [rx.mapArray](https://moyamapper.github.io/rx/mapArray/)) |
| mapArrayResult  | Response -> (MoyaMapperResult, [Model]) ( [mapArrayResult](https://moyamapper.github.io/core/mapArrayResult/)   [rx.mapArrayResult](https://moyamapper.github.io/rx/mapArrayResult/)) |

❗除了 `fetchJSONString` 的默认解析路径是`根路径`之外，其它方法的默认解析路径为插件对象中的 `modelKey`



如果接口请求后 `json` 的数据结构与下图类似，则使用 `MoyaMapper` 是最合适不过了



![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/success-obj.png)



```swift
// Normal
let model = response.mapObject(MMModel.self)
print("name -- \(model.name)")
print("github -- \(model.github)")

// 打印json
print(response.fetchJSONString())

// Rx
rxRequest.mapObject(MMModel.self)
    .subscribe(onSuccess: { (model) in
        print("name -- \(model.name)")
        print("github -- \(model.github)")
    }).disposed(by: disposeBag)
```

附： [fetchJSONString的详细使用](https://moyamapper.github.io/core/fetchJSONString/)



![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/success-array.png)




```swift
// Normal
let models = response.mapArray(MMModel.self)
let name = models[0].name
print("count -- \(models.count)")
print("name -- \(name)")

// 打印 json 模型数组中第一个的name
print(response.fetchString(keys: [0, "name"]))

// Rx
rxRequest.mapArray(MMModel.self)
    .subscribe(onSuccess: { models in
        let name = models[0].name
        print("count -- \(models.count)")
        print("name -- \(name)")
    }).disposed(by: disposeBag)
```

附：[mapArray的详细使用说明](https://moyamapper.github.io/core/mapArray/)



![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/fail.png)




```swift
// Normal
let (isSuccess, tipStr) = response.mapResult()
print("isSuccess -- \(isSuccess)")
print("tipStr -- \(tipStr)")

// Rx
rxRequest.mapResult()
    .subscribe(onSuccess: { (isSuccess, tipStr) in
        print("isSuccess -- \(isSuccess)") // 是否为 "000"
        print("retMsg -- \(retMsg)") // "缺少必要参数"
    }).disposed(by: disposeBag)
```

附：[mapResult的详细使用说明](https://moyamapper.github.io/core/mapResult/)


## 统一处理网络请求结果

> 在APP的实际使用过程中，会遇到各种各样的网络请求结果，如:服务器挂了、手机无网络，此时 `Moya` 返回的 `Response` 为 nil，这样我们就不得不去判断 `Error`。但是使用 `MoyaMapperPlugin` 就可以让我们只关注 `Response`

```swift
// MoyaMapperPlugin 的初始化方法
public init<T: ModelableParameterType>(
    _ type: T,
    transformError: Bool = true
)

type : ModelableParameterType  用于定义字段路径，做为全局解析数据的依据
transformError : Bool  是否当网络请求失败时，自动转换请求结果，默认为 true
```

- 当请求失败的时候，此时的 `result.response` 为 `nil`，根据`transformError`是否为`true` 判断是否创建一个自定义的 `response` 并返回出去。


➡ 本来可以请求到的数据内容



![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/success-obj.png)




➡ 现在关闭网络，再请求数据

- 正常情况下，即不做任何不处理的时候， `Response` 为 `nil` 

- 经过 `MoyaMapperPlugin` 处理的后可得到转换后的 `Response` ，如图



![](linxunfeng.github.io/images/2018/10/Swift-掌控Moya的网络请求、数据解析与缓存/plugin.png)



这里将请求失败进行了统一处理，无论是服务器还是自身网络的问题，`retStatus` 都为 MMStatusCode.loadFail，但是 errorDescription 会保持原来的样子并赋值给 `retMsg`。

> - `retStatus` 值会从枚举 `MMStatusCode` 中取  `loadFail.rawValue` ，即 `700` 
> - 取 类型为 `ModelableParameterType`  的 `type` 中 `statusCodeKey` 所指定的值 为键名，`retMsg`也同理

ps: 这个时候可以通过判断 `retStatus` 或 `response.statusCode` 是否与 `MMStatusCode.loadFail.rawValue` 相同来判断是否显示加载失败的空白页占位图

```swift
enum MMStatusCode: Int {
    case cache = 230
    case loadFail = 700
}
```
枚举 `MMStatusCode` 中除了 `loadFail` ，还有 `cache`，我们已经知道 `loadFail` 在数据加载失败的时候会出现，那 `cache` 是在什么时候出来呢？不急，看下一节就知道了。

## 数据缓存

##### 一、基本使用

```
// 缓存
@discardableResult
MMCache.shared.cache`XXX`(value : XXX, key: String, cacheContainer: MMCache.CacheContainer = .RAM)  -> Bool
// 取舍
MMCache.shared.fetch`XXX`Cache(key: String, cacheContainer: MMCache.CacheContainer = .RAM)
```

缓存成功会返回一个 `Bool` 值，这里可不接收

| XXX 所支持类型 |             |
| -------------- | ----------- |
| Bool           | -           |
| Float          | -           |
| Double         | -           |
| String         | -           |
| JSON           | -           |
| Modelable      | [Modelable] |
| Moya.Response  | -           |
| Int            | UInt        |
| Int8           | UInt8       |
| Int16          | UInt16      |
| Int32          | UInt32      |
| Int64          | UInt64      |

> 其中，除了 `Moya.Response` 之外，其它类型皆是通过 `JSON` 来实现缓存

所以，如果你想清除这些类型的缓存，只需要调用如下方法即可

```swift
@discardableResult
func removeJSONCache(_ key: String, cacheContainer: MMCache.CacheContainer = .RAM) -> Bool

@discardableResult
func removeAllJSONCache(cacheContainer: MMCache.CacheContainer = .RAM) -> Bool
```

清除 `Moya.Response` 则使用如下两个方法

```swift
@discardableResult
func removeResponseCache(_ key: String) -> Bool

@discardableResult
func removeAllResponseCache() -> Bool
```

再来看看MMCache.CacheContainer

```swift
enum CacheContainer {
    case RAM 	// 只缓存于内存的容器
    case hybrid // 缓存于内存与磁盘的容器
}
```

> 这两种容器互不相通，即 即使key相同，使用 `hybrid` 来缓存后，再通过 `RAM` 取值是取不到的。

- RAM : 仅缓存于内存之中，缓存的数据在APP使用期间一直存在
- hybrid ：缓存于内存与磁盘中，APP重启后也可以获取到数据



##### 二、缓存网络请求

内部缓存过程：

1. APP首次启动并进行网络请求，网络数据将缓存起来
2. APP再次启动并进行网络请求时，会先返回缓存的数据，等请求成功后再返回网络数据
3. 其它情况只会加载网络数据
4. 每次成功请求到数据后，都会对缓存的数据进行更新

```swift
// Normal
func cacheRequest(
    _ target: Target, 
    cacheType: MMCache.CacheKeyType = .default, 
    callbackQueue: DispatchQueue? = nil, 
    progress: Moya.ProgressBlock? = nil, 
    completion: @escaping Moya.Completion
) -> Cancellable

// Rx
func cacheRequest(
    _ target: Base.Target, 
    callbackQueue: DispatchQueue? = nil, 
    cacheType: MMCache.CacheKeyType = .default
) -> Observable<Response>
```

> 实际上是对 `Moya` 请求后的 `Response` 进行缓存。 其实与 `Moya` 自带的方法相比较只多了一个参数 `cacheType: MMCache.CacheKeyType` ，定义着缓存中的 `key` ，默认为 `default` 

下面是 `MMCache.CacheKeyType` 的定义

```
/**
 let cacheKey = [method]baseURL/path
 
 - default : cacheKey + "?" + parameters
 - base : cacheKey
 - custom : cacheKey + "?" + customKey
 */
public enum CacheKeyType {
    case `default`
    case base
    case custom(String)
}
```

> 如果你想缓存`多页`列表数据的`最新一页`数据，此时使用 `default` 是不合适的，因为 `default` 使用的 `key` 包含了 `pageIndex`，这样就达不到只缓存 `最新一页数据` 的目的， 所以这里应该使用 `base` 或者 `custom(String)` 


我们可以来试一下带缓存的请求

```swift
/*
 * APP第一次启动并进行网络请求，网络数据将缓存起来
 * APP再次启动并进行网络请求时，会先加载缓存，再加载网络数据
 * 其它情况只会加载网络数据
 * 每次成功请求到数据都会进行数据更新
 */
lxfNetTool.rx.cacheRequest(.data(type: .all, size: 10, index: 1))
    .subscribe(onNext: { response in
        log.debug("statusCode -- \(response.statusCode)")
    }).disposed(by: disposeBag)

// 传统方式
/*
let _ = lxfNetTool.cacheRequest(.data(type: .all, size: 10, index: 1)) { result in
    guard let resp = result.value else { return }
    log.debug("statusCode -- \(resp.statusCode)")
}
*/
```

打印结果

```
// 首次使用APP
statusCode -- 200

// 关闭并重新打开APP，再请求一下
statusCode -- 230
statusCode -- 200

// 然后再请求一下
statusCode -- 200
```

这里的 `230` 就是 `MMStatusCode.cache.rawValue`


## CocoaPods

- 默认安装

MoyaMapper默认只安装Core下的文件
```ruby
pod 'MoyaMapper'
```

- RxSwift拓展
```ruby
pod 'MoyaMapper/Rx'
```

- 缓存拓展
```ruby
pod 'MoyaMapper/MMCache'
```

- Rx缓存
```ruby
pod 'MoyaMapper/RxCache'
```