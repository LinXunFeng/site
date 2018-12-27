---
title: Python - 虚拟环境
date: 2018-04-02 16:26:22
categories: "Python"
tags:
  - python
---


<Excerpt in index | 首页摘要> 

平时在开发时我们都会先安装一些python需要的包，每次安装都会有一个版本，如果不同项目需要不同版本的包时就会出现不兼容的情况。应对这种情况我们就可以搭建多个虚拟环境来应对不同的环境需求，在虚拟环境中搭建一个Python项目运行所需要的那些包，将来根据运行的项目来切换不同环境即可

+<!-- more -->

<The rest of contents | 余下全文>


> 平时在开发时我们都会先安装一些python需要的包，每次安装都会有一个版本，如果不同项目需要不同版本的包时就会出现不兼容的情况。应对这种情况我们就可以搭建多个虚拟环境来应对不同的环境需求，在虚拟环境中搭建一个Python项目运行所需要的那些包，将来根据运行的项目来切换不同环境即可

我们可以在当前用户的家目录中找到【.virtualenvs】文件夹，查看当前所有的虚拟环境

![virtualenvs](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/virtualenvs.png)


## 创建虚拟环境
- 创建：mkvirtualenv [虚拟环境名称]

```shell
mkvirtualenv lxfenv1
```

![安装成功](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/mkvirtualenv.png)

注：创建的过程需要联网

![目录结构](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/目录结构.png)

```
Installing setuptools, pkg_resources, pip, wheel...done.
```
刚刚安装时提示安装的这些东西就存放在你创建好的虚拟环境下的【lib/python2.7/site-packages/】目录中

![lib目录](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/lib目录.png)

- 退出：deactivate
当我们安装好虚拟环境后默认就使用了该虚拟环境，如图标识处可以看出

![当前环境](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/当前环境.png)

如果你想退出当前的虚拟环境，或以使用如下命令：

```shell
deactivate
```

![退出环境](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/退出环境.png)


- 进入：workon [虚拟环境名称]

使用指定的虚拟环境则使用如下命令：

```
workon lxfenv1
```

![workon](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/workon.png)


- 删除：rmvirtualenv [虚拟环境名称]

```shell
rmvirtualenv lxfenv1
```

## 安装拓展包
- 查看当前安装好的包

```
pip list
或者
pip freeze
```

![查看当前安装好的包](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/查看当前安装好的包.png)

ps: 上图`pip list`中列出的四个包是安装虚拟环境必定会安装的包
如图，`pip list`会列出所有的包，而`pip freeze`只会列出扩展的包

- 安装指定包
```
pip install django==1.8.2
# ==1.8.2 为指定版本号，不写则直接安装最新的包
```
**注： pip install xxx 会自动删除旧版本，再安装新版本**

如果不知道包名可以到[pypi](https://pypi.python.org)上搜索

![安装指定的包](linxunfeng.github.io/images/2018/04/Python - 虚拟环境/安装指定的包.png)
