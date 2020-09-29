---
title: Flutter - 低版本在iOS14上遇到的问题与解决方案
date: 2020-09-29 00:00:00
categories: "Flutter"
tags:
  - Dart
  - Flutter
---

<Excerpt in index | 首页摘要> 

近期将测试机升级至 iOS14 ，测试使用 Flutter混合开发 的线上 APP，没发现什么问题，但是使用 Xcode 安装APP 的场景下，断开 Xcode 后再运行却闪退了。

+<!-- more -->

<The rest of contents | 余下全文>

## 一、概述

近期将测试机升级至 `iOS14` ，测试使用 `Flutter混合开发` 的线上 `APP`，没发现什么问题，但是使用 `Xcode` 安装`APP`的场景下，断开 `Xcode` 后再运行却闪退了。

以我们公司的 `APP` 测试的结果如下：

| APP来源 | 是否闪退                           | 模式    |
| ------- | ---------------------------------- | ------- |
| 线上    | 否                                 | release |
| 蒲公英  | 是                                 | debug   |
| Xcode   | 是（断开 `Xcode` 后再打开 `APP` ） | debug   |

### 问题原因

闪退的原因是原因 `Flutter SDK`,  `Flutter` 官方的更新速度也是快，对 `iOS14` 进行了说明：  [Flutter官网说明链接](https://flutter.dev/docs/development/ios-14)

大致意思就是说，如果我们在 `iOS14` 的真机上安装了 `debug模式` 编译出来的 `flutter` 应用，那么在断开编译安装连接后，将无法从桌面上打开该应用程序。

### 解决方案

1. 再次是使用 `Xcode` 或 `flutter run` 来运行。
2. 设置 `Flutter` 模块的编译模式为 `profile` 或 `release`

### 补充说明

1. 该闪退的情况只发生在真机，并且在模拟器运行的时候， `Flutter` 模块的编译模式需要为 `debug`， 如果设置了 `release`，编译将会报错。
2. 官方指出如果是纯 `Flutter` 可以直接使用 `master channel` 的 `Flutter版本` 秒杀这个问题，但对混合开发并没有该说明，加上我们是使用闲鱼的 `flutter_boost` 实现的混合开发，限制了 `Flutter` 的版本，所以我也就没有去实践该方案对我们是否可行



## 二、尝试解决

根据自己的实际情况，我选择了上述的第二个解决方案。

### 配置

用 `Xcode` 打开工程项目，在包含 ，在 `Build Settings` 的最下方找到 `User-Defined`，点击 `+ ` 按钮，添加一个键为 `FLUTTER_BUILD_MODE` ，值为 `release` 的配置。

![](/images/2020/09/Flutter-低版本在iOS14上遇到的问题与解决方案/01.png)

![](/images/2020/09/Flutter-低版本在iOS14上遇到的问题与解决方案/03.png)

### 运行

再次运行到真机上，断开 `Xcode` 运行不会再崩溃了

### 问题

真机的问题看似是解决了，但是会有问题

问题一：`release` 或 `profile` 模式下，`Flutter` 使用的 `AOT`，一些功能不能使用，如：代码断点调试，热重载

问题二：上面也提到了，模拟器只能运行在 `debug` 模式下，而我们无法避免会在真机和模拟器之间反复切换运行，每次切换就需要手动调整 `FLUTTER_BUILD_MODE` 的值，十分麻烦

那有什么好的办法解决上面遇到的问题呢？



### 三、优化方案

其实，真机上的 `APP` 在断开 `Xcode` 后无法运行，这个对我们开发者来说不是什么问题，问题是给到测试人员就必须要可以打开才行，包括蒲公英上的包，所以为了节省这些不必要的时间，我们需要自己动手撸一个帮助我们切换 `Flutter编译模式` 的脚本。

在修改 `FLUTTER_BUILD_MODE` 的值时，我从 `git` 中发现，实际上是修改了 `项目.xcodeproj`，那目前有什么工具可以帮助我们修改 `xcodeproj` 文件呢？

这里我找到了[mod-pbxproj](https://github.com/kronenthaler/mod-pbxproj)，安装和使用在该库的 `wiki` 上写的很清楚，这里就不再赘述了，直接上代码

```python
import getopt
import sys
from pbxproj import XcodeProject

if __name__ == "__main__":
    argv = sys.argv[1:]
    # 处理flutter_build_mode
    flutter_build_mode = (False, "debug/release")
    # target名称
    target_name = None

    try:
        opts, args = getopt.getopt(argv, "p:m:t:", ["path=, mode=, target="])
    except getopt.GetoptError:
        print('switch_flutter_build_mode.py -p "plist文件路径" -m "模式(release|debug)" -t "target名称"')
        sys.exit(1)

    for opt, arg in opts:
        if opt in ["-p", "--path"]:
            project_path = arg
            if len(project_path) == 0:
                print('请输入项目的地址')
                sys.exit(2)
        if opt in ["-m", "--mode"]:
            flutter_build_mode = (True, arg if len(arg) > 0 else "release")
        if opt in ["-t", "--target"]:
            target_name = arg

    # 处理flutter
    if flutter_build_mode[0]:
        fileName = project_path.split("/")[-1]
        if not fileName.endswith("xcodeproj"):
            print("请使用-p指定.xcodeproj文件的路径")
            sys.exit(3)
        project = XcodeProject.load(project_path + '/project.pbxproj')
        # 设置 User-Defined (如果target_name是None，则每个target都会设置flag)
        project.set_flags('FLUTTER_BUILD_MODE', flutter_build_mode[1], target_name)
        project.save()
```

使用也很简单，终端直接输入如下命令

```shell
python switch_flutter_build_mode.py -p 'xxx/项目.xcodeproj' -t target名称 -m release
```

各参数说明

| 参数 | 用途                                       |
| ---- | ------------------------------------------ |
| -p   | `xcodeproj` 文件的路径                     |
| -t   | `target` 名称                              |
| -m   | 编译模式 ( `release`、`debug`、`profile` ) |

PS: 我使用的 `Python3`，上面的脚本也是基于 `Python3` 写的

我们是使用 `Jenkins` 进行打包并自动上传至蒲公英的，所以只需要在 `Jenkins` 中配置打包前调用该脚本即可。

最后再结合 [Shuttle](http://fitztrev.github.io/shuttle/) 这个软件，就可以实现以界面的方式去切换编译模式了

![](/images/2020/09/Flutter-低版本在iOS14上遇到的问题与解决方案/02.png)



## 四、最后说两句

本文是基于 `Flutter混合开发` 进行说明的，如果有什么不对或不足的地方，欢迎指正，感谢大家的阅读





