---
title: Docker - Dockerfile的使用
date: 2021-01-31 14:23:49
permalink: /pages/05278d/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---


> Dockerfile: 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
>
> 应用: 将部署过程中涉及到的所有步骤全部写入到 `Dockerfile` 中，到时只需要执行 `Dockerfile` 就可以自动完成相应的操作
## 快速入门
### 编辑Dockerfile


在当前目录下创建 `Dockerfile` 并进行编辑
```shell
vim Dockerfile
```
操作：启动 `ubuntu` 镜像，在启动起来后去更新 `ubuntu` 容器下的软件资源
内容如下
```dockerfile
# From: 启动运行一个镜像资源
From ubuntu
# Run: 在启动起来的容器中执行指令
RUN apt-get update
```
### 运行Dockerfile
构建镜像命令
```shell
docker build -t [镜像名]:[版本号] [Dockerfile所在目录]
```
指定在当前目录下去查找 `Dockerfile` 文件，并将 `Dockerfile` 自动化处理后的（更新了软件资源）容器打成名为 `lxf` 的镜像资源

```shell
docker build -t lxf .
```
执行效果：
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145009.png)
待执行完成后，当前拥有的镜像资源就多出了 `lxf` 这一个，可以看到 `lxf` 这个镜像的大小是要比 `ubuntu` 的镜像要大一点的
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145041.png)

## 基础指令详解


### FROM
格式：
```dockerfile
FROM <image>
FROM <image>:<tag>
```
说明：

- `FROM` 是 `Dockerfile` 里的第一条而且只能是除了首行注释之外的第一条指令



### RUN
格式：
```dockerfile
# shell模式
RUN <command>

# exec模式
RUN ["executable", "param1", "param2"]
```
说明：

- 表示当前镜像构建时候运行的命令

注释：
```dockerfile
# shell 模式：类似于 /bin/bash -c command 
RUN echo hello 

# exec 模式：类似于 RUN ["/bin/bash", "-c", "command"] 
RUN ["echo", "hello"]
```


执行多条指令

- 一条条指令写
```dockerfile
RUN echo hello
RUN echo world
```

- 将指令用 `&&` 连接起来
```dockerfile
RUN echo hello && echo world
```


### MAINTAINER
格式：
```dockerfile
MAINTAINER <name>
```
说明：

- 指定该 `Dockerfile` 文件的维护者信息



### EXPOSE
> 设置容器对外开放的端口

格式：
```dockerfile
EXPOSE <port> [<port>...]
```
解释：

- 设置 `Docker` 容器对外暴露的端口号， `Docker` 为了安全，不会自动对外打开端口，如果需要外部提供访问，还需要启动容器时增加 `-p` 或者 `-P` 参数对容器的端口进行分配。



### ENTRYPOINT
> 设置容器在启动后去执行一个命令

格式：
```dockerfile
# exec 模式
ENTRYPOINT ["executable", "param1","param2"]

# shell模式
ENTRYPOINT command param1 param2
```
解释：

- 每个 `Dockerfile` 中只能有一个 `ENTRYPOINT` ，当指定多个时，只有最后一个起效。



`EXPOSE` 和 `ENTRYPOINT` 结合使用的例子，可以全文看完后再回到这里看该例子
```dockerfile
# 使用django镜像资源
From django
# 切换目录
WORKDIR /home
# 创建一个名为lxf的django项目
RUN django-admin startproject lxf
# 切换目录
WORKDIR /home/lxf
# 对外开放8000端口
EXPOSE 8000
# 容器启动后，将django服务开启，并指定端口号为8000
ENTRYPOINT python3 manage.py runserver 0.0.0.0:8000
```
执行构建镜像命令
```shell
docker build -t lxf .
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145109.png)
现在我们将构建好的 `lxf` 容器运行起来，并随机分配端口

```shell
docker run -dit -P lxf
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145121.png)
可以看到， `Docker` 为我们随机分配了 `55001` 端口映射到容器的 `8000` 端口，并且可以正常访问到容器的 `django` 服务
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145133.png)

## 文件指令详解
### ADD
格式：
```dockerfile
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```
说明：

- 将指定的 `<src>` 文件复制到容器文件系统中的 `<dest>` 
- `src` 指的是宿主机，`dest`  指的是容器
- 如果源文件是个压缩文件，则 `Docker` 会自动帮解压到指定的容器中(无论目标是文件还是目录，都会当成目录处理)。



如：将宿主机下的 `./data` 目录下的所有文件(夹)，全部复制到容器的 `/home` 目录下
```dockerfile
From ubuntu
ADD ./data /home
```
注：目录本身即 `data` 目录并不会复制到容器中，只复制 `data` 目录下的文件(夹)，如果想连同 `data` 文件夹也复制过去，可以修改为如下指令
```dockerfile
From ubuntu
ADD ./data /home/data
```


### COPY
格式：
```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```
解释：

- 单纯复制文件场景， `Docker` 推荐使用 `COPY` 
- 如果源文件是个压缩文件， `Docker` 会直接将压缩文件复制进容器内，不会像 `ADD` 那样先解压再复制



注： `COPY` 与 `ADD` 基本上是一样的，只是面对源文件是压缩文件时处理方式不同而已， `ADD` 会先解压再将解压后的内容复制到容器， `COPY` 不会进行解压，而是直接将压缩包复制过去



## 环境指令详解
### ENV
> 设置环境变量

格式：
```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```
解释：

- 设置环境变量，可以在 `RUN` 之前使用，然后 `RUN` 命令时调用，容器启动时这些环境变量都会被指定



如：设置了环境变量 `name` ，并赋值为 `lxf` ，使用 `RUN` 命令打印 `name` 变量的值，可以成功打印出来
```dockerfile
ENV name=lxf
RUN echo $name # 会打印出lxf
```
并且，当进入容器后也可以正常打印出该变量的值
```shell
root@5721971f92e4:/# echo $name
lxf
```


### WORKDIR
> 切换目录

格式：
```dockerfile
WORKDIR /path/to/workdir
```
解释：

- 切换目录，为后续的 `RUN` 、 `CMD` 、 `ENTRYPOINT` 指令配置工作目录。 
- 相当于 `cd` 命令，可以使用多个 `WORKDIR` 指令进行多次切换，后续命令如果参数是相对路径，则会基于之前命令指定的路径
- 如果我们指定切换到一个不存在的目录， `Docker` 会帮我们自动创建相应的目录

举例：
```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
最终路径为 `/a/b/c` 


如果我们想要容器被运行起来时，自动进入到 `/home` ，可以按如下指令设置
```dockerfile
 From ubuntu
 WORKDIR /home
```
执行 `Dockerfile` 构建镜像完成后运行起来，此时容器便会自动进入到 `/home` 目录
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145219.png)
对比一下之前没有设置使用过 `WORKDIR` ，运行起来的容器，会默认进入到根目录
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131145231.png)










