---
title: Docker - 安装、加速和基本使用
date: 2021-01-24 21:52:14
permalink: /pages/2ffb7f/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---

> Docker 容器是一个开源的应用容器引擎，让开发者可以以统一的方式打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何安装了docker引擎的服务器上



本文主要以 `Mac` 平台为例

## 安装

### 方式一: Homebrew

```
brew cask install docker
```

### 方式二: 桌面程序

> 我使用该方式

访问 https://www.docker.com/get-started，在 `Docker Desktop` 下选择你当前系统对应的软件进行下载安装

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124215528.png)





## 镜像加速

### 镜像

#### Daocloud

> 实测，速度不咋地



打开 [daocloud.io](https://www.daocloud.io/mirror) 网站，注册并登录，点击加速器按钮，或者直接打开 https://www.daocloud.io/mirror，打开的页面上会显示与你账号相关的加速地址



![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124215606.png)



#### 其它

> 实测，速度杠杠的，推荐使用

| 镜像                 | 地址                               |
| -------------------- | ---------------------------------- |
| Docker中国区官方镜像 | https://registry.docker-cn.com     |
| 网易                 | http://hub-mirror.c.163.com        |
| ustc                 | https://docker.mirrors.ustc.edu.cn |



```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```



### 配置

> 该部分内容取自 `daocloud`

#### Linux

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://xxx.m.daocloud.io
```

该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件 /etc/docker/daemon.json 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7、Fedora、Arch Linux、openSUSE Leap 42.1，其他版本可能有细微不同。更多详情请访问文档。

#### macOS

Docker For Mac

右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中加入下面的镜像地址:

```
http://xxx.m.daocloud.io
```

点击 Apply & Restart 按钮使设置生效。

Docker Toolbox 等配置方法请参考[帮助文档](http://guide.daocloud.io/dcs/daocloud-9153151.html#docker-toolbox)。



我自己的版本是 `3.1.0(51484)` ，与上述的不太一致。

如图所示，切换到 `Docker Engine` 下，在右侧所示的 `JSON` 内容中添加 `registry-mirrors`

```
"registry-mirrors": ["你的加速器地址"]
```

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124215627.png)

最后点击 `Apply&Restart` 按钮即可。

#### Windows

Docker For Windows

在桌面右下角状态栏中右键 docker 图标，修改在 Docker Daemon 标签页中的 json ，把下面的地址:

```
http://xxx.m.daocloud.io
```

加到" registry-mirrors"的数组里。点击 Apply 。

Docker Toolbox 等配置方法请参考[帮助文档](http://guide.daocloud.io/dcs/daocloud-9153151.html#docker-toolbox)。



## 基本使用

### 查看版本

```
docker -v
```

输出结果

```
Docker version 20.10.2, build 2291f61
```

### 查看Docker信息

```
docker info
```

可以使用该命名查看当前的镜像地址数组

```
 ...
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: true
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://registry.docker-cn.com/
  http://hub-mirror.c.163.com/
  https://docker.mirrors.ustc.edu.cn/
 Live Restore Enabled: false
 ...
```