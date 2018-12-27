---
title: iOS - 通过runtime获取某个类中所有的变量和方法
date: 2017-09-12 08:32:11
categories: "iOS"
tags:
  - iOS
  - Objective-C
---

<Excerpt in index | 首页摘要> 

苹果官方的类中只提供给我们一小部分成员变量和方法,但有时候我们需要的恰好就没有提供,这样就会令开发人员十分懊恼了,那怎样才能获取该类中所有的变量及方法,用来查找是否有相对应的变量和方法呢?
我们可以使用苹果自带的 运行时(runtime) 来获取
+<!-- more -->
<The rest of contents | 余下全文>

苹果官方的类中只提供给我们一小部分成员变量和方法,但有时候我们需要的恰好就没有提供,这样就会令开发人员十分懊恼了,那怎样才能获取该类中所有的变量及方法,用来查找是否有相对应的变量和方法呢?
我们可以使用苹果自带的 运行时(runtime) 来获取

## 运行时(Runtime):
- 苹果官方一套[C语言](http://lib.csdn.net/base/c)库 
- 能做很多底层操作(比如访问隐藏的一些成员变量\成员方法....)

以下以 UITextField 为例

### 一. 包含运行时头文件
```objc
#import <objc/runtime.h>  
```
### 二. 获取所有的成员变量
```objc
unsigned int count = 0;
    
// 拷贝出所胡的成员变量列表
Ivar *ivars = class_copyIvarList([UITextField class], &count);
    
for (int i = 0; i<count; i++) {
    // 取出成员变量
    Ivar ivar = *(ivars + i);
        
    // 打印成员变量名字
    LXFLog(@"%s", ivar_getName(ivar));
        
    // 打印成员变量的数据类型
    LXFLog(@"%s", ivar_getTypeEncoding(ivar));
}
    
// 释放
free(ivars);
```
Swift的写法如下
```swift
var count: UInt32 = 0
let ivars = class_copyIvarList(UIViewController.self, &count)!
for i in 0..<count {
    let namePoint = ivar_getName(ivars[Int(i)])!
    let name = String(cString: namePoint)
    print(name)
}

```
### 三. 获取所有的成员方法
// 下面的UITextField改为你想获取所有属性的类名
// methCount: 这个类所有属性的个数
```objc
unsigned int methCount = 0;
Method *meths = class_copyMethodList([UITextField class], &methCount);
    
for(int i = 0; i < methCount; i++) {
        
    Method meth = meths[i];
        
    SEL sel = method_getName(meth);
        
    const char *name = sel_getName(sel);
        
    NSLog(@"%s", name);
}
    
free(meths);
```
最后,通过KVC的方式给相应的成员变量赋值即可!
如:
```objc
// 修改点位文字颜色
UILabel *placeholderLabel = [self valueForKeyPath:@"_placeholderLabel"];
placeholderLabel.textColor = [UIColor redColor];
// 或者这样
[self setValue:[UIColor grayColor] forKeyPath:@"_placeholderLabel.textColor"];
```