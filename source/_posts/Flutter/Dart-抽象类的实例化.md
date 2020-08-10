---
title: Dart - 抽象类的实例化
date: 2020-06-07 11:27:00
categories: "Flutter"
tags:
  - Dart

---

<Excerpt in index | 首页摘要> 

抽象类不能用于创建实例，但是有没有发现，`Dart` 提供的 `Map` 和 `List` 就是抽象类，却可以直接使用它们创建出一个实例对象

+<!-- more -->

<The rest of contents | 余下全文>

## 一、抽象类的使用
`Dart` 抽象类可以只声明方法，也可以有具体的方法实现，但是不能直接用抽象类来创建实例，只能被继承使用或者充当接口。

定义一个抽象类 `Animal`
```dart
abstract class Animal {
  // 仅声明eat方法
  void eat();

  // 声明方法，且有具体实现
  void sleep() {
    print("睡觉");
  }
}
```

继承使用

```dart
class Cat extends Animal {
  @override
  void eat() {
    print("喵喵吃");
    sleep();
  }

// 可以不实现 sleep 方法
}
```
充当接口
```dart

class Cat implements Animal {
  void eat() {
    print("吃");
  }

  // 必须实现 sleep 方法
  void sleep() {
    print('睡');
  }
}
```
实例化
```dart
final animal = Animal();
// 抽象类实例化会报错
// Error: The class 'Test' is abstract and can't be instantiated.
```
> - 抽象类不能实例化。
> - 继承: 子类比较实现抽象方法，子类可以不重写抽象类中已实现的方法。
> - 接口: 必须实现抽象类中声明的所有方法

## 二、抽象类的实例化

上面提到了抽象类不能用于创建实例，但是有没有发现，`Dart` 提供的 `Map` 和 `List` 就是抽象类，却可以直接使用它们创建出一个实例对象
```dart
final list = List();
final dict = Map<String, dynamic>();
```
我们来看一下 `Map` 的源码：
![Map源码](/images/2020/06/Dart-抽象类的实例化/Map源码.png)

`Map` 的确是抽象类，不过此时我们也注意到了，在 `Map` 这个抽象类中，定义了一个工厂构造方法，**这就是使抽象类可实例化的关键所在，因为工厂方法可以返回一个实例对象，但这个对象的类型不一定就是当前类!**

在这个地方，`Map` 的工厂方法并没有具体的实现，而只是在工厂构造方法前加了一个关键字 `external`。
`external` 关键字可以让方法的声明与实现分离，即 可以由外部来帮我们完成具体的方法实现，那外部如何才能关联到该声明的方法呢？这里就需要用到注解 `@patch`，使外部的方法实现与该声明的方法绑定

> `external` 可以分离方法的声明与实现
> `@patch` 关联某个类中用 `external` 修饰的方法的实现

根据如下路径可以找到 `Map` 的具体实现源码
```dart
// flutter/bin/cache/dart-sdk/lib/_internal/vm/lib/map_patch.dart

@patch
factory Map() => new LinkedHashMap<K, V>();
```

可以看到，这里使用了 `LinkedHashMap` 来实现 `Map` 。

我们再去看一下 `LinkedHashMap` 的实现源码，路径如下：
```dart
// flutter/bin/cache/dart-sdk/lib/collection/linked_hash_map.dart

external factory LinkedHashMap(
    {bool Function(K, K)? equals,
    int Function(K)? hashCode,
    bool Function(dynamic)? isValidKey});
```
这里我们又发现 `LinkedHashMap` 也仅仅只是声明，找到具体实现

```dart
// flutter/bin/cache/dart-sdk/lib/_internal/vm/lib/collection_patch.dart

@patch
class LinkedHashMap<K, V> {
  @patch
  factory LinkedHashMap(
      {bool equals(K key1, K key2)?,
      int hashCode(K key)?,
      bool isValidKey(potentialKey)?}) {
    if (isValidKey == null) {
      if (hashCode == null) {
        if (equals == null) {
          return new _InternalLinkedHashMap<K, V>();
        }
        hashCode = _defaultHashCode;
      } else {
        if (identical(identityHashCode, hashCode) &&
            identical(identical, equals)) {
          return new _CompactLinkedIdentityHashMap<K, V>();
        }
        equals ??= _defaultEquals;
      }
    } else {
      hashCode ??= _defaultHashCode;
      equals ??= _defaultEquals;
    }
    return new _CompactLinkedCustomHashMap<K, V>(equals, hashCode, isValidKey);
  }

...
}
```

可以看到，`LinkedHashMap`的工厂构造方法返回的实例类型是 `_InternalLinkedHashMap` 或 `_CompactLinkedCustomHashMap` ，这里我们再看一下这两个类的实现源码

