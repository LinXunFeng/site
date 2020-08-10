---
title: Flutter - 打印好用的Debug日志
date: 2020-06-26 00:35:00
categories: "Flutter"
tags:
  - Dart
  - Flutter
---

<Excerpt in index | 首页摘要> 

做 `iOS` 开发时这个功能很常用， 在 `OC` 和 `Swift` 中都可以很轻松实现，因为系统本来就提供了用于日志输出的预处理宏，只要我们拿来拼接就可以了，但是在 `Dart` 中并不提供这些，那有什么办法实现它呢？

+<!-- more -->

<The rest of contents | 余下全文>

## 一、思考
做 `iOS` 开发时这个功能很常用， 在 `OC` 和 `Swift` 中都可以很轻松实现，因为系统本来就提供了用于日志输出的预处理宏，只要我们拿来拼接就可以了，但是在 `Dart` 中并不提供这些，那有什么办法实现它呢？

我们回想在开发过程中，是不是发现只要一不小心抛异常，就可以看到类似如下的打印内容，而且还能清楚的知道异常是在哪个文件和哪一行的代码造成的。

![](/images/2020/06/Flutter-打印好用的Debug日志/抛异常.png)

> 所以如果我们可以在调用函数时拿到当前调用堆栈，就可以取到一系列想要的数据。

## 二、实践
在 `dart:core` 中提供了 `堆栈跟踪(StackTrace)`，可以通过 `StackTrace.current` 取到当前的堆栈信息，打印如下图所示，会发现这不好拿到我们想要的信息。

![](/images/2020/06/Flutter-打印好用的Debug日志/StackTrace.png)

这里我用到了官方开发的一个包 [stack_trace](https://pub.dev/packages/stack_trace)，它可以将堆栈信息变得更多人性化，并方便我们查看堆栈信息和获取想要的数据。

**ps: `stack_trace` 在 `Flutter` 环境下直接导包即可使用，而在纯 `Dart` 下需要将其添加为依赖于`pubspec.yaml`中。**

```yaml
dependencies:
  stack_trace: ^1.9.3
```


那下面我们来试试 [stack_trace](https://pub.dev/packages/stack_trace) 的威力吧
```dart
import 'package:stack_trace/stack_trace.dart';

// 将 StackTrace 对象转换成 Chain 对象
// 当然，这里也可以直接用 Chain.current();
final chain = Chain.forTrace(StackTrace.current);
// 拿出其中一条信息
final frames = chain.toTrace().frames;
final frame = frames[1];
// 打印
print("所在文件：${frame.uri} 所在行 ${frame.line} 所在列 ${frame.column}");

// 打印结果
// flutter: 所在文件：package:flutterlog/main.dart 所在行 55 所在列 23
```

## 三、呈上代码

下面我做了一点封装，直接拿走即可使用，打印效果如下所示：

完整的代码和示例请到GitHub上[【查看】](https://github.com/LinXunFeng/flutter_log)。

![](/images/2020/06/Flutter-打印好用的Debug日志/打印效果.png)

代码：
```dart
// log.dart

enum FLogMode {
  debug,    // 💚 DEBUG
  warning,  // 💛 WARNING
  info,     // 💙 INFO
  error,    // ❤️ ERROR
}

void FLog(dynamic msg, { FLogMode mode = FLogMode.debug }) {
  if (kReleaseMode) { // release模式不打印
    return;
  }
  var chain = Chain.current(); // Chain.forTrace(StackTrace.current);
  // 将 core 和 flutter 包的堆栈合起来（即相关数据只剩其中一条）
  chain = chain.foldFrames((frame) => frame.isCore || frame.package == "flutter");
  // 取出所有信息帧
  final frames = chain.toTrace().frames;
  // 找到当前函数的信息帧
  final idx = frames.indexWhere((element) => element.member == "FLog");
  if (idx == -1 || idx+1 >= frames.length) {
    return;
  }
  // 调用当前函数的函数信息帧
  final frame = frames[idx+1];

  var modeStr = "";
  switch(mode) {
    case FLogMode.debug:
      modeStr = "💚 DEBUG";
      break;
    case FLogMode.warning:
      modeStr = "💛 WARNING";
      break;
    case FLogMode.info:
      modeStr = "💙 INFO";
      break;
    case FLogMode.error:
      modeStr = "❤️ ERROR";
      break;
  }

  print("$modeStr ${frame.uri.toString().split("/").last}(${frame.line}) - $msg ");
}
```