---
title: Docker - 操作镜像资源
date: 2021-01-24 22:38:10
permalink: /pages/084bf9/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---



![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224049.jpeg)

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224119.png)


## 搜索镜像资源
```shell
docker search nginx
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224150.png)
一般选择 `STARS` 数最大的


## 拉取镜像
比如拉取上提及的 `nginx` 镜像
```shell
docker image pull nginx
```
命令执行后就开始对镜像进行拉取了
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224211.png)


## 查看当前拥有的镜像
> 镜像ID: 镜像的唯一标识，如果镜像ID相同，则说明是同一个镜像
>
> TAG: 用来区分不同的发行版本，如果不指定具体标记，则默认使用latest来标记信息

```shell
docker image ls
# 或
docker images
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224239.png)


## 查看镜像的详情信息
```shell
# docker image inspect 镜像名
docker image inspect ubuntu
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224256.png)


## 删除镜像
完整写法
```shell
docker image rm ubuntu
```
简洁写法
```shell
docker rmi ubuntu
```
除了可以根据镜像名来删除外，也可以使用镜像ID，如使用上述的 `ubuntu` 镜像ID
```shell
docker rmi f643c72bc252
```
如果我们对同一个镜像打了多个 `tag` ，导致同一个镜像ID存在多个镜像名称，那此时可以使用 `name:tag` 的格式来删除镜像，如：
```shell
docker rmi ubuntu:latest

# docker rmi ubuntu_lxf:v1.0
```


## 镜像标签
```shell
# docker tag 当前镜像名:镜像版本 新的镜像名:新的版本
docker tag ubuntu:latest ubuntu_lxf:latest

# docker tag ubuntu:latest ubuntu_lxf:v1.0
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224315.png)
注：

- `ubuntu` 和 `ubuntu_lxf` 的 `IMAGE_ID` 是相同的
- 结合删除镜像的命令就可以实现镜像重命名功能



## 导出镜像
> docker save 会保存镜像的所有历史记录和元数据信息

```shell
# docker save -o 包文件 镜像
docker save -o ubuntu.tar ubuntu

# docker save 镜像1 ... 镜像n > 包文件
docker save ubuntu nginx > lxf_images.tar
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224337.png)


## 导入镜像
先删除 `ubuntu` 镜像
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224351.png)

```shell
# docker load -i 镜像包名
docker load -i ubuntu.tar

# docker load < 镜像包名
docker load < ubuntu.tar

# docker load --input 镜像包名
docker load --input ubuntu.tar
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224416.png)
可以看到， `ubuntu` 镜像已经成功导入进来了


## 查看镜像历史
```shell
# docker image history 镜像名

docker image history ubuntu
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210124224432.png)
