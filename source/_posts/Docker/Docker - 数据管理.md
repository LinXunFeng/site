---
title: Docker - 数据管理
date: 2021-01-31 14:23:29
permalink: /pages/867207/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145525.jpeg)


## 拷贝数据


### 宿主机文件 -> 容器内
```shell
docker cp 需要拷贝的文件或目录 容器名:容器内的目录路径
```
如：把宿主机当前目录下的 `lxf.sh` 文件，拷贝到  `lxfubuntu1` 容器下的 `/data` 目录中
```shell
docker cp lxf.sh lxfubuntu1:/data
```
### 容器内 -> 宿主机文件 
```shell
docker cp 容器名:容器内需要拷贝的文件或目录 宿主机目录  
```
如：把 `lxfubuntu1` 容器中 `/data` 目录下的 `lxf.sh` 文件，拷贝到宿主机 `~/lxf/` 目录下
```shell
docker cp lxfubuntu1:/data/lxf.sh ~/lxf/
```


## 数据卷管理

> 数据卷管理就是将容器的某个目录，映射到宿主机，作为数据存储同步的目录

命令：
```shell
docker run -itd --name [容器名字] -v [宿主机目录]:[容器目录] [镜像名称] [命令(可选)]
```
下面进行操作示范：


在宿主机创建一个名为 `data` 的目录，这个名字可任意
```shell
mkdir data
```
将宿主机的 `data` 目录映射到容器中的 `/home` 目录
```shell
docker run -it -v ~/lxf/data:/home ubuntu /bin/bash
```
`-v` ：挂载一个数据卷
接着，我在容器的 `/home` 目录下创建一个 `lxfdir` 目录
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143655.png)
此时，宿主机的 `data` 目录下也会同步多了一个 `lxfdir` 目录
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143708.png)

这个宿主机的 `data` 目录就叫数据卷。

除了宿主机与容器之间可以进行数据交互外，如果两个容器的目录都映射到同一个宿主机目录，那还可以让多个容器间进行数据共享。

## 数据卷容器
> 数据卷容器也是一个容器，目的是专门用于提供数据卷给其它容器挂载，从而实现多个容器之间同步数据的更新。

### 创建数据卷模板容器
命令：
```shell
docker create -v [容器数据卷目录] --name [容器名字] [镜像名称] [命令(可选)]
```
对`ubuntu` 镜像做了文件映射，得到数据卷目录为 `/data` 的模板容器
```shell
docker create -v /data ubuntu
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143736.png)
注意看该模板容器的状态是 `Created` ，即并没有运行，容器名为  `determined_nightingale`  


### 基于数据卷模板创建容器


命令:
```shell
docker run --volumes-from [数据卷容器id/name] -tid --name [容器名字] [镜像名称] [命令(可选)]
```
创建 `lxfubuntu1` 容器
```shell
docker run -it --volumes-from determined_nightingale --name lxfubuntu1 ubuntu /bin/bash
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143750.png)
在容器的根目录下，会基于上述模板创建了一个名为 `data` 的目录（原 `ubuntu` 镜像中是没有的）


我们再创建一个 `lxfubuntu2` 容器
```shell
docker run -it --volumes-from determined_nightingale --name lxfubuntu2 ubuntu /bin/bash
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143805.png)
此时，我们在 `lxfubuntu1` 的 `data` 目录下创建一个名为 `lxf` 的目录， `lxfubuntu2` 的 `data` 目录中也会同步到相同的数据
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143817.png)
这样，只要我们是基于数据卷模板容器创建出来的容器，就可以得到一个数据共享的 `data` 目录，在该 `data` 目录中对文件的操作，都可以同步到各个由该模板容器创建出来的容器中。


### 与宿主机同步文件
>  数据卷容器可以实现多个容器的数据同步，但是数据是保存在数据卷内，并没有保存到宿主机的文件目录中。
>
> 如果想将宿主机的文件同步到各个容器，可以使用 `docker cp` 将宿主机下的文件拷贝到数据卷容器即可，反之亦然


如：把宿主机当前目录下的 `lxf.sh` 文件，拷贝到数据卷容器 `determined_nightingale` 中 `/data` 目录下
```shell
docker cp lxf.sh determined_nightingale:/data
```
如：把数据卷容器 `determined_nightingale` 中 `/data` 目录下的 `lxf.sh` 文件，拷贝到宿主机 `~/lxf` 目录下
```shell
docker cp determined_nightingale:/data/lxf.txt ~/lxf
```
是不是有人要问了，如果我基于数据卷模板创建容器时，顺带设置数据卷呢？，命令如下所示
```shell
docker run -it -v ~/lxf/data:/data --volumes-from determined_nightingale --name lxfubuntu3 ubuntu /bin/bash
```
很遗憾，只有数据卷配置生效，数据卷容器配置不生效～