---
title: Swift - PropertyWrapper
date: 2021-04-22 00:18:23
categories: "iOS"
tags:
  - iOS
  - Swift
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---

## 一、知识点

> Property Wrapper，即属性包装器，其作用是将属性的 `定义代码` 与属性的`存储方式代码` 进行分离，抽取的`管理的存储代码`只需要编写一次，即可将功能应用于其它属性上。

### 1、基础用法

功能需求：确保值始终小于或等于12

这里我们直接使用 `property wrapper` 进行封装演示

```swift
@propertyWrapper
struct TwelveOrLess {
    private var number: Int
    
    // wrappedValue变量的名字是固定的
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
    
    init() {
        self.number = 0
    }
}

struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
}

var rectangle = SmallRectangle()
print(rectangle.height) // 0

rectangle.height = 10
print(rectangle.height) // 10

rectangle.height = 24
print(rectangle.height) // 12
```
这里可以注意到，在创建 `SmallRectangle` 实例时，并不需要初始化 `height` 和 `width`

原因：
被 `property wrapper` 声明的属性，实际上在存储时的类型是 `TwelveOrLess`，只不过编译器施了一些魔法，让它对外暴露的类型依然是被包装的原来的类型。
上面的 `SmallRectangle`  结构体，等同于下方这种写法

```swift
struct SmallRectangle {
    private var _height = TwelveOrLess()
    private var _width = TwelveOrLess()
    var height: Int {
        get { return _height.wrappedValue }
        set { _height.wrappedValue = newValue }
    }
    var width: Int {
        get { return _width.wrappedValue }
        set { _width.wrappedValue = newValue }
    }
}
```

### 2、设置初始值
```swift
@propertyWrapper
struct SmallNumber {
    private var maximum: Int
    private var number: Int
    
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, maximum) }
    }
    
    init() {
        maximum = 12
        number = 0
    }
    
    init(wrappedValue: Int) {
        print("init(wrappedValue:)")
        maximum = 12
        number = min(wrappedValue, maximum)
    }
    
    init(wrappedValue: Int, maximum: Int) {
        print("init(wrappedValue:maximum:)")
        self.maximum = maximum
        number = min(wrappedValue, maximum)
    }
}
```
使用了 `@SmallNumber` 但没有指定初始化值

```swift
struct ZeroRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int
}

var zeroRectangle = ZeroRectangle()
print(zeroRectangle.height, zeroRectangle.width) // 0 0
```

使用了 `@SmallNumber` ，并指定初始化值

这里会调用 `init(wrappedValue:)` 方法 

```swift
struct UnitRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber var width: Int = 1
}

var unitRectangle = UnitRectangle()
print(unitRectangle.height, unitRectangle.width) // 1 1
```

使用@SmallNumber，并传参进行初始化

这里会调用 `init(wrappedValue:maximum:)` 方法 

```swift
struct NarrowRectangle {
    // 报错：Extra argument 'wrappedValue' in call
    // @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int = 1
    // 这种初始化是可以的，调用 init(wrappedValue:maximum:) 方法
    // @SmallNumber(maximum: 9) var height: Int = 2
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}

var narrowRectangle = NarrowRectangle()
print(narrowRectangle.height, narrowRectangle.width) // 2 3

narrowRectangle.height = 100
narrowRectangle.width = 100
print(narrowRectangle.height, narrowRectangle.width) // 5 4
```

### 3、projectedValue

> `projectedValue`为 `property wrapper` 提供了额外的功能（如：标志某个状态，或者记录 `property wrapper` 内部的变化等）
>
> 两者都是通过实例的属性名进行访问，唯一不同的地方在于，`projectedValue` 需要在属性名前加上 `$` 才可以访问
>
> - `wrappedValue`: `实例.属性名`
> - `projectedValue`: `实例.$属性名`


 下面的代码将一个 `projectedValue` 属性添加到 `SmallNumber` 结构中，以在存储该新值之前跟踪该属性包装器是否调整了该属性的新值。

