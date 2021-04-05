---
title: Docker - Compose的使用
date: 2021-04-05 14:27:52
permalink: /pages/e3acb4/
categories:
  - Docker
tags:
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---

## 一、服务编排

> 服务编排：按照一定的业务规则批量管理容器

微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启动和停止，那么维护的工作量就会很大。

比如这些工作：

- 要从 `Dockerfile build image` 或者去 `dockerhub` 拉取 `image` 
- 要创建多个 `container` 
- 要管理这些 `container` (启动停止删除)

这时候需要使用官方推出的服务编排



## 二、Docker Compose

> `Docker Compose` 是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止

使用步骤：

1. 利用 `Dockerfile` 定义运行环境镜像
2. 使用 `docker-compose.yml` 定义组成应用的各服务（启动顺序、关联关系等）
3. 运行 `docker-compose up` 启动应用



### 1、安装

如果使用的是 [Docker Desktop](https://www.docker.com/products/docker-desktop)，那就不需要再额外安装 `Compose` 了，因为它已经包含了 `Docker Compose` 。

下面是 `Linux` 系统下安装 `Compose` 的方式

```shell
# 以编译好的二进制包方式安装到Linux系统中
$ curl -L https://github.com/docker/compose/releases/download/1.28.6/run.sh > /usr/local/bin/docker-compose

# 设置文件可执行权限
$ chmod +x /usr/local/bin/docker-compose

# 查看版本信息（-v 或 --version 都可以）
$ docker-compose --version
$ docker-compose -v
```



### 2、卸载

```shell
# 二进制包方式安装的，删除二进制文件即可
rm /usr/local/bin/docker-compose
```



### 3、实战

先来编写配置文件，新建一个 `docker-compose.yml` 文件，文件名是固定的

```shell
touch docker-compose.yml
```

这里我用 `VSCode` 打开进行编辑

```shell
code docker-compose.yml
```

文件内容：

```yaml
version: '3'
services: 
  nginx: # 服务名字自行定义
    image: nginx # 指定通过nginx该镜像进行启动，这里可以指定镜像的版本号，如果不指定，默认为latest
    ports: # 端口映射，即 -p 参数，将宿主机的80端口映射到容器的80端口
      - 8090:80
    links: # 指明当前服务可以访问到jenkins这个服务
      - jenkins
    volumes: # 目录映射，即 -v 参数，（宿主机目录:容器目录）
      - ~/data/lxf/nginx/conf.d:/etc/nginx/conf.d
  jenkins: # 服务名字自行定义
    image: jenkins
    expose:  # 暴露指定的端口
      - "8080"
      - "5000"
```

创建 `~/data/lxf/nginx/conf.d` 目录，并进入该目录

```shell
mkdir -p ~/data/lxf/nginx/conf.d
cd ~/data/lxf/nginx/conf.d
```

在该目录中创建一个 `conf` 文件，比如 `lxf.conf` ，文件内容如下

```
server {
  listen 80;
  access_log off;
  
  location / {
    proxy_pass http://jenkins:8080;
  }
}
```

配置反向代理，当访问 `80` 端口时会反向代理到 `http://jenkins:8080`，这里的 `jenkins` 就是上述 `docker-compose.yml` 文件中 `links` 指定的 `jenkins` 



在 `docker-compose.yml` 文件所在目录，使用 `docker-compose` 启动容器

```shell
# -d: 后台启动
docker-compse up [-d]
```

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource20210320170901/image/20210405143327.png)

测试访问

```
127.0.0.1:8090
或
本机ip:8090
```

![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource20210320170901/image/20210405143344.png)

## 三、引用

[Docker Compose 官方文档](https://docs.docker.com/compose/) 