```dart
// flutter/bin/cache/dart-sdk/lib/_internal/vm/lib/compact_hash.dart

@pragma("vm:entry-point")
class _InternalLinkedHashMap<K, V> extends _HashVMBase
    with
        MapMixin<K, V>,
        _LinkedHashMapMixin<K, V>,
        _HashBase,
        _OperatorEqualsAndHashCode
    implements LinkedHashMap<K, V> {
  _InternalLinkedHashMap() {
    _index = new Uint32List(_HashBase._INITIAL_INDEX_SIZE);
    _hashMask = _HashBase._indexSizeToHashMask(_HashBase._INITIAL_INDEX_SIZE);
    _data = new List.filled(_HashBase._INITIAL_INDEX_SIZE, null);
    _usedData = 0;
    _deletedKeys = 0;
  }
}

......

class _CompactLinkedIdentityHashMap<K, V> extends _HashFieldBase
    with
        MapMixin<K, V>,
        _LinkedHashMapMixin<K, V>,
        _HashBase,
        _IdenticalAndIdentityHashCode
    implements LinkedHashMap<K, V> {
  _CompactLinkedIdentityHashMap() : super(_HashBase._INITIAL_INDEX_SIZE);
}

class _CompactLinkedCustomHashMap<K, V> extends _HashFieldBase
    with MapMixin<K, V>, _LinkedHashMapMixin<K, V>, _HashBase
    implements LinkedHashMap<K, V> {
  final _equality;
  final _hasher;
  final _validKey;

  // TODO(koda): Ask gbracha why I cannot have fields _equals/_hashCode.
  int _hashCode(e) => _hasher(e);
  bool _equals(e1, e2) => _equality(e1, e2);

  bool containsKey(Object? o) => _validKey(o) ? super.containsKey(o) : false;
  V? operator [](Object? o) => _validKey(o) ? super[o] : null;
  V? remove(Object? o) => _validKey(o) ? super.remove(o) : null;

  _CompactLinkedCustomHashMap(this._equality, this._hasher, validKey)
      : _validKey = (validKey != null) ? validKey : new _TypeTest<K>().test,
        super(_HashBase._INITIAL_INDEX_SIZE);
}
```
它们都是一个普通的类，没有工厂构造方法，也就是说 `Map` 中的 `external factory Map();` 最终返回的最终实例类型为 `_InternalLinkedHashMap` 或 `_CompactLinkedCustomHashMap `

我们可以做一个简单的验证
```dart
final map = Map();
print(map.runtimeType);

// 打印结果
// _InternalLinkedHashMap<dynamic, dynamic>
```

我们来试着来实例化一个抽象类吧
```dart
abstract class Animal {
  void eat();

  void sleep() {
    print("睡觉");
  }

  factory Animal() {
    return Cat();
  }
}

 class Cat implements Animal {
  void eat() {
    print("吃");
  }

  void sleep() {
    print('睡');
  }
}
```

```dart
final animal = Animal();
print(animal.runtimeType); 

// 打印结果: Cat
```

可能会有同学要问了，这里用的是接口的方式，可以用继承的方式吗？
**很遗憾不行，因为在抽象类中定义了工厂构造方法后，在子类中不能定义除工厂构造方法外的其它构造方法了，会报错~**

总结一下：
> 抽象类无法直接创建实例，但是可以通过实现工厂构造方法来间接实现抽象类的实例化！

## 三、补充
那饶了这么一大圈，为什么不直接在声明的时候就给它实现了呢？🤔
这样做的好处就是：

> - 复用同一套API的声明
> - 可以针对不同的平台做不同的实现

而 `针对不同的平台做不同的实现` 这一点在下方给出的源码中可以看出

```dart
// flutter/bin/cache/dart-sdk/lib/io/file_system_entity.dart
abstract class _FileSystemWatcher {
  external static Stream<FileSystemEvent> _watch(
      String path, int events, bool recursive);
  external static bool get isSupported;
}
```

```dart
// flutter/bin/cache/dart-sdk/lib/_internal/vm/bin/file_patch.dart

@patch
static Stream<FileSystemEvent> _watch(
    String path, int events, bool recursive) {
  if (Platform.isLinux) {
    return new _InotifyFileSystemWatcher(path, events, recursive)._stream;
  }
  if (Platform.isWindows) {
    return new _Win32FileSystemWatcher(path, events, recursive)._stream;
  }
  if (Platform.isMacOS) {
    return new _FSEventStreamFileSystemWatcher(path, events, recursive)
        ._stream;
  }
  throw new FileSystemException(
      "File system watching is not supported on this platform");
}
```