```swift
@propertyWrapper
struct SmallNumber1 {
    private var number: Int
    var projectedValue: Bool
    init() {
        self.number = 0
        self.projectedValue = false
    }
    var wrappedValue: Int {
        get { return number }
        set {
            if newValue > 12 {
                number = 12
                projectedValue = true
            } else {
                number = newValue
                projectedValue = false
            }
        }
    }
}
struct SomeStructure {
    @SmallNumber1 var someNumber: Int
}
var someStructure = SomeStructure()

someStructure.someNumber = 4
print(someStructure.$someNumber) // false

someStructure.someNumber = 55
print(someStructure.$someNumber) // true
```

这里的 `someStructure.$someNumber` 访问的是 `projectedValue`

### 4、使用限制

- 不能在协议里的属性使用

```swift
protocol SomeProtocol {
    // Property 'sp' declared inside a protocol cannot have a wrapper
    @SmallNumber1 var sp: Bool { get set }
}
```


- 不能在 extension 内使用

```swift
extension SomeStructure {
    // Extensions must not contain stored properties
    @SmallNumber1 var someProperty2: Int
}
```

- 不能在 `enum` 内使用

```swift
enum SomeEnum {
    // Property wrapper attribute 'SmallNumber1' can only be applied to a property
    @SmallNumber1 case a
    case b
}
```

- `class` 里的 `wrapper property` 不能覆盖其他的 property

```swift
class AClass {
    @SmallNumber1 var aProperty: Int
}

class BClass: AClass {
    // Cannot override with a stored property 'aProperty'
    override var aProperty: Int = 1
}
```

- `wrapper` 属性不能定义 `getter` 或 `setter` 方法

```swift
struct SomeStructure2 {
    // Property wrapper cannot be applied to a computed property
    @SmallNumber1 var someNumber: Int {
        get {
            return 0
        }
    }
}
```

- `wrapper` 属性不能被 `lazy`、 `@NSCopying`、 `@NSManaged`、 `weak`、 或者 `unowned` 修饰 


## 二、实际应用

> [Foil](https://github.com/jessesquires/Foil) -- 对 `UserDefaults` 进行了轻量级的属性包装第三方库
>
> 这部分我们主要简单的看下该第三方库的核心实现与使用

### 1、使用

- 声明
```swift
// 声明使用的key为flagEnabled，默认值为true
@WrappedDefault(key: "flagEnabled", defaultValue: true)
var flagEnabled: Bool

// 声明使用的key为timestamp
@WrappedDefaultOptional(key: "timestamp")
var timestamp: Date?
```

- 获取
```swift
// 获取变量在UserDefault中对应存储的值
self.flagEnabled
self.timestamp
```

- 赋值
```swift
// 设置UserDefault中对应存储的值
self.flagEnabled = false
self.timestamp = Date()
```

### 2、核心代码

`WrappedDefault.swift` 文件

```swift
@propertyWrapper
public struct WrappedDefault<T: UserDefaultsSerializable> {
    private let _userDefaults: UserDefaults

    /// 使用UserDefaults是所使用的key
    public let key: String

    /// 从UserDefaults中获取到的值
    public var wrappedValue: T {
        get {
            self._userDefaults.fetch(self.key)
        }
        set {
            self._userDefaults.save(newValue, for: self.key)
        }
    }

    // 初始化方法
    public init(
        keyName: String,        
        defaultValue: T,
        userDefaults: UserDefaults = .standard
    ) {
        self.key = keyName
        self._userDefaults = userDefaults
        
        // 对key所对应的值进行初始化（已有值则跳过，没有则进行初始化）
        userDefaults.registerDefault(value: defaultValue, key: keyName)
    }
}
```

`WrappedDefaultOptional.swift` 文件
```swift
@propertyWrapper
public struct WrappedDefaultOptional<T: UserDefaultsSerializable> {
    private let _userDefaults: UserDefaults

    public let key: String

    /// 从UserDefaults中获取到的值，无则返回nil
    public var wrappedValue: T? {
        get {
            self._userDefaults.fetchOptional(self.key)
        }
        set {
            if let newValue = newValue {
                // 更新值
                self._userDefaults.save(newValue, for: self.key)
            } else {
                // 删除值
                self._userDefaults.delete(for: self.key)
            }
        }
    }

    public init(keyName: String,
                userDefaults: UserDefaults = .standard) {
        self.key = keyName
        self._userDefaults = userDefaults
    }
}
```


## 三、资料

[Swift官方文档](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID617)

[apple / swift-evolution](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)