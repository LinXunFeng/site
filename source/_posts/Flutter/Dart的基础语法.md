---
title: Dart的基础语法
date: 2021-08-07 08:58:22
categories: "Flutter"
tags:
  - Dart
---



<Excerpt in index | 首页摘要> 

Dart的基础语法总结：变量/常量、数据类型、集合、函数、运算符、流程控制、类、枚举、泛型、库的使用

+<!-- more -->

<The rest of contents | 余下全文>



> 相关代码存放于：[https://github.com/LinXunFeng/DartStudy](https://github.com/LinXunFeng/DartStudy)



## 一、变量/常量



### 明确声明

```dart
// 明确声明
String name = 'lxf';
int age = 18;
double height = 1.8;
print('$name, $age, $height'); // lxf, 18, 1.8
```



### 类型推导

#### var

`runtimeType` : 用于获取变量当前的类型

```dart
var count = 1; // count的类型为int
print(count.runtimeType); // int
// count = "abc"; // 报错，无法为int类型赋值一个字符串
```

#### dynamic

> `dynamic` 声明的变量可以赋值为任意类型
>
> 在开发中一般不这种做，因为会带来潜在的危险

```dart
dynamic type = 'lxf';
print(type.runtimeType); // String

type = 18;
print(type.runtimeType); // int

// type.abc(); // 编译期不会报错，运行时找不到方法报错
```

#### final & const

> `final` 和 `const` 都用来定义变量

```dart
final gender = 'male';
// gender = 'female' // 报错
const country = 'China';
// country = 'USA'; // 报错
```

区别： 

- `const` : 必须赋值，接收一个常量值（即编译期间就需要确定的值）
- `final` : 可以通过计算/函数动态获取值（即运行时能确定的值）

```dart
String getName() {
  return 'lxf';
}

main(List<String> args) {
	final myName = getName();
  // const myName = getName(); // 报错
}
```



## 二、数据类型

> - `int`: 整数
> - `double`: 浮点数

注：`int` 和 `double` 表示的范围不是固定的，具体取决于运行 `Dart` 的平台



### 整型

```dart
// 整型
int age = 18;
int hexAge = 0x12;
print(age); // 18
print(hexAge); // 18
```

### 浮点型

```dart
// 浮点类型 double
double height = 1.8;
print(height); // 1.8
```

### 布尔类型

```dart
var isFlag = true;
print('$isFlag ${isFlag.runtimeType}'); // true bool
```

- Dart中没有非0即真或非空即真

```dart
var msg = 'lxf';
if (msg) { // 报错 Conditions must have a static type of 'bool'.
  print(msg);
}
```

### 字符串

```dart
// 字符串
var s1 = 'hello lxf';
var s2 = "hello lxf";
```

多行字符串

```dart
// 多行字符串
var s3 = '''
  hello
  hey
  lxf''';
```

字符串拼接

- 字符串和其他变量或表达式拼接: 使用 `${expression}`, 如果表达式是一个标识符, 那么 `{}` 可以省略

```dart
var s4 = s1 + s2;
print(s4); // hello lxfhello lxf

var s5 = 's1 = $s1 and s4 = ${s1 + s2}';
print(s5); // s1 = hello lxf and s4 = hello lxfhello lxf
```



## 三、集合

### List

> - 使用 `[]` 字面量

```dart
var letters = ['a', 'b', 'c', 'd'];
print('$letters ${letters.runtimeType}'); // [a, b, c, d] List<String>

List<int> numbers = [1, 2, 3, 4];
print('$numbers ${numbers.runtimeType}'); // [1, 2, 3, 4] List<int>
```

`List` 特有的操作

```dart
// 由于List的元素是有序的，所有它还提供了一个删除指定索引位置上元素的方法removeAt
// List根据index删除元素
numbers.removeAt(3);
print('$numbers'); // [2, 3, 4]
```

### Set

> - 使用 `{}` 字面量
> - 与 `List` 的两个不同点：`Set` 是无序的，且元素不重复

```dart
var lettersSet = {'a', 'b', 'c', 'd'};
print('$lettersSet ${lettersSet.runtimeType}'); // {a, b, c, d} _CompactLinkedHashSet<String>

Set<int> numbersSet = {1, 2, 3, 4};
print('$numbersSet ${numbersSet.runtimeType}'); // {1, 2, 3, 4} _CompactLinkedHashSet<int>
```

### Map

```dart
var infoMap1 = {'name': 'lxf', 'age': 18};
print('$infoMap1 ${infoMap1.runtimeType}');
// {name: lxf, age: 18} _InternalLinkedHashMap<String, Object>

Map<String, Object> infoMap2 = {'height': 1.8, 'country': 'China'};
print('$infoMap2 ${infoMap2.runtimeType}');
// {height: 1.8, country: China} _InternalLinkedHashMap<String, Object>
```

`Map` 特有的操作

```dart
// 1.根据key获取value
print(infoMap1['name']); // lxf

// 2.获取所有的entries
print('${infoMap1.entries} ${infoMap1.entries.runtimeType}');
// (MapEntry(name: lxf), MapEntry(age: 18)) MappedIterable<String, MapEntry<String, Object>>

// 3.获取所有的keys
print('${infoMap1.keys} ${infoMap1.keys.runtimeType}');
// (name, age) _CompactIterable<String>

// 4.获取所有的values
print('${infoMap1.values} ${infoMap1.values.runtimeType}');
// (lxf, 18) _CompactIterable<Object>

// 5.判断是否包含某个key或者value
print('${infoMap1.containsKey('age')} ${infoMap1.containsValue(18)}');
// true true

// 6.根据key删除元素
infoMap1.remove('age');
print('${infoMap1}'); // {name: lxf}
```

### 集合共有的操作

```dart
// 长度
print(letters.length); // 4
print(lettersSet.length); // 4
print(infoMap1.length); // 2

// 添加、删除、包含元素
numbers.add(5);
numbersSet.add(5);
print('$numbers $numbersSet'); // [1, 2, 3, 4, 5] {1, 2, 3, 4, 5}

numbers.remove(1);
numbersSet.remove(1);
print('$numbers $numbersSet'); // [2, 3, 4, 5] {2, 3, 4, 5}

print(numbers.contains(2)); // true
print(numbersSet.contains(2)); // true
```



## 四、函数

>  函数也是对象，类型为 `Function`



### 函数定义

> 注意点：`Dart` 不支持函数重载！（函数重载：函数名称相同，参数不同）

```dart
返回值 函数的名称(参数列表) {
  函数体
  return 返回值
}
```

普通函数

```dart
num sum1(num num1, num num2) {
  return num1 + num2;
}

print(sum1(1, 2)); // 3
```

省略了类型注释也可以正常工作，但是不建议这么做

```dart
sum2(num1, num2) {
  return num1 + num2;
}

print(sum2(1, 2)); // 3
```

如果函数的实现只有一个表达式，则可以使用箭头语法

```dart
sum3(num num1, num num2) => num1 + num2;

print(sum3(1, 2)); // 3
```



### 可选参数

#### 命名可选参数

> - 定义：命名可选参数: `{param1, param2, ...}`
>
> - 注意：
>   - 如果确定传入的参数不能为 `null`，则需要加 `required` 关键字
>   - 如果传入的参数可能为 `null`，则参数类型后面需要加 `?`

```dart
// 命名可选参数
printInfo1(String name, {int? age, double? height}) {
  print('name = $name age = $age height = $height');
}

// 确定参数不能为null，需要添加 required 关键字
// printInfo1(String name, {required int age, required double height}) {}
```

 参数默认值

```dart
printInfo3(String name, {int age = 18, double height = 1.8}) {
  print('name = $name age = $age height = $height');
}
```

调用

```dart
// 命名可选参数
printInfo1('lxf'); // name = lxf age = null height = null
printInfo1('lxf', age: 18); // name = lxf age = 18 height = null
printInfo1('lxf', age: 18, height: 1.8); // name = lxf age = 18 height = 1.8
printInfo1('lxf', height: 1.8); // name = lxf age = null height = 1.8

// 参数默认值
printInfo3('lxf'); // name = lxf age = 18 height = 1.8
```

#### 位置可选参数

> - 定义：位置可选参数: `[param1, param2, ...]`
> - 注意：位置可选参数无法使用 `required` 关键字

```dart
// 位置可选参数
printInfo2(String name, [int? age, double? height]) {
  print('name = $name age = $age height = $height');
}
```

参数默认值

```dart
printInfo4(String name, [int age = 18, double height = 1.8]) {
  print('name = $name age = $age height = $height');
}
```

调用

```dart
// 位置可选参数
printInfo2('lxf'); // name = lxf age = null height = null
printInfo2('lxf', 18); // name = lxf age = 18 height = null
printInfo2('lxf', 18, 1.8); // name = lxf age = 18 height = 1.8

// 参数默认值
printInfo4('lxf'); // name = lxf age = 18 height = 1.8
```



## 五、运算符

### 常见的运算符

```dart
var num = 10;
print(num / 3); // 除法：3.3333333333333335
print(num ~/ 3); // 整除：3
print(num % 3); // 取模：1
```

### 赋值运算符

> `??=` : 当变量为 `null` 时，使用后面的内容进行赋值

```dart
// 赋值运算符
var name = null;
name ??= 'lxf';
print(name); // lxf

var name1 = 'lin';
name1 ??= 'lxf';
print(name1); // lin
```



### 条件运算符

> `??` : 当变量为 `null` 时，则使用后面的值，否则使用变量的值

```dart
var temp = null;
var other = temp ?? 'lxf';
print(other); // lxf
```



### 级联

> `..` : 可以对对象进行连续操作

```dart
class Person {
  String name;

  Person(this.name);

  void run() {
    print("$name is running");
  }

  void eat() {
    print('$name is eating');
  }

  void swim() {
    print('$name is swimming');
  }
}

main(List<String> args) {
  final p1 = Person('lxf');
  p1.run();
  p1.eat();
  p1.swim();

  final p2 = Person('lin')
    ..run()
    ..eat()
    ..swim();
}
```

## 六、流程控制



### `if` 和 `else`

>  不支持非空即真或非0即真，必须是明确的 `bool` 类型

```dart
var isHot = true;
if (isHot) {
  print('热门');
} else {
  print('冷门');
}
```

### `for`

```dart
// for
for (var i = 0; i < 10; i++) {
  print(i);
}

// for in 遍历集合
var names = ['lxf', 'lin'];
for (var name in names) {
  print(name);
}
```

### `while`

```dart
var index = 0;
while (index < 5) {
  print('index -- $index');
  index++;
}
```

### `do-while`

```dart
var index = 0;
do {
  print('index -- $index');
  index++;
} while (index < 10);
```



### `break`

> - 退出循环

```dart
var i = 1;
while (i <= 10) {
  if (i % 5 == 0) {
    print("满足条件，退出循环");
    break;
  }
  // 继续下一个循环
  i++;
}
```



### `continue`

> - 终止当前迭代并开始后续迭代
> - 不会退出循环

```dart
var num = 0;
var count = 0;

for (num = 0; num <= 20; num++) {
  if (num % 2 == 0) {
    continue;
  }
  count++;
}
print("累加次数: $count"); // 累加次数: 10
```



### `switch-case`

```dart
var direction = 'east';
switch (direction) {
  case 'east':
    print('东面');
    break;
  case 'south':
    print('南面');
    break;
  case 'west':
    print('西面');
    break;
  case 'north':
    print('北面');
    break;
  default:
    print('其他方向');
}
```



## 六、类

类的定义如下：

```dart
class 类名 {
  类型 成员名;
  返回值类型 方法名(参数列表) {
    方法体
  }
}
```

### `late` 关键字

> 作用：显式声明一个非空的变量，但不初始化，延迟变量的初始化

默认的构造函数是无参构建函数，这意味着属性无法从默认构建函数得到初始化，这时有两个选择，

选择一：自定义一个构造函数对属性进行初始化

选择二：使用 `late` 关键字修饰属性，延迟变量的初始化



加上 `late` 关键字可以通过静态检查，但是会带来运行时风险，你需要保证属性在使用前有进行初始化！

```dart
class Person {
  late String name;
  late int age;
}

main(List<String> args) {
  var p1 = Person();
  p1.name = 'lxf';
  print(p1.name); // lxf
  print(p1.age); // 报错：LateInitializationError: Field 'age' has not been initialized.
}
```



### 构造方法

#### 普通构造方法

当类中没有指定构造方法时，默认会有一个无参的构造方法

```dart
class Person {
  String? name;
  int? age;
}


main(List<String> args) {
  var p1 = Person(); // 默认的构造方法
}
```

可以根据自已的需求，定义自已的构建方法，不过需要注意的是：

当定义了自已的构建方法，则默认的无参构造方法会失效，无法使用！

```dart
class Person {
  String name;
  int age;

  Person(this.name, this.age);
}
```



#### 命名构造方法

由于 `Dart` 不支持方法重载，所以没办法创建相同名称的构造方法，如果希望能实现更多的构造方法，这时就需要使用命名构造方法

```dart
class Person {
  String? name;
  int? age;

  Person.withArgs(String name, int age) {
    this.name = name;
    this.age = age;
  }
}


main(List<String> args) {
  // var p1 = Person(); // 默认构造方法失效
  var p1 = Person.withArgs('lxf', 18);
  print(p1.name);
  print(p1.age);
}
```



#### 重定向构造方法

> 在一个构造方法中，调用另一个构造方法进行初始化

```dart
class Person {
  String name;
  int age;

  Person(this.name, this.age);

  // 重定向构造方法
  Person.from(String name): this(name, 0);
}
```



#### 常量构造方法

> 使用 `const` 修饰构造方法，当传入相同参数时，可以确保创建出来的对象是相同的
>
> 注意点：
>
> - 拥有常量构造方法的类中，所有的成员变量必须是final修饰的
> - 为了创建出相同的对象，在使用常量构造方法时，需要使用 `const` 关键字



```dart
class Person {
  // 拥有常量构造方法的类中，所有的成员变量必须是final修饰的
  final String name;

  const Person(this.name);
}

main(List<String> args) {
  var p1 = const Person('lxf');
  var p2 = const Person('lxf');
  var p3 = const Person('lxf1');
  
  const p4 = Person4('lxf');
  
  // 判断是不是同一个对象
  print(identical(p1, p2)); // true
  print(identical(p1, p3)); // false
  print(identical(p1, p4)); // true
}
  
```



#### 工厂构造方法



```dart
class Person {
  late String name;

  static final Map<String, Person> _cache = <String, Person>{};

  // 工厂构造方法
  factory Person(String name) {
    var p = _cache[name];
    if (p == null) {
      p = Person._internal(name);
      _cache[name] = p;
    }
    return p;
  }

  Person._internal(this.name);
}

main(List<String> args) {
  var p1 = Person5('lxf');
  var p2 = Person5('lxf');
  // 判断是不是同一个对象
  print(identical(p1, p2)); // true
}
```



注意：当工厂构造方法与默认构造方法重名时，无法被继承，会报错

```shell
The default constructor of superclass 'Father' (called by the implicit default constructor of 'Son') must be a generative constructor, but factory found.

父类的构造器必须是一个生成的构造器，而不能是工厂构造器
```

如下方代码所示

```dart
class Father {

  factory Father() {
    return Father._internal();
  }

  Father._internal();
}

class Son extends Father {} // 编译报错
```

需要添加一个生成构造器，并将工厂构造器改为命名构造器

```dart
class Father {

  Father();

  factory Father.factory() {
    return Father._internal();
  }
  Father._internal();
}

class Son extends Father {}
```



### `getter` 和 `setter`

> 当希望类中定义的属性不轻易给外部访问时，就可以用到 `getter` 和 `setter`

```dart
class Marker {
  String _color;

  String get color => _color;

  set color(String color) {
    _color = color;
  }

  Marker(this._color);
}

main(List<String> args) {
  final m = Marker('red');
  m.color = 'blue';
  print(m.color); // blue
}
```



### 继承

> 继承使用 `extends` 关键字，子类中使用 `super` 来访问父类

```dart
class Animal {
  String name;

  Animal(this.name);

  eat() {
    print("吃吃吃");
  }
}

class Dog extends Animal {
  Dog(String name) : super(name);
}

main(List<String> args) {
  var dog = Dog("Spike");
  dog.eat(); // 吃吃吃
  print(dog.name); // Spike
}
```



重写父类的方法，`@override` 是注释，用来标明当前为重写的方法，可有可无，但一般保留着

```dart
class Dog extends Animal {
  Dog(String name) : super(name);

  // 重写父类的eat方法
  @override
  eat() {
    print("$name 在吃");
  }
}
```



### 抽象类

> 抽象类：使用 `abstract` 声明的类，用于定义通用接口，无法被实例化，方法可以没有实现，该方法称为抽象方法，交由子类去实现

```dart
abstract class Building {
  getHeight();

  getArea() {
    print("hello");
  }
}

class Tower extends Building {
  @override
  getHeight() {
    print("55m");
  }
}

main(List<String> args) {
  var tower = Tower();
  tower.getArea(); // hello
  tower.getHeight(); // 55m
}
```



### 隐式接口

- `Dart` 没有关键字可以用来声明接口，所有的类都是隐式接口
- 当一个类被当作接口使用时，那么实现这个接口的类必须实现接口中的所有方法

- 实现接口使用 `implements` 关键字
- 被实现的接口方法不可以使用 `super` 调用接口中的方法



```dart
class Runner {
  run() {
    print("running");
  }
}

abstract class Fly {
  fly();
}

class SuperPerson implements Runner, Fly {
  @override
  run() {
    // 实现接口的方法不可以调用super
    // 报错：The method 'run' is always abstract in the supertype.
    // super.run();
    print('自由自在的奔跑');
  }

  @override
  fly() {
    print('起飞');
  }
}
```



### Mixin混入

`Dart` 不支持多继承，使用接口时需要实现接口中的所有方法，而我们需要在当前类可以复用其它类中的实现时就需要用到 `Mixin混入` 的方式。



> - 定义：To implement a mixin, create a class that extends Object and declares no constructors. Unless you want your mixin to be usable as a regular class, use the mixin keyword instead of class. 
> - 取自官方链接：https://dart.dev/guides/language/language-tour#adding-features-to-a-class-mixins
> - 大致意思：创建一个继承自 `Object` 的类，且没有声明构造函数，则该类就可被混入。除非你希望你的混入类可以被当作常规类使用，否则请使用 `mixin` 关键字，而不是 `class`



特性：

- `mixin` 关键字声明混入（非必要，也可以用 `class`）
- 混入类继承自 `Object`，且不可以有构造方法

- `on` 关键字限定指定类的子类可以使用当前混入（不加默认限定为 `Object`）
- 使用 `mixin` 后无法被继承

- 使用 `with` 进行混入
- 父类与混入的方法重名时，使用的是混入的方法



```dart
// mixin Runner {} // 可以不使用on，默认是 on Object

mixin Runner on Animal {
  run() {
    print('跑起来了');
  }
}

class Animal {
  run() {
    print('Animal run');
  }
}

class SuperPeople extends Animal with Runner {} // 必须继承 Animal 才可以混入 Runner

// 报错：'Runner' can't be mixed onto 'Object' because 'Object' doesn't implement 'Animal'.
// class Person with Runner {}

// 报错：Classes can only extend other classes.
// class Person extends Runner {} // Runner无法被继承

main(List<String> args) {
  var p = SuperPeople();
  p.run(); // 跑起来了（使用的是混入的方法，而不是父类中的）
}
```



### 当前类、父类与混入的实现优先级

> 总结：当前类 > 混入 > 父类

```dart
class A {
  void method() {
    print('I am A');
  }
}

class B {
  void method() {
    print('I am B');
  }
}

class C extends A with B {
  // void method() {
  //   print('I am C');
  // }
}

main(List<String> args) {
  var c = C();
  /**
   * C 有无自己的 method 方法实现：
   *  - 有：打印：I am C
   *  - 无：打印：I am B
   */
  c.method();
}
```



## 七、枚举

```dart
// 定义枚举
enum Color { red, blue, green }

main(List<String> args) {
  print(Color.red); // Color.red

  print(Color.red.index); // 0 枚举的下标从0开始
  print(Color.blue.index); // 1

  print(Color.values); // [Color.red, Color.blue, Color.green]

  var color = Color.green;
  switch (color) {
    case Color.red:
      print("红");
      break;
    case Color.blue:
      print("蓝");
      break;
    case Color.green:
      print("绿");
      break;
  }
}
```

## 八、泛型



###  `List` 使用泛型

```dart
// List
var myList = ['lxf', 18];
print(myList.runtimeType); // List<Object>

// List使用泛型，限制类型
var numList = <int>[10, 20];
List<String> nameList = ["lxf", "lin"]; // 一般使用这种

// 报错: The element type 'int' can't be assigned to the list type 'String'.
// List<String> nameList = ["lxf", "lin", 18];
```

### `Map` 使用泛型

```dart
// Map
var msgMap = {1: 'one', 'name': 'lxf', 'age': 18};
print(msgMap.runtimeType); // _InternalLinkedHashMap<Object, Object>

// Map使用泛型，限制类型
var infoMap = <String, String>{'name': 'lxf', 'age': '18'};
// Map<String, String> profileMap = {'name': 'lxf', 'age': '18'}; // 一般使用这种

// 报错：The element type 'int' can't be assigned to the map value type 'String'.
// Map<String, String> profileMap = {'name': 'lxf', 'age': 18};
```

### 类定义泛型

将类的属性以泛型做为类型声明，由外部来决定最终的类型

```dart
class Point<T> {
  T x;
  T y;

  Point(this.x, this.y);
}

main(List<String> args) {
  Point p = Point<double>(10.5, 20.5);
  print(p.x.runtimeType); // double
}
```

如果希望属性的类型只能是数字，可以使用 `extends` 来限制泛型 `T` 只能为 `num` 的子类

```dart
class Size<T extends num> {
  T width;
  T height;

  Size(this.width, this.height);
}

main(List<String> args) {
  Size s = Size<int>(20, 10);
  // 报错：A value of type 'Size<String>' can't be assigned to a variable of type 'Size<num>'.
  // Size s1 = Size<String>("20", "10"); 
}
```



## 九、库的使用

### 库的导入 

#### `Dart` 标准库

> `Dart` 标准库的导入前缀为 `dart:`

```dart
import 'dart:io';
import 'dart:html';
import 'dart:math';
```

#### 自定义库

`libs/math_utils.dart` 文件的内容：

```dart
num sum(num num1, num num2) {
  return num1 + num2;
}

num multiply(num num1, num num2) {
  return num1 * num2;
}
```

使用

```dart
// 导入自定义库
import 'libs/math_utils.dart';

main(List<String> args) {
  print(sum(10, 20)); // 30
  print(multiply(10, 20)); // 200
}
```

#### 第三方库

在项目目录下创建 `pubspec.yaml` 文件，在 `dependencies` 下添加需要依赖的第三方的库名与版本

> 第三库的使用说明可在 [https://pub.flutter-io.cn](https://pub.flutter-io.cn/) 上查找

```dart
name: dartStudy
description: this is lxf's study project
dependencies:
  dio: ^4.0.0
environment:
  sdk: ">=2.10.0 <3.0.0"
```

填写好依赖后，在终端下执行 `dart pub get` 命令，终端会自动拉取对应版本的第三方文件，之后在代码中就可以导入并使用

```dart
// 导入第三方库
import 'package:dio/dio.dart';

main(List<String> args) {
  getHttp();
}

void getHttp() async {
  try {
    var response = await Dio().get('http://www.google.cn');
    print(response);
  } catch (e) {
    print(e);
  }
}
```



#### 延迟加载

> - 作用：在需要的时候才进行加载
> - 好处：减少 `App` 的启动时间
> - 使用：使用 `deferred as`

```dart
// 延迟加载
import 'package:dio/dio.dart' deferred as diolib;
```

在使用延迟加载的库中，需要调用 `loadLibrary` 以加载声明为延迟加载的库

```dart
main(List<String> args) async {
  // 调用loadLibrary方法来加载延迟加载的库
  await diolib.loadLibrary();
  diolib.Dio().get('http://www.google.cn');
}
```



#### 给库起别名

> 一般在与别的库的内容冲突时使用

```dart
import 'libs/math_utils.dart' as MathUtils; // 别名

main(List<String> args) {
  print(MathUtils.sum(10, 20)); // 30
  print(MathUtils.multiply(10, 20)); // 200
}
```



#### 指定库内容的显示与隐藏

```dart
import 'libs/math_utils.dart' show sum, multiply; // show: 显示指定的内容（即部分引入）
import 'libs/math_utils.dart' hide sum; // hide: 隐藏指定的内容
```



### `library`、`part` 和 `part of`



> 作用：
>
> - `library`：定义库名
> - `part`：关联分库文件
> - `part of`：声明隶属哪个库，使用的是 `library` 定义的库名
>
> 
>
> 特点：
>
> - `library`、`part` 和 `part of` 都必须定义在文件的第一行，否则会报错
> - 主库可以访问分库中的私有成员
> - 如果需要导入其它库，只能在主库中导入，无法在分库中导入，且 `import` 只能放在 `library` 和 `part` 之间 
> - 如果外部需要使用到分库的内容，只需要导入主库即可



文件目录结构：

```shell
├── 16_库的定义.dart
└── lib
    └── part
        ├── fitsize.dart
        ├── fitsize_part1.dart
        └── fitsize_part2.dart
```



`fitsize.dart` 文件内容

```dart
// 定义库名
library fitsize;

// 关联分库
part 'fitsize_part1.dart';
part 'fitsize_part2.dart';

void fit() {
  // 可以访问分库下的私有成员

  part1PublicMethod(); // part1 public method
  _part1PrivateMethod(); // part1 private method

  part2PublicMethod(); // part2 public method
  _part2PrivateMethod(); // part2 private method
}
```

`fitsize_part1.dart` 文件内容

```dart
part of fitsize; // 声明为主库的一部分

part1PublicMethod() {
  print("part1 public method");
}

_part1PrivateMethod() {
  print("part1 private method");
}
```

`fitsize_part2.dart` 文件内容

```dart
part of fitsize; // 声明为主库的一部分

part2PublicMethod() {
  print("part2 public method");
}

_part2PrivateMethod() {
  print("part2 private method");
}
```

这样三个 `dart` 文件就组成了 `fitsize` 库，如果外部需要使用到分库的内容，只需要导入主库即可。



如果需要导入其它库，只能在主库中导入，无法在分库中导入，且 `import` 只能放在 `library` 和 `part` 之间，否则会报错。

```dart
// fitsize.dart 主库文件

// 定义库名
library fitsize;

import 'dart:io'; // 导入其它库，只能放在这个位置

// 关联分库
part 'fitsize_part1.dart';
part 'fitsize_part2.dart';
```



### `export`



> 官方不再推荐使用 `part` 关键字，而是推荐使用 `export`
>
> - 使用：在一个 `dart` 文件里使用 `export` 关键字，将其它库文件导出
> - 作用：这样外部在使用时只需要导入一个文件即可使用完整的库内容



文件目录结构：

```shell
├── 16_库的定义.dart
└── lib
    └── export
        ├── lxf_utils.dart
        └── source
            ├── lxf_file_util.dart
            └── lxf_math_util.dart
```



`lxf_utils.dart` 文件内容：

```dart
// 定义库名
library lxf_utils;

// 导出所有的库
export './source/lxf_math_util.dart';
export './source/lxf_file_util.dart';
```



`lxf_math_util.dart` 文件内容：

```dart
num sum(num num1, num num2) {
  return num1 + num2;
}
```



`lxf_file_util.dart` 文件内容：

```dart
writeFile(String name, String dir_path) {
  print('写入文件 $dir_path$name ');
}
```



导入并调用

```dart
import 'lib/export/lxf_utils.dart' as utils;

main(List<String> args) {
  print(utils.sum(1, 2)); // 3
  utils.writeFile('lxf.jpg', '/lxf/Desktop/'); // 写入文件 /lxf/Desktop/lxf.jpg
}
```
