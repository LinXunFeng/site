---
title: Docker - 操作容器
date: 2021-01-31 14:22:18
permalink: /pages/861244/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---


![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131142731.jpeg)



![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131142751.png)



## 运行容器


这里我使用 `ubuntu` 镜像，创建一个名为 `lxfubuntu` 的容器，并运行进入容器
```shell
# docker run -it --name [容器名] [镜像名] /bin/bash

docker run -it --name lxfubuntu ubuntu /bin/bash
```
命令参数详解：

- `--name` : 定义容器名称，如果不使用，则会随机产生一个名字
- `-i` : 让容器的标准输入保持打开
- `-t` : 让 `docker` 分配一个伪终端，并绑定到窗口的标准输入上
- `-d` : 以守护进程的方式运行容器，不占用终端
- `/bin/bash` : 执行一个命令



如图，执行后即可进入容器中
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131142815.png)


如果想创建并以守护进程的方式运行容器，可以使用 `-d` 
```shell
docker run -dit --name lxfubuntu1 ubuntu /bin/bash
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131142828.png)


```shell
# docker start [container_id]

docker start c97abc3f151a
```


## 进入容器
如果我们想进入上述提及的以守护进程方式运行的容器中，可以使用 `docker exec` 
```shell
docker exec -it lxfubuntu1 /bin/bash
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131142848.png)
如果在此时退出该守护进程的容器时，该容器依旧在后台运行


## 退出容器

两种方式

- 方法一： 终端输入 `exit` 后回车
- 方法二： `Ctrl + D` 


如果容器是以守护进程的方式运行，在进入容器后退出，不停止容器的运行，即依旧在后台运行，否则将在容器退出时顺带停止容器


## 停止容器
```shell
# docker stop [container_id|container_name]

docker stop lxfubuntu1
# 或
docker stop c97abc3f151a
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143004.png)
## 启动容器
```shell
# docker start [container_id|container_name]

docker start lxfubuntu1
# 或
docker start c97abc3f151a
```
## 重启容器
```shell
# docker restart [container_id|container_name]

docker restart lxfubuntu1
# 或
docker restart c97abc3f151a
```


## 查看容器


查看正在运行的容器
```shell
docker container ls
# 或
docker ps
```


查看所有的容器，包括已经停止的容器
```shell
docker container ls -a
docker container ls --all
# 或
docker ps -a
docker ps --all
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143019.png)


如果只是想查看容器的ID，可以使用参数 `-q` 
```shell
docker ps -aq
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143032.png)
## 删除容器
```shell
# docker [container] rm [container_id|container_name]

docker container rm lxfubuntu
# 或
docker rm lxfubuntu1
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143045.png)

如果删除一个正在运行的容器，需要加 `-f` 
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143103.png)


结合上述查看容器中 `-q` 的功能，可以实现批量删除容器的功能
```shell
# docker ps -aq 获取所有的容器id
docker rm -f $(docker ps -aq)
```
## 查看容器详情
```shell
# docker inspect [container_id|container_name]

docker inspect lxfubuntu1
```
## 查看容器状态
```shell
# docker container stats [container_id|container_name]

docker container stats lxfubuntu1
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143123.png)
## 打包容器为新的镜像资源
> 容器在安装了各种各样的服务后，将该容器打包为镜像资源，接着将该镜像资源进行打包，最后再发给其它人使用

将 `lxfubuntu1` 这个容器打包成名为 `my_image` 的镜像
```shell
# docker commit 容器名 镜像资源名
docker commit lxfubuntu1 my_image
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143137.png)


