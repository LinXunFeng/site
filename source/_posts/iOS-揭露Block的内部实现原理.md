---
title: iOS - 揭露Block的内部实现原理
date: 2017-09-12 09:42:47
categories: "iOS"
tags:
  - iOS
  - Objective-C
---

<Excerpt in index | 首页摘要> 

想必大家对block都很熟悉了，虽然都会用，但是你真的知道它的原理吗？比如为什么要加上__block，这个修饰符到底有什么用？不加会有什么后果？block又是如何实现的等等。。。该篇文章就为大家揭晓关于Block的实现原理~

+<!-- more -->

<The rest of contents | 余下全文>

> 想必大家对block都很熟悉了，虽然都会用，但是你真的知道它的原理吗？比如为什么要加上__block，这个修饰符到底有什么用？不加会有什么后果？block又是如何实现的等等。。。该篇文章就为大家揭晓关于Block的实现原理~

## 抛砖引玉
先给出问题，大家思考下结果吧，如果分别调用以下两个方法，结果如何？
```objc
void blockFunc1()
{
    int num = 100;
    void (^block)() = ^{
        NSLog(@"num equal %d", num);
    };
    num = 200;
    block();
}
```
```objc
void blockFunc2()
{
    __block int num = 100;
    void (^block)() = ^{
        NSLog(@"num equal %d", num);
    };
    num = 200;
    block();
}
```
答案是
```
blockFunc1 : num equal 100
blockFunc2 : num equal 200
```
是不是有人答错了？再来两个函数。这两个的结果与blockFunc2一样，打印出来的 num 为 200
```objc
// 全局变量
int num = 100;
void blockFunc3()
{
    void (^block)() = ^{
        NSLog(@"num equal %d", num);
    };
    num = 200;
    block();
}
```
```objc
void blockFunc4()
{
    static int num = 100;
    void (^block)() = ^{
        NSLog(@"num equal %d", num);
    };
    num = 200;
    block();
}
```
> 疑问：
> 我们发现num做为局部变量时加上 _ _block 修饰符、num做为全局变量以及num为静态局部变量时在block中输出结果是一样的，皆为被修改之后的值，而做为局部变量并且未加上__block的num在block中输出的值却还是未赋值之前的值。这是为什么呢？探索这个问题我们就需要看看底层结构是如何实现的了

## 探索内部原理
Objective-C是一个全动态语言，它的一切都是基于runtime实现的！在运行时会将OC转换成C，我们可以利用这个来查看关于block在内部是如何实现的
新建一个Command Line Tool项目，将以上代码放入main.m中，如图


![main.m](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/1.png)

这里我们打开终端，cd到项目目录下，然后将用下面的命令将OC重写为C
```
clang -rewrite-objc main.m
```

![rewrite-objc](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/2.png)
这时我们可以发现当前目录下多了一个main.cpp文件，打开它并滚到最下面
![打开main.cpp](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/3.png)

![main.cpp](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/4.png)
这里我们可以看到blockFunc1的C语言实现方法

```c
void blockFunc1()
{
    int num = 100;
    void (*block)() = ((void (*)())&__blockFunc1_block_impl_0((void *)__blockFunc1_block_func_0, &__blockFunc1_block_desc_0_DATA, num));
    num = 200;
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
}
```
去掉类型转换
```c
void blockFunc1()
{
    int num = 100;
    // *************************重点句***********************
    void (*block)() = &__blockFunc1_block_impl_0(__blockFunc1_block_func_0, &__blockFunc1_block_desc_0_DATA, num));
    // *****************************************************
    num = 200;
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
}
```
这里我们可以看到
> block实际上是指向结构体的指针

该结构体为
![__blockFunc1_block_impl_0](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/5.png)

我们来看下带__block的blockFunc2

![blockFunc2](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/6.png)
在 blockFunc1 中，block指向了一个名为__blockFunc1_block_impl_0的结构体，并且在初始化时输入了三个参数(__blockFunc1_block_impl_0最后的flags有默认参数，所以可以不用传参)，第三个参数就是我们写的num，与blockFunc2相比较，这里的num并没有带*号，所以说在这里它只是传值而非传址，而下面的【num = 200;】也就没什么卵用了。这就是blockFunc2、blockFunc3与blockFunc4为什么能打印出num改变后的值，而blockFunc1不行的原因。

![](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/7.png)

在这里我们也可以看出：
> 编译器会将block的内部代码生成对应的函数

** SO **
> 我们总结下，block在内部会作为一个指向结构体的指针，当调用block的时候其实就是根据block对应的指针找到相应的函数，进而进行调用，并传入自身

## __block的实现

我们再来看看 _ _block，_ _block也被转换成了结构体，并含有5个变量
```
struct __Block_byref_num_0 {
  void *__isa;  // isa指针
__Block_byref_num_0 *__forwarding;  // 实例本身
 int __flags; 
 int __size;
 int num;  // 我们的num值
};
```

![](http://linxunfeng.github.io/images/2017/09/iOS - 揭露Block的内部实现原理/8.png)
图片对应着blockFunc2中的

```
__block int num = 100;
```
当创建num并用__block修饰的时候，会初始化这五个变量
当我们执行
```
num = 200;
```
对应着
```
(num.__forwarding->num) = 200;
```
上面刚刚提到过 _ _forwarding是实例本身，即类型结构体__Block_byref_num_0的&num，再找到对应的num变量，将其原来的100修改为200~~

到此，关于Block内部实现的揭晓也就到此结束了，希望本文能让你对block有更深的理解，感谢你耐心的阅读！