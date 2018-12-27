---
title: iOS 组件化开发（二）：远程私有库的更新与子库
date: 2018-04-06 19:50:41
categories: "iOS"
tags:
  - iOS
  - Git
  - GitHub
  - Cocoapods
  - 组件化
---


<Excerpt in index | 首页摘要> 

在上一篇【[iOS 组件化开发（一）：远程私有库的基本使用](http://linxunfeng.top/2018/04/06/iOS-组件化开发（一）：远程私有库的基本使用/)】中我们已经实战了远程私有库的基本操作，但是组件不可能上传一次就完事了，随着业务的增加，我们的组件可能还需要添加更多的东西，或者修复一些问题，这就需要我们对私有库代码进行升级与维护

+<!-- more -->
<The rest of contents | 余下全文>

> 在上一篇【[iOS 组件化开发（一）：远程私有库的基本使用](http://linxunfeng.top/2018/04/06/iOS-组件化开发（一）：远程私有库的基本使用/)】中我们已经实战了远程私有库的基本操作，但是组件不可能上传一次就完事了，随着业务的增加，我们的组件可能还需要添加更多的东西，或者修复一些问题，这就需要我们对私有库代码进行升级与维护

这里以对基础组件里添加了一个Cache工具为例

![添加Cache工具](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/添加Cache工具.png)

添加完成后我们需要更新到远程仓库

## 一、更新远程仓库
cd 到本地仓库的位置，执行以下操作
### 1、代码更新

```
git add .
git commit -m '更新描述'
git push origin master
```
![代码升级](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/代码升级.png)


### 2、版本更新

**版本更新 这一步非常重要，为更新索引库做准备**

```
git tag -a '新版本号' -m '注释'
git push --tags
```
![版本升级](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/版本升级.png)

查看远程仓库，标签数已经有2个了，点进去就可以看到0.2.0，这里我们就不去看了
![](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/2个标签.png)


## 二、修改描述文件并更新索引库

### 1、修改Sepc
打开你的`xx.podspec`文件，将原本的版本号改为`0.2.0`，与刚刚的tag保持一致
```
 s.version = '0.2.0'
```

### 2、验证远程Spec

```
pod spec lint --private
```

![验证远程Spec](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/验证远程Spec.png)


### 3、更新索引库
```
pod repo push 索引库名称 xxx.podspec
```
![更新索引库](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/更新索引库.png)


## 三、更新使用
```
// --no-repo-update 不更新本地索引库
// 因为刚刚已经自己手动更新过了，所以这里我们选择跳过更新
pod update --no-repo-update
```
![更新框架](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/更新框架.png)

![更新成功](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/更新成功.png)

<hr>

## 四、第三方依赖

当我们的私有库需要依赖其它第三方才可以正常使用时，我们就需要在spec文件中开启依赖，例如下面所示代码，表明当前仓库需要依赖AFN和SDWebImage
```
s.dependency 'AFNetworking', '~> 3.2.0'
s.dependency 'SDWebImage', '~> 4.3.3'
```
修改后更新操作同上所述，这里就不再赘述了。

但是这里存在一个问题，如果来了一位新的小伙伴，他所负责的部分只需要LXFBase下的Category，而LXFBase下的Cache才需要依赖SDWebImage，此时他若是pod一整个LXFBase岂不是平白无故安装了第三方依赖库，那应该怎么做呢？

> 方案就是可以通过子库Subspecs来解决因需要一个小小的工具而依赖整个基础组件的问题

## 五、子库Subspecs

什么是Subspecs？这里我们可以搜索一下SDWebImage

```
pod search 'SDWebImage'
```

![Subspecs](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/Subspecs.png)

可以看到，如果我们只需要用到SDWebImage中的GIF功能，那么并不需要将整个SDWebImage都下载下来，在Podfile中将~~`pod 'SDWebImage' `~~ 改为 `pod SDWebImage/GIF`即可单独使用这一功能

那接下来我们就来看看怎么描述一个子库吧

### 子库格式
```
s.subspec '子库名称' do |别名|

end
```
因为这里已经分离出子库了，所以`s.source_files`和`s.dependency`就不能这么使用了，需要我们在子库里分别指定，所以我们直接把原来的`s.source_files`和`s.dependency`都注释掉。写法参考如下


```
# s.source_files = 'LXFBase/Classes/**/*'
# s.dependency 'SDWebImage', '~> 4.3.3'

s.subspec 'Cache' do |c|
  c.source_files = 'LXFBase/Classes/Cache/**/*'
  c.dependency 'SDWebImage', '~> 4.3.3'
end

s.subspec 'Category' do |c|
  c.source_files = 'LXFBase/Classes/Category/**/*'
end

s.subspec 'Tool' do |t|
  t.source_files = 'LXFBase/Classes/Tool/**/*'
end
```

修改后再按之前的步骤更新索引库和组件库就可以了

**ps: 在添加第三方依赖描述后做验证或者上传操作可能会很慢，因为它在克隆第三方库如SDWebImage，有兴趣的可以在命令后面加入`--verbose`来查看详情情况**
```
pod spec lint --private --verbose
```

在成功更新组件库和索引库后我们再来搜索一下试试
```
pod search 'LXFBase'
```

![subspec添加成功](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/subspec添加成功.png)

现在就可以爱装哪个就装哪个了，在Podfile中指定要安装的子库就行了
```
pod 'LXFBase/Cache'
```

```
pod install
```

![安装指定子库与依赖库](linxunfeng.github.io/images/2018/04/iOS 组件化开发（二）：远程私有库的更新与子库/安装指定子库与依赖库.png)

