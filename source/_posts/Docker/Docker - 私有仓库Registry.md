---
title: Docker - 私有仓库Registry
date: 2021-01-31 14:23:19
permalink: /pages/bfa843/
categories: 
  - Docker
tags: 
  - Docker
author: 
  name: LinXunFeng
  link: https://github.com/LinXunFeng
---


![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143241.jpeg)



> 私有仓库: 在本地（局域网）搭建的一个类似公共仓库的东西，我们可以将镜像提交到私有仓库中，供局域网内的其它人拉取使用。
> 本文以 `Registry` 为例，并在提供私有仓库的主机上操作



## 拉取私有仓库镜像
请先确保你当前拥有的镜像有 `registry` 
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143259.png)
如果没有，可以先拉取下来

```shell
docker image pull registry
```
## 设置私有仓库地址
```shell
vim /etc/docker/daemon.json
```
修改 `insecure-registries` 的值，提供私有仓库的主机的ip地址和端口
```json
{
  ...
  "insecure-registries":[
    "192.168.1.234:5000"
  ],
  ...
}
```
Mac软件版
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143315.png)
**修改后重新启动 `docker` 服务**


## 运行私有仓库镜像资源
将 `registry` 镜像生成一个容器并运行起来
```shell
# -p 5000:5000 
# 第一个是容器使用的端口，第二个是本地端口，这里是本地端口映射到把容器的端口
docker run -d -p 5000:5000 registry
```
```shell
~/lxf ❯ docker ps
❯ docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS      NAMES
f12ad7ae43ca   registry   "/entrypoint.sh /etc…"   9 minutes ago   Up 9 minutes   5000/tcp   nostalgic_elion
```
此时你可以访问如下地址，如果看到 `{}` 就说明 `Registry` 运行正常
```shell
http://192.168.1.234:5000/v2/
```


## 上传镜像


比如此时我要将 `ubuntu` 这个镜像上传到私有仓库
```shell
# 给ubuntu镜像打一个tag，命名需为 私有仓库主机ip:端口/镜像名:[版本号,不加默认为latest]
docker tag ubuntu:latest 192.168.1.234:5000/ubuntu:v0.1
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143336.png)
开始上传镜像至本地的私有仓库中

```shell
# docker push <registry_ip>:<registry_port>/<image_name>:<image_tag>
docker push 192.168.1.234:5000/ubuntu:v0.1
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143349.png)
## 拉取镜像
先将本地的 `v0.1` 删掉
```shell
docker rmi 192.168.1.234:5000/ubuntu:v0.1
```
拉取私有仓库中 `ubuntu` 的 `0.1` 版本镜像
```shell
# docker pull <registry_ip>:<registry_port>/<image_name>:<image_tag>
docker pull 192.168.1.234:5000/ubuntu:v0.1
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143403.png)
## 搜索镜像
`Registry` 不支持通过 `docker search` 这种方式去搜索镜像，会报 `404` 的错误
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143415.png)
需要使用 `V2 API` 去查询


### 列出仓库中所有的镜像
```shell
curl 192.168.1.234:5000/v2/_catalog
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143430.png)
### 列出指定镜像的所有标签
```shell
# curl -X GET http://<registry_ip>:<registry_port>/v2/<image_name>/tags/list
curl 192.168.1.234:5000/v2/lxf/tags/list
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143447.png)
## 删除镜像
查找指定标签的镜像的 `digest` ，再根据这个 `digest` 来删除，以删除 `lxf:0.2` 镜像为例


先执行命令找到该镜像的 `digest` 
```shell
curl -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET  http://192.168.1.234:5000/v2/lxf/manifests/0.2 2>&1 | grep Docker-Content-Digest | awk '{print ($3)}'
```
得到输出值
```shell
sha256:4e4bc990609ed865e07afc8427c30ffdddca5153fd4e82c20d8f0783a291e241
```
根据 `digest` 来删除镜像
```shell
curl -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE http://192.168.1.234:5000/v2/lxf/manifests/sha256:4e4bc990609ed865e07afc8427c30ffdddca5153fd4e82c20d8f0783a291e241
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143500.png)
这个时候只是删除镜像的元数据，并没有真正从硬盘上删除镜像，需要执行垃圾回收才行。


### 删除失败
遇到 `405 UNSUPPORTED` 错误
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143512.png)


需要在运行 `Registry` 容器时设置`REGISTRY_STORAGE_DELETE_ENABLED` 为 `true` 

举例
`docker-compose.yaml`：设置环境变量

```yaml
environment:
    REGISTRY_STORAGE_DELETE_ENABLED: "true"
```


`docker run`：添加参数
```shell
# -e REGISTRY_STORAGE_DELETE_ENABLED="true"
docker run -d -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED="true" registry
```


## 垃圾回收
> 执行垃圾回收，上述删除的镜像才会真正从硬盘上移除

```shell
docker exec -it registry的容器名 /bin/registry garbage-collect /etc/docker/registry/config.yml
```
![](https://cdn.jsdelivr.net/gh/FullStackAction/PicBed@resource/image/20210131143530.png)
