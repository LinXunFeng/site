---
title: 解决Transporter一直卡正在验证的问题
date: 2020-03-12 22:16:00
categories: "iOS"
tags:
  - iOS

---

<Excerpt in index | 首页摘要> 

苹果的上传应用工具 `Transporter` 虽然挺好用，但是估计也不少人跟我一样遇到过这样的问题，就是一直卡在 `正在验证`，不采取点措施估计能一直卡下去~

+<!-- more -->

<The rest of contents | 余下全文>

> 苹果的上传应用工具 `Transporter` 虽然挺好用，但是估计也不少人跟我一样遇到过这样的问题，就是一直卡在 `正在验证`，不采取点措施估计能一直卡下去~

![正在验证APP](https://linxunfeng.github.io/images/2020/03/解决Transporter一直卡正在验证的问题/正在验证APP.jpg)

其实原因很简单，就是 `/User/当前登录用户/Library/Caches/com.apple.amp.itmstransporter` 这个目录里的文件不全，一直处于下载更新的状态。

## 解决方案

### 方案一
科学上网前提下，在终端下执行  `Transporter` 包内的 `iTMSTransporter`，
```shell
/Applications/Transporter.app/Contents/itms/bin/iTMSTransporter
```
因为国外服务器（`contentdelivery.itunes.apple.com:443`）对我们来说会很慢，所以这个过程最好弄下科学环境。

直到出现这个命令说明界面就可以了
![](https://linxunfeng.github.io/images/2020/03/解决Transporter一直卡正在验证的问题/iTMSTransporter.png)

### 方案二
适用人群
-  没有科学环境
- 速度要求高的
- 比较懒的

可以使用 `transporter_fix`
GitHub地址：[https://github.com/LinXunFeng/transporter_fix](https://github.com/LinXunFeng/transporter_fix)
执行文件下载地址：[点我下载](https://github.com/LinXunFeng/transporter_fix/releases)，下载后双击运行即可。

![](https://linxunfeng.github.io/images/2020/03/解决Transporter一直卡正在验证的问题/transporter_fix.png)

比较懒的朋友看到这就可以了，要求速度快的就继续往下看

**重点**

- `transporter_fix` 默认是从 `github` 上下载 `com.apple.amp.itmstransporter.zip` 到同目录级别的 `files` 目录下。
- 如果 `files` 目录下已存在 `com.apple.amp.itmstransporter.zip` ，则不会重新下载。

所以，如果当前正常网络对 `github` 不给力，可以到[这里](https://www.lanzous.com/b0aqkmhpg)，找个离当前日期最近的压缩包，下载下来后改名为 `com.apple.amp.itmstransporter.zip`，存放 `files` 目录下即可

![](https://linxunfeng.github.io/images/2020/03/解决Transporter一直卡正在验证的问题/itmstransporter压缩包存放位置.png)


