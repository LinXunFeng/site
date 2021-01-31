---
title: Docker - 网络管理
date: 2021-01-31 14:23:39
permalink: /pages/337d0f/
categories: 
  - 后端
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131150136.jpeg)



> 默认情况下，容器和宿主机之间网络是隔离的 

现在启动了一个 `nginx` 容器
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143943.png)
如图所示 `nginx` 使用了 `80` 端口，但是我们去浏览器里访问 `localhost:80` 是无法访问到 `nginx` 服务的
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143955.png)
这个时候可以通过端口映射的方式，将容器中的端口，映射到宿主机的某个端口上，从而使我们能够通过宿主机的 `ip+端口` 的方式来访问容器里的内容


## 随机端口映射


> `-P` ：自动绑定所有对外提供服务的容器端口，映射的端口会从没有使用的端口池中自动随机选择，如果连续启动多个容器的话，则下一个容器的端口默认是当前容器的端口号+1

命令
```shell
docker run -d -P [镜像名称]
```
在刚才的命令基本上加 `-P` 
```shell
docker run -dit -P --name lxfnginx nginx
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144009.png)
这样宿主机的 `55000` 端口就映射到了窗口的 `80` 端口，访问也是成功的。
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144026.png)


## 指定端口映射


命令
```shell
docker run -d -p [宿主机ip]:[宿主机端口]:[容器端口] --name [容器名称] [镜像名称]
```
将宿主机上的 `8000` 端口映射到 `nginx` 容器的 `80` 端口
```shell
docker run -dit -p 8000:80 --name lxfnginx nginx
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144045.png)
再去访问 `localhost:8000` 时也是可以正常访问到的
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144059.png)

`0.0.0.0` 表示地址全匹配，即 `127.0.0.1` 或者宿主机的局域网IP `192.168.1.x` 这种都可以访问到 `nginx` 服务，如果我们想指定绑定的 `IP` ，可以在映射端口前加上 `IP` 即可

```shell
docker run -dit -p 192.168.1.234:8000:80 --name lxfnginx nginx
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144117.png)
虽然图中显示的是 `127.0.0.1:55000->80` ，但是经过测试

- `192.168.1.234:8000` 是可以正常访问
- `127.0.0.1:55000` 和 `127.0.0.1:8000` 无法访问

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144134.png)

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131144148.png)

## 多端口映射
命令
```shell
docker run -d -p [宿主机端口1]:[容器端口1]  -p [宿主机端口2]:[容器端口2] --name [容器名称] [镜像名称] 
```
如下命令所示
```shell
docker run -d -p 7000:443 -p 8000:80 --name lxfnginx nginx
```
## 共享网络

> 共享网络: 容器与宿主机共享网络信息

默认情况下，容器的网络信息与宿主机是相互独立的，这样会有什么问题呢？


假如运行一个提供了 `django` 服务的容器，宿主机本地提供了 `MySql` 服务，此时容器想访问宿主机的数据库，是无法访问到的，这个时候容器就需要共享宿主机的网络


在启动容器时，加上 `--network=host` 即可共享网络，命令如下
```shell
docker run -dit --network=host lxf_django
```
这样，容器与宿主机的网络就是共享的状态，此时的容器也就可以访问到宿主机的数据库了
