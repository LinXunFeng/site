---
title: iOS 组件化开发（三）：加载资源文件
date: 2018-04-06 19:52:11
categories: "iOS"
tags:
  - iOS
  - Git
  - GitHub
  - Cocoapods
  - 组件化
---

<Excerpt in index | 首页摘要> 

经过前两篇文章的学习，相信对组件化开发有了大致的了解，那我们这篇文章就来讲讲资源文件的加载吧

+<!-- more -->
<The rest of contents | 余下全文>


> 经过前两篇文章的学习，相信对组件化开发有了大致的了解，那我们这篇文章就来讲讲资源文件的加载吧

这里我新建了一个LXFMain组件库，主要是用来显示TabBar的玩意，然后再进行组件化抽离出来，其中的过程这里不再赘述，还没了解过的同学建议先阅读下这两篇文章吧

[iOS 组件化开发（一）：远程私有库的基本使用](http://linxunfeng.top/2018/04/06/iOS-组件化开发（一）：远程私有库的基本使用/)

[iOS 组件化开发（二）：远程私有库的更新与子库](http://linxunfeng.top/2018/04/06/iOS-组件化开发（二）：远程私有库的更新与子库/)

这里跟之前不一样的地方在于多了图片资源，组件的核心代码放在Classes文件夹中，而图片我们则存放于Assets目录下，如图所示 

![存放位置](/images/2018/04/iOS 组件化开发（三）：加载资源文件/存放位置.png)

### 一、修改Spec
将关于资源加载的注释去掉
```
s.resource_bundles = {
 # 'LXFMain' => ['LXFMain/Assets/*.png']
 'LXFMain' => ['LXFMain/Assets/*']
}
```

回到LXFMain的模板库，我们进行一次本地的安装和测试(`pod install`)

![](/images/2018/04/iOS 组件化开发（三）：加载资源文件/修改Spec-podInstall.png)

可以看到，图片资源也安装进来了，但是运行的效果如下图，图片并不能成功加载出来

![没有图标](/images/2018/04/iOS 组件化开发（三）：加载资源文件/没有图标.png)

## 二、修改加载资源代码
这是当前加载图片的相关代码
```
[UIImage imageNamed:@"图片名称"];
```

![show in finder](/images/2018/04/iOS 组件化开发（三）：加载资源文件/showInFinder.png)

右击显示包内容
![LXFMain.framework](/images/2018/04/iOS 组件化开发（三）：加载资源文件/LXFMain.framework.png)

图片就在这个`LXFMain.bundle`里面(这里就不截图看了)，这里主要是让大家对这个目录结构有个了解

我们对`imageNamed`进行跳转到定义操作
![imageNamed](/images/2018/04/iOS 组件化开发（三）：加载资源文件/imageNamed.png)

```
// load from main bundle
```
可以看到，官方注释着`imageNamed`加载的是main bundle中的资源,mainBundle的位置如下图
![mainBundle](/images/2018/04/iOS 组件化开发（三）：加载资源文件/mainBundle.png)

这样当然就无法加载到图片啦，我们需要让它加载自己当前所在bundle里的图片 ，所以加载图片的代码需要进行修改
```objc
NSString *normalImgName = @"个人@2x.png";
NSBundle *curBundle = [NSBundle bundleForClass:self.class]; // 获取当前bundle
NSString *normalImgPath = [curBundle pathForResource:normalImgName ofType:nil inDirectory:@"LXFMain.bundle"];
UIImage *normalImage = [UIImage imageWithContentsOfFile:normalImgPath];
```

但是直接写`LXFMain.bundle`并不好，不可控，所以还需要改进一下：
```
NSString *normalImgName = [NSString stringWithFormat:@"%@@2x.png", normalImg];
NSBundle *curBundle = [NSBundle bundleForClass:self.class];
//  *********** 重点 ***********   //
NSString *curBundleName = curBundle.infoDictionary[@"CFBundleName"];
NSString *curBundleDirectory = [NSString stringWithFormat:@"%@.bundle", curBundleName];
NSString *normalImgPath = [curBundle pathForResource:normalImgName ofType:nil inDirectory:curBundleDirectory];
//  ***************************   //
UIImage *normalImage = [UIImage imageWithContentsOfFile:normalImgPath];
```

![成功加载](/images/2018/04/iOS 组件化开发（三）：加载资源文件/成功加载.png)

## 三、聊聊xib
Xib的加载也是如此
```
NSBundle *curBundle = [NSBundle bundleForClass:self.class];
LXFCenterView *centerView = (LXFCenterView *)[curBundle loadNibNamed:@"LXFCenterView" owner:nil options:nil].firstObject;
centerView.frame = CGRectMake(30, 140, 200, 100);
[self.view addSubview:centerView];
```
不过xib中值得一提的是，如果是直接在xib中拖入一个imageView控件来设置图片的加载，我们则需要在图片名字前加上当前bundle名称
```
LXFMain.bundle/个人
```

这里除了当前xib要加载的图片不属于mainBundle这个原因之外，还有一点就是xib文件与bundle存放位置属于同一级别，故直接使用相对路径的方式，在图片名字前加上bundle名称即可。


![同一目录级别](/images/2018/04/iOS 组件化开发（三）：加载资源文件/同一目录级别.png)


![xib上的操作](/images/2018/04/iOS 组件化开发（三）：加载资源文件/xib上的操作.png)

虽然无法在xib上直接看到效果，不过确实是有效的
![xib成功显示图片](/images/2018/04/iOS 组件化开发（三）：加载资源文件/xib成功显示图片.png)

## 四、遇到的小问题
```
[!] Unable to find a pod with name, author, summary, or description matching `lxfmain`
```
我做完一切操作后发现搜索报上面那个错，解决方案是删除本地索引文件，然后再搜索一遍，系统会自动帮你再生成一切本地索引文件，然后就搞定了～
```
rm -rf ~/Library/Caches/CocoaPods/search_index.json 
pod search lxfmain
```

