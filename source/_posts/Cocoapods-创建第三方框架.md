---
title: Cocoapods 创建第三方框架
date: 2018-04-06 19:10:01
categories: "iOS"
tags:
  - Git
  - GitHub
  - Cocoapods
---

<Excerpt in index | 首页摘要> 
将框架中的主要文件放入到一个指定文件夹中，比如叫Classes或者Lib都可以

+<!-- more -->
<The rest of contents | 余下全文>

##  一、上传项目到github
将框架中的主要文件放入到一个指定文件夹中，比如叫Classes或者Lib都可以
![目录结构](linxunfeng.github.io/images/2018/04/Cocoapods 创建第三方框架/目录结构.png)

- 打开终端，cd到框架目录 

```
cd /Users/lxf/xxxx/LXFPhotoHelper 
```

- 初始化仓库

```
git init
```

-  将当前目录添加到缓存区

```
git add .
```

- 提交到本地仓库

```
git commit -m '描述'
```

- 添加远程仓库地址

```
git remote add origin https://github.com/LinXunFeng/xxx.git
```

- 提交到远程仓库

```
git push origin master
```

如果出现如下提示
```
fatal: unable to access 'https://github.com/xxx/xxx.git/': The requested URL returned error: 403
```
原因是本地缓存了用户名和密码
编辑.git目录下的config文件
```
vi .git/config
```
找到url那一行，在github.com前加上用户名后保存，再重新执行推送操作
```
url = https://LinXunFeng@github.com/LinXunFeng/xxx.git
```

- 打标签

```
// 具体说明可以执行`git tag --help`后查看
// git tag -a '版本号' -m 'tag描述'
// 注意一下，这里打的标签只是在本地
git tag '0.0.1'
```

- 推着所有标签至远程仓库

```
// 只推着指定版本
// git push origin 版本号 
git push --tags
```


## 二、创建并修改podspec文件
- 创建Spec文件
```
// 名称一般与工程名称保持一致
pod spec create 框架名称
```

![podspec文件](linxunfeng.github.io/images/2018/04/Cocoapods 创建第三方框架/podspec文件.png)

- 修改Spec文件
```
  s.name         = "LXFPhotoHelper（仓库名称）"
  s.version      = "0.0.1（版本号，这里跟下面s.source中的tag有关）"
  s.summary      = "对你自己仓库的简单描述，不要写太多字"
  s.description  = "这个是详细描述，这里需要注意的是，这里文字的长度需要比  
  s.summary的要长，不然会出现警告"
  s.homepage     = "仓库首页地址，如https://github.com/LinXunFeng/LXFPhotoHelper"
  s.license      = "MIT"
  s.author       = { "LinXunFeng" => "598600855@qq.com" }
  # source存放的地址是代码的真正地址
  s.source       = { :git => "仓库对应的git地址，如https://github.com/LinXunFeng/LXFPhotoHelper.git", :tag => "#{s.version}" }
  # pod install时真正下载下来的文件路径，这里指定的是你仓库下的Classes目录中的所有.h和.m文件（填写的是相对地址）
  # ** 通配目录
  s.source_files  = "Classes", "Classes/**/*.{h,m}"

  # s.library = "sqlite3" # 框架依赖系统的sqlite3
```
也可以上官网的手册【[Podspec Syntax Reference](https://guides.cocoapods.org/syntax/podspec.html)】上查看

这里需要我们注意的是`s.version= "0.0.1"`，这里的版本号要与刚刚打的tag一致

## 三、注册trunk

```
// --verbose 打印详情信息
// pod trunk register 邮箱 '你的名称' --verbose
pod trunk register 598600855@qq.com 'LinXunFeng' --verbose
```
然后去验证邮箱
![验证成功](linxunfeng.github.io/images/2018/04/Cocoapods 创建第三方框架/验证成功.png)
验证成功后会提示我们回到终端，并敲入`pod trunk push 名称.podspec`

## 四、上传Spec

执行`pod trunk push`后会有一个审核的过程，如果提示没有通过，有ERROR就修改好后重新push，如果只是WARN可以选择在`pod trunk push`后面加上`--allow-warnings`来忽略它们
```
pod trunk push LXFPhotoHelper.podspec --allow-warnings
```

如果出现如下信息，则说明你的框架名字已被占用，得重新改个名字～
所以，在创建你自己的cocoapods仓库时最好是到[cocoapods.org](https://cocoapods.org/)上先查一下有没有相同名字的
```
[!] You (xxx@qq.com) are not allowed to push new versions for this pod. The owners of this pod are yyy@qq.com.
```

上传成功后会自动帮我们更新本地仓库，如果无法搜索到自己的框架，可以先删掉本地的索引文件后再搜索一次
```
rm ~/Library/Caches/CocoaPods/search_index.json
```

当使用pod search 命令可以搜索自己的框架时, 那么就意味着审核通过了


