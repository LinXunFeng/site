---
title: iOS 组件化开发（四）：fastlane实现pod自动化
date: 2018-04-06 19:53:20
categories: "iOS"
tags:
  - iOS
  - Git
  - GitHub
  - Cocoapods
  - 组件化
---

<Excerpt in index | 首页摘要> 

在第一次组件化的时候，需要执行很多操作，这些操作可以在【[iOS 组件化开发（一）：远程私有库的基本使用](http://linxunfeng.top/2018/04/06/iOS-组件化开发（一）：远程私有库的基本使用/)】，这里就不再赘述，在组件化后的重复性操作就是升级，而升级这个过程是一模一样的。那么，我们有什么办法可以很方便的搞定这一过程来节约我们大量的时间呢？

+<!-- more -->
<The rest of contents | 余下全文>

![](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/fastlane.png)

> 在第一次组件化的时候，需要执行很多操作，这些操作可以在【[iOS 组件化开发（一）：远程私有库的基本使用](http://linxunfeng.top/2018/04/06/iOS-组件化开发（一）：远程私有库的基本使用/)】，这里就不再赘述，在组件化后的重复性操作就是升级，而升级这个过程是一模一样的。那么，我们有什么办法可以很方便的搞定这一过程来节约我们大量的时间呢？

## 一、升级必备操作
修改完核心代码后，一共还需要做以下几步：<br>
1、修改spec文件（修改s.version，s.description等）<br>
2、`pod install` （使Example与pod下来的库产生关联）<br>
3、提交本地仓库代码至远程仓库<br>
4、打标签，并提交至远程<br>
5、验证spec，并提至私有索引库

## 二、Fastlane

### 1、简介
[Fastlane文档说明 ](https://docs.fastlane.tools/)
Fastlane是一个ruby脚本集合，它可以按照我们指定的路线，在指定位置执行我们所要执行的操作。这里我们称这样的路线为「航道(lane)」，这样的操作称为「Action」

Action是Fastlane自动化流程中的最小执行单元，用来执行Fastlane脚本中的命令，关于更多的描述可以到[Actions - fastlane docs](https://docs.fastlane.tools/actions/Actions/)查看，里面也介绍了常用的action有哪些，顺带附上[action的源码地址](https://github.com/fastlane/fastlane/tree/master/fastlane/lib/fastlane/actions)，这个源码在后面自定义起参考作用

### 2、 安装
- 确保ruby为最新版本
```
brew update
brew install ruby
```

- 安装fastlane
```
sudo gem install -n /usr/local/bin fastlane
```

- 查看当前fastlane版本
```
fastlane --version
```

- 查看所有action
```
fastlane actions
```

### 三、fastlane初始化
cd到你的本地组件仓库的根目录
- ~~初始化fastlane~~
~~`fastlane init`~~
~~不过这个步骤对我们来说可以跳过，在init后提示你输入一些东西，包括上传需要用到的APPLE ID什么的一堆东西，由于我们并不涉及这些，所以我们使用更方便的方式~~

```
# 创建一个fastlane文件夹
# 进入fastlane目录
# 创建一个Fastfile文件
mkdir fastlane
cd fastlane
touch Fastfile
```
![目录结构](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/目录结构.png)

### 1、修改Fastfile
```
desc '描述航道作用'
lane :航道名称 do |options|

// options 可以用来传递参数
// 示例：varName = options[:name]

// 航道上需要执行的操作

end
```

航道上要扫描的操作可以到[Actions](https://docs.fastlane.tools/actions)上查找，可以通过关键字搜索，如下图
![cocoapods](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/cocoapods.png)
点进去可以看到具体的使用及参数说明

![使用说明](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/使用说明.png)

这里附上本人的Fastfile内容：
```
desc 'LXFUpdatePodTool 航道用来自动化升级维护私有库'
lane : LXFUpdatePodTool do |options|

tagNum = options[:tag]
podspecName = options[:specName]

# 航道上需要执行的操作
# 具体action到 https://docs.fastlane.tools/actions 上面查找
# 这里的路径以仓库根目录为准

# 1、修改spec文件（修改s.version，s.description等）
# 2、pod install （使Example与pod下来的库产生关联）
cocoapods(
  clean: true,
  podfile: "./Example/Podfile"
)


# 3、提交本地仓库代码至远程仓库
git_add(path: ".")
git_commit(path: ".", message: "upgrade repo")
push_to_git_remote


# 4、打标签，并提交至远程
add_git_tag(
  tag: tagNum
)
push_git_tags


# 5、验证spec，并提至私有索引库
pod_lib_lint(allow_warnings: true)
# 因为本地索引库repo的名字是基本上不会去改变的，所以这里直接写死 LXFSpecs
# podspec的名字需要由外界传入
pod_push(path: "#{podspecName}.podspec", repo: "LXFSpecs")


end
```

### 2、验证Fastfile

```
fastlane lanes
```
![Fastfile验证成功](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/Fastfile验证成功.png)

### 3、执行fastlane

**需要在组件仓库的根目录下执行**
![根目录](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/根目录.png)


```
fastlane LXFUpdatePodTool tag:0.1.1 specName:LXFMain
```
![开始执行](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/开始执行.png)

![上传完成](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/上传完成.png)

整个执行过程不超过30秒～

## 四、自定义Action
以上的过程已经可以完成一整个自动化更新了，但是有一点需要注意的是，这个输入的tag可能会面临一个问题，那就是本地和远程都可能已经存在，即发生冲突，这个时候我们可以选择自动删除本地和远程冲突的那个tag，再重新上传当前tag

### 1、创建一个新的action
```
fastlane new_action
```
按要求输入Action名称
![输入action名称](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/输入action名称.png)

完成后fastlane目录下就会多出一个名为actions的文件夹，里面存放的就是你自定义action
![](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/actions文件夹.png)

### 2、编辑自定义action
打开remove_git_tag.rb，开始自定义我们的action吧，什么？不会语法怎么办？可以参考别人的嘛，上面给出的[action的源码地址](https://github.com/fastlane/fastlane/tree/master/fastlane/lib/fastlane/actions)就有用武之地了，比如[pod_push](https://github.com/fastlane/fastlane/blob/master/fastlane/lib/fastlane/actions/pod_push.rb)。这里我直接贴出我已经完成的主要代码

```
# 可以使用 fastlane action remove_git_tag 来参看详细描述

def self.run(params)
  # 这里写要执行的操作 
  # params[:参数名称] 参数名称与下面self.available_options中的保持一致
  tagNum = params[:tagNum]
  rmLocalTag = params[:rmLocalTag]
  rmRemoteTag = params[:rmRemoteTag]

  command = []
  if rmLocalTag
    # 删除本地标签
    # git tag -d 标签名称
    command << "git tag -d #{tagNum}"
  end
  if rmRemoteTag
    # 删除远程标签
    # git push origin :标签名称
    command << "git push origin :#{tagNum}"
  end

  result = Actions.sh(command.join('&'))
  UI.success("Successfully remove tag 🚀 ")
  return result

end

def self.description
  # 对当前脚本的简单描述
  "删除tag"
end

def self.details
  # 对当前脚本的具体描述
  "使用当前action来删除本地和远程冲突的tag"
end

def self.available_options
  # 用来传递参数
  [ 
    FastlaneCore::ConfigItem.new(key: :tagNum,
                                  description: "输入即将删除的tag",
                                  is_string: true),
    FastlaneCore::ConfigItem.new(key: :rmLocalTag,
                                  description: "是否删除本地tag",
                                  optional:true,
                                  is_string: false,
                                  default_value: true),
    FastlaneCore::ConfigItem.new(key: :rmRemoteTag,
                                  description: "是否删除远程tag",
                                  optional:true,
                                  is_string: false,
                                  default_value: true)
  ]
end

def self.authors
  # 作者姓名
  ["LinXunFeng"]
end
```

### 3、查看action描述

同样，这里先cd到组件库的根目录下执行，原因是这个自定义action只存在当前根目录下的fastlane中，其它fastlane的非自定义的action就不用在当前根目录下操作～
```
fastlane action remove_git_tag
```
![查看具体描述](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/查看具体描述.png)


### 4、测试执行

先来看看当前组件库已存在的tag
```
git tag
```
![已存在的tag](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/已存在的tag.png)

可以看到，我是已经有一个`0.1.1`版本的了。这时我们再来执行一次LXFUpdatePodTool航道
```
fastlane LXFUpdatePodTool tag:0.1.1 specName:LXFMain
```
![自动清除](linxunfeng.github.io/images/2018/04/iOS 组件化开发（四）：fastlane实现pod自动化/自动清除.png)

## 五、工具拿走
[LXFUpdatePodTool](https://github.com/LinXunFeng/LXFUpdatePodTool) 已经传到我的GitHub上，需要的同学就拿走吧，顺手给个Star咯 Orz
