---
title: iOS - LXFDrawBoard 多功能小画板
date: 2017-09-25 18:38:12
categories: "iOS"
tags:
  - iOS项目
  - Objective-C
---

<Excerpt in index | 首页摘要> 
将LXFDrawBoard拖入项目中，导入头文件LXFDrawBoard.h，需要什么笔刷可以在Brush文件夹中找到即可使用，具体使用方法可以参考Demo
+<!-- more -->
<The rest of contents | 余下全文>



# LXFDrawBoard
多功能小画板 GitHub: [Demo](https://github.com/LinXunFeng/LXFDrawBoard)

## Usage
> 将LXFDrawBoard拖入项目中，导入头文件LXFDrawBoard.h，需要什么笔刷可以在Brush文件夹中找到即可使用，具体使用方法可以参考Demo

**LXFDrawBoardDelegate**

返回需要添加的描述

```objc
- (NSString *)LXFDrawBoard:(LXFDrawBoard *)drawBoard textForDescLabel:(UILabel *)descLabel;
```

当添加或修改描述时调用

```objc
- (void)LXFDrawBoard:(LXFDrawBoard *)drawBoard clickDescLabel:(UILabel *)descLabel;
```



## 笔刷

### 2017–09-25 更新
橡皮擦 LXFEraserBrush
![LXFEraserBrush](https://github.com/LinXunFeng/LXFDrawBoard/raw/master/Screenshots/橡皮擦.gif)

***

铅笔 LXFPencilBrush
![LXFPencilBrush](http://upload-images.jianshu.io/upload_images/2144614-db7ee9b3b6dc7d3c.gif?imageMogr2/auto-orient/strip)

箭头 LXFArrowBrush
![LXFArrowBrush](http://upload-images.jianshu.io/upload_images/2144614-52f9e8c9b036669a.gif?imageMogr2/auto-orient/strip)

直线 LXFLineBrush
![LXFLineBrush](http://upload-images.jianshu.io/upload_images/2144614-17043979b48ec450.gif?imageMogr2/auto-orient/strip)

文本 LXFTextBrush
![LXFTextBrush](http://upload-images.jianshu.io/upload_images/2144614-9b9e9406b075d68a.gif?imageMogr2/auto-orient/strip)

矩形 LXFRectangleBrush

![LXFRectangleBrush](http://upload-images.jianshu.io/upload_images/2144614-d4bfbe94682acafb.gif?imageMogr2/auto-orient/strip)

马赛克 LXFMosaicBrush
![LXFMosaicBrush](http://upload-images.jianshu.io/upload_images/2144614-a0d920442de28de4.gif?imageMogr2/auto-orient/strip)

撤销与反撤销
![撤销与反撤销](http://upload-images.jianshu.io/upload_images/2144614-5e0390d0baff8047.gif?imageMogr2/auto-orient/strip)