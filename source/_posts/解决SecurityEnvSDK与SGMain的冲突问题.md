---
title: 解决SecurityEnvSDK与SGMain的冲突问题
date: 2020-03-14 13:21:00
categories: "iOS"
tags:
  - iOS
  - Cocoapods
---

<Excerpt in index | 首页摘要> 

报错是说有重复类，解决的办法比较简单粗暴，就是把在Xcode里全文搜索
` -framework "SecurityEnvSDK"`，接着全文替换为空字符串就可以了。

+<!-- more -->

<The rest of contents | 余下全文>

## 问题
在集成友盟统计和阿里百川之后项目报如下错误

```shell
duplicate symbol '_OBJC_CLASS_$_tdvSFHFKeychainUtils' in:
    /Users/.../Pods/UMCSecurityPlugins/thirdparties/SecurityEnvSDK.framework/SecurityEnvSDK(SecurityEnvSDK99999999.o)
    /Users/.../阿里百川/WXFrameworks/SGMain.framework/SGMain(SGMain99999999.o)
duplicate symbol '_OBJC_METACLASS_$_tdvSFHFKeychainUtils' in:
    /Users/.../Pods/UMCSecurityPlugins/thirdparties/SecurityEnvSDK.framework/SecurityEnvSDK(SecurityEnvSDK99999999.o)
    /Users/.../阿里百川/WXFrameworks/SGMain.framework/SGMain(SGMain99999999.o)
duplicate symbol '_OBJC_CLASS_$_SGDataCollectionLock' in:
    /Users/.../Pods/UMCSecurityPlugins/thirdparties/SecurityEnvSDK.framework/SecurityEnvSDK(SecurityEnvSDK99999999.o)
    /Users/.../阿里百川/WXFrameworks/SGMain.framework/SGMain(SGMain99999999.o)
duplicate symbol '_OBJC_METACLASS_$_SGDataCollectionLock' in:
    /Users/.../Pods/UMCSecurityPlugins/thirdparties/SecurityEnvSDK.framework/SecurityEnvSDK(SecurityEnvSDK99999999.o)
    /Users/.../阿里百川/WXFrameworks/SGMain.framework/SGMain(SGMain99999999.o)
```

>  报错是说有重复类，解决的办法比较简单粗暴，就是把在Xcode里全文搜索
>  ` -framework "SecurityEnvSDK"`，接着全文替换为空字符串就可以了。

虽然解决这个问题的方式很简单，但是每次 `pod install` 后都要做一遍该操作，这就很无语了 。

那有什么办法可以让我们不用自己去做这个烦琐的事情呢？

## 改进
1. 首先要搞清楚，上面的操作原理是怎么回事？其它很简单，就是将下面这两个文件中 `OTHER_LDFLAGS` 所在行的内容里，把 `-framework "SecurityEnvSDK"` 置为空字符串。
```shell
Pods/Target Support Files/Pods-项目名/Pods-项目名.debug.xcconfig
Pods/Target Support Files/Pods-项目名/Pods-项目名.release.xcconfig
```

2. 有什么办法可以让我们在适当的时候去执行这个置空字符串的操作？
`Cocoapods` 提供了一个很好用的 `Hook` 就是 `post_install`，这个钩子的作用就是方便我们在执行 `pod install` 之后去做一些其它配置，这里我们就用它来搞事情。

`Podfile` 文件中使用的是 `ruby` 语言，`ruby` 执行终端命令的代码如下所示：
```shell
post_install do |installer|
  # command = "echo 'hello world'"
  command = "终端命令"
  system(command)
end
```
OK，现在开始搞事！
## 步骤
1. 在项目的目录，即与`Pods`平级目录中，新建一个文件，名为 `fix.py`

```shell
.
├── ...
├── Podfile
├── Podfile.lock
├── Pods
│   ├── ...
│   └── ...
└── fix.py
```

2. 在 `fix.py` 中粘贴如下内容
```python
# -*- coding: UTF-8 -*-
import sys, os, getopt, codecs

def get_current_file_name():
    """获取当前文件名称"""
    return os.path.split(__file__)[-1]

def replace_all_str(file_path, for_str, to_str):
    """
    全文搜索替换或单行替换
    :param file_path: 文件路径
    :param for_str: 要被替换的内容
    :param to_str: 替换之后的内容
    """
    if not os.path.exists(file_path):
        # 文件不存在
        print('文件不存在')
        return
    bak_file_path = file_path+".bak"
    with codecs.open(file_path, 'r', encoding='utf-8') as f, codecs.open(bak_file_path, 'w', encoding='utf-8') as f_w:
        lines = f.readlines()
        for line in lines:
            if "OTHER_LDFLAGS" in line and for_str in line:
                line = line.replace(for_str, to_str)
            f_w.write(line)

    os.remove(file_path)
    os.rename(bak_file_path, file_path)

def throwParamError():
    print("请正确输入命令： %s -p 项目名称" % get_current_file_name())
    sys.exit(0)

def main(argv):
    project_name = ""
    try:
        opts, args = getopt.getopt(argv, "p:", ["project="])
    except getopt.GetoptError:
        throwParamError()
    for opt, arg in opts:
        # print("opt -- ", opt)
        # print("arg -- ", arg)
        if opt in ('-p', '--project'):
            project_name = arg

    if not len(project_name):
        throwParamError()
    
    path_str = "Pods/Target Support Files/Pods-%s/Pods-%s.%s.xcconfig"
    xcconfig_debug_path = path_str % (project_name, project_name, "debug")
    xcconfig_release_path = path_str % (project_name, project_name, "release")
    # print(xcconfig_debug_path)
    # print(xcconfig_release_path)
    be_fixed_str = '-framework "SecurityEnvSDK"'
    replace_all_str(xcconfig_debug_path,  be_fixed_str, '')
    replace_all_str(xcconfig_release_path,  be_fixed_str, '')
    print("%s is fixed successfully" %project_name)

if __name__ == "__main__":
    main(sys.argv[1:])
```
3. 打开 `Podfile`，在内容最后添加如下内容
```ruby
post_install do |installer|
  # 解决SecurityEnvSDK与SGMain的冲突问题
  command = "python fix.py -p 项目名称"
  system(command)
end
```
4. 执行`pod install`

好了，现在开始就又可以继续愉快的搬砖了~

## GitHub
相关代码文件可以到这里下载，如果觉得不错，不妨给个 `Star` 鼓励一下
[https://github.com/LinXunFeng/fix_confict_SecurityEnvSDK_SGMain](https://github.com/LinXunFeng/fix_confict_SecurityEnvSDK_SGMain)