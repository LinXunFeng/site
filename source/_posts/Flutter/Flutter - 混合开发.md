---
title: Flutter - 混合开发
date: 2020-07-12 00:00:00
categories: "Flutter"
tags:
  - Dart
  - Flutter

---

<Excerpt in index | 首页摘要> 

目前大多数公司都有自己开发多年的项目，不可能直接用 `Flutter` 从头开发一套，那样不实现，除非是小项目，因此只能是在原有的基础上用 `Flutter` 来开发新业务或重构旧业务，而这里就需要用到 `Flutter` 的 `混合开发`

+<!-- more -->

<The rest of contents | 余下全文>

> 目前大多数公司都有自己开发多年的项目，不可能直接用 `Flutter` 从头开发一套，那样不实现，除非是小项目，因此只能是在原有的基础上用 `Flutter` 来开发新业务或重构旧业务，而这里就需要用到 `Flutter` 的 `混合开发`



### 一、创建Flutter模块

使用混合开发就不能像之前一样直接上来就创建一个 `Flutter` 项目，而是要使用 `Flutter模板`

```shell
# flutter_module_lxf 可以随便你命名
flutter create --template module flutter_module_lxf

# --template 可以替换为 -t
# flutter create -t module flutter_module_lxf
```

创建出来的 `Flutter` 模块依然是可以像之前创建的`Flutter项目` 一样打开和运行的。 

目录下有也有 `ios` 和 `android` 目录，只不过前面加了个点 ，成了点目录。

![](/images/2020/07/Flutter-混合开发/混合开发iOS01.png)



## 二、iOS

### 集成

> 通过 `Cocoapods` ，将 `Flutter` 模块编译成一个库，再到原生项目中进行引入和使用即可

在 `Podfile` 中添加两行配置

```ruby
# 指定我们刚刚创建的 Flutter 模块的路径
flutter_application_path = '../flutter_module_lxf'

# 拼接脚本文件的路径: .ios/Flutter/podhelper.rb
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')
```

在每个需要引用 `Flutter` 的 `Target` 下，都需要添加一行配置

```ruby
install_all_flutter_pods(flutter_application_path)
```

添加后如下所示：

```ruby
flutter_application_path = '../flutter_module_lxf'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

use_frameworks!
target 'LXFFlutterHybridDemo' do
  
  install_all_flutter_pods(flutter_application_path)
  
end
```

添加完成后，执行一次 `pod install`

混合开发混合开发![](/images/2020/07/Flutter-混合开发/混合开发iOS02.png)

### 使用

> 两个步骤
>
> 1. 获取 Flutter引擎 `FlutterEngine`
> 2. 通过 `FlutterEngine` 创建 `FlutterViewController`



#### 基本使用

`AppDelegate` 类中声明一个 `FlutterEngine` 变量，在 `didFinishLaunchingWithOptions` 方法中启动 `Flutter引擎`

```swift
// AppDelegate.swift

import Flutter

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  
    // 创建 Flutter引擎
    lazy var flutterEngine = FlutterEngine(name: "lxf")

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // 启动 Flutter引擎
        flutterEngine.run()
        
        return true
    }
  
  ...
}
```

 `ViewController` 中添加一个按钮，点击弹出 `Flutter模块`

```swift
// ViewController.swift

override func viewDidLoad() {
  super.viewDidLoad()

  let btn = UIButton(type: .custom)
  btn.frame = CGRect(x: 100, y: 200, width: 200, height: 44)
  btn.backgroundColor = .black
  btn.addTarget(self, action: #selector(showFlutterVc), for: .touchUpInside)
  btn.setTitle("弹出Flutter模块", for: .normal)
  self.view.addSubview(btn)
}

@objc func showFlutterVc() {
  // 创建FlutterViewController
  // 这里的 engine 可以传 nil，Flutter会帮我们自动创建一个引擎，但是性能较差
  let flutterVc = FlutterViewController(engine: fetchFlutterEngine(), nibName: nil, bundle: nil)
  self.present(flutterVc, animated: true, completion: nil)
}

func fetchFlutterEngine() -> FlutterEngine {
  return (UIApplication.shared.delegate as! AppDelegate).flutterEngine
}
```

![](/images/2020/07/Flutter-混合开发/混合开发iOS04.jpg)



如果遇到报 `Command PhaseScriptExecution failed with a nonzero exit code` 错误，如下图所示：

![](/images/2020/07/Flutter-混合开发/混合开发iOS03.png)

请先用 `Android Studio` 或 `VSCode` 打开 `Flutter模块` 项目并运行到iOS设备上，让其帮我们对iOS项目进行一些初始化配置。成功运行后就可以关闭 `Flutter模块` 项目的运行了，接着再用 `Xcode` 打开原生项目运行即可。



#### 修改初始路由

官方文档里面提到，修改初始路由，需要在 `Flutter引擎` 在 `run` 之前，通过 `invokeMethod` 调用 `setInitialRoute` 方法进行设置，代码如下

```swift
// 修改初始路由
flutterEngine.navigationChannel.invokeMethod("setInitialRoute", arguments: "/other")
// 启动 Flutter引擎
flutterEngine.run()
```

