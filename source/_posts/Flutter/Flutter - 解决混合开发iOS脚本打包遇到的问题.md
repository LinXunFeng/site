---
title: Flutter - 解决混合开发iOS脚本打包遇到的问题
date: 2020-08-12 00:00:00
categories: "Flutter"
tags:
  - Dart
  - Flutter

---

<Excerpt in index | 首页摘要> 

使用Xcode手动打包是正常的，但是使用脚本打包会报错

+<!-- more -->

<The rest of contents | 余下全文>

使用 `Xcode` 手动打包是正常的，但是使用脚本打包会报错，错误如下：

```shell
The following build commands failed:
	PhaseScriptExecution [CP-User]\ Run\ Flutter\ Build\ Script .../Script-C3A097A8FE12FF5F875B057C.sh
	
flutter build ios --release
then re-run Archive from Xcode.
Command PhaseScriptExecution failed with a nonzero exit code
```



## 定位错误

![](/images/2020/08/Flutter-解决混合开发iOS脚本打包遇到的问题/01.png)

到 `Flutter` 环境目录下，按图上所示地址找到 `xcode_backend.sh`，也可以直接看 [官方脚本链接](https://github.com/flutter/flutter/blob/bcc42901ad6bb3ec644be225b5f9cd998852e0ef/packages/flutter_tools/bin/xcode_backend.sh#L90-L101)

```shell
  # Archive builds (ACTION=install) should always run in release mode.
  if [[ "$ACTION" == "install" && "$build_mode" != "release" ]]; then
    EchoError "========================================================================"
    EchoError "ERROR: Flutter archive builds must be run in Release mode."
    EchoError ""
    EchoError "To correct, ensure FLUTTER_BUILD_MODE is set to release or run:"
    EchoError "flutter build ios --release"
    EchoError ""
    EchoError "then re-run Archive from Xcode."
    EchoError "========================================================================"
    exit -1
  fi
```



## 解决方案

可以看到，官方脚本的说明里面给出两个解决方案

> 方案一：直接设置 `FLUTTER_BUILD_MODE` 为 `release`
>
> 方案二：先运行 `flutter build ios --release` ，再使用 `Xcode` 去打包

这里我们是用 `Jenkins` 脚本进行打包，所以方案二不适用，方案一更加方便些

```shell
# 设置Flutter的编译模式为release
export FLUTTER_BUILD_MODE=release

# 执行原有项目的打包脚本
./script/build_iOS.sh
```