但是，我发现这样写并没有起任何作用，在 `Flutter` 的官方 `issue` 上也有人提到这个问题： [【setInitialRoute is broken for iOS add-to-app #59895】](https://github.com/flutter/flutter/issues/59895)，目前只能官方进行修复和调整 `API`

临时可以使用如下方式实现：

```swift
let flutterVc = FlutterViewController(project: FlutterDartProject(), nibName: nil, bundle: nil)
flutterVc.setInitialRoute("/other")
self.present(flutterVc, animated: true, completion: nil)
```

虽然这么写可以实现这个功能，但是会有明显的类似卡顿的现象，因为使用这种方式去创建 `FlutterViewController` 之前，会隐式创建和启动一个 `FlutterEngine`，而我们弹出 `FlutterViewController` 时 `FlutterEngine` 还没加载完毕，所以我们会看到先弹出了一个透明的界面，再显示 `/other` 路由对应的界面视图。



#### 使用 FlutterAppDelegate

使用 `FlutterAppDelegate `这个不是必要的操作，但是如果你想让 `Flutter模块` 也能使用原生的功能的话，建议使用

>  原生功能
>
> - 处理 `openURL` 的回调
> - 列表视图在点击状态栏后滚到顶部

```swift
class AppDelegate: FlutterAppDelegate 
```

更具体的使用，请阅读 [官方文档](https://flutter.dev/docs/development/add-to-app/ios/add-flutter-screen?tab=no-engine-vc-swift-tab#using-the-flutterappdelegate)



## 三、Android

修改安卓项目 根目录下的 `settings.gradle` 文件 

```groovy
// settings.gradle

include ':app'                                    // assumed existing content
setBinding(new Binding([gradle: this]))                                // new
evaluate(new File(                                                     // new
  settingsDir.parentFile,                                              // new
  // 这里的 flutter_module_lxf 请修改为你自己创建的Flutter模板目录名称
  'flutter_module_lxf/.android/include_flutter.groovy'                 // new
))  
```

![](/images/2020/07/Flutter-混合开发/混合开发iOS09.png)

修改安卓项目 `app` 目录下的 `build.gradle` 文件 

```groovy
// app/build.gradle

dependencies {
  ...
  // 配置flutter依赖
  implementation project(':flutter')
}
```

如果在编译的时候遇到如下错误

```
Default interface methods are only supported starting with Android N (--min-api 24): void androidx.lifecycle.DefaultLifecycleObserver.onCreate(androidx.lifecycle.LifecycleOwner)
```

请确认是否指定了使用 `Java 8` 进行编译 [【官方文档 -  Java 8 requirement】](https://flutter.dev/docs/development/add-to-app/android/project-setup#java-8-requirement)

修改安卓项目  `app` 目录下的 `build.gradle` 文件 

```
// app/build.gradle

android {
	...
  compileOptions {
      sourceCompatibility 1.8
      targetCompatibility 1.8
  }
  ...
}
```

![](/images/2020/07/Flutter-混合开发/混合开发iOS08.png)

修改 `app/src/main/AndroidManifest.xml` 文件

```xml
// app/src/main/AndroidManifest.xml

<activity
  android:name="io.flutter.embedding.android.FlutterActivity"
  android:theme="@style/AppTheme"
  android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
  android:hardwareAccelerated="true"
  android:windowSoftInputMode="adjustResize"
  />
```

![](/images/2020/07/Flutter-混合开发/混合开发iOS10.png)

添加一个按钮，点击弹出 `Flutter模块`



```xml
<!--  activity_main.xml  -->

<Button
    android:id="@+id/btn"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textSize="20sp"
    android:text="弹出Flutter模块"
    android:background="#000000"
    android:textColor="#ffffff"
    android:gravity="center"
    android:onClick="btnClick"
    />
```



```java
// MainActivity.java

public void btnClick(View v) {
    startActivity(
        FlutterActivity.createDefaultIntent(this)
    );
}
```



## 四、调试与热重载

> 由于当前我们是使用原生开发工具(如：Xcode)来运行项目，每次修改我们的 
>  `Flutter模块` 的代码，也就需要重新运行才能看到效果，不像之前按下 `Cmd + s` 就能进行热重载。这样 `Flutter模块` 的开发效率极其低下，那有没有办法可以让我们像之前开发 `Flutter` 项目时那样进行 `热重载` 呢？答案是有的



`Flutter` 官方提供了 `flutter attach` ，以辅助我们开发，到终端下执行

```shell
flutter attach
```

如果当前有多个设备，会提示我们需要指定 `attach` 哪个设备

![](/images/2020/07/Flutter-混合开发/混合开发iOS05.png)

按要求加上指定参数即可

```shell
flutter attach -d FE305309-9E79-418D-BA3F-7EFECF2980BC
```

![](/images/2020/07/Flutter-混合开发/混合开发iOS06.png)

如图，这样就关联上了，你在 `dart` 文件里面对界面进行任何修改后，按 `r` 进行热重载，按 `R` 进行热启动。



如果你使用的是 `Android Studio`，可以直接选择对应的设备后，点击右边的 `Flutter Attach` 按钮，执行成功后就可以跟之前一样按 `Cmd + s` 进行热重载了。

![](/images/2020/07/Flutter-混合开发/混合开发iOS07.png)



## 五、资料

- GitHub

  [LXFFlutterHybridDemo](https://github.com/LinXunFeng/LXFFlutterHybridDemo)

- 官方文档

  [add-to-app](https://flutter.dev/docs/development/add-to-app) | [add-to-app/ios](https://flutter.dev/docs/development/add-to-app/ios) | [add-to-app/android](https://flutter.dev/docs/development/add-to-app/android) |  [Debugging & hot reload](https://flutter.dev/docs/development/add-to-app/debugging)

