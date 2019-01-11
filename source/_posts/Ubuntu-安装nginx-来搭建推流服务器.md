---
title: Ubuntu安装nginx来搭建推流服务器
date: 2017-09-12 09:54:14
categories: "Linux"
tags:
  - Ubuntu
  - Linux
---

<Excerpt in index | 首页摘要> 

如果安装命令回车之后出现如下信息，请参考[【Ubuntu “无法获得锁”解决方案】](http://www.linuxidc.com/Linux/2009-07/20740.htm)解决，但是我亲测对我没用，直接重启搞定

+<!-- more -->

<The rest of contents | 余下全文>

## 安装nginx
安装两个依赖库
```shell
sudo apt-get install autoconf automake
sudo apt-get install libpcre3 libpcre3-dev
```
安装zlib库
```shell
sudo apt-get install openssl
sudo apt-get install libssl-dev
```
如果安装命令回车之后出现如下信息，请参考[【Ubuntu “无法获得锁”解决方案】](http://www.linuxidc.com/Linux/2009-07/20740.htm)解决，但是我亲测对我没用，直接重启搞定
```
E: 无法获得锁 /var/lib/dpkg/lock - open (11: 资源暂时不可用)
E: 无法锁定管理目录(/var/lib/dpkg/)，是否有其他进程正占用它？
```

进入家目录，新建一个文件夹，这里以lxf为例
```
cd ~
mkdir lxf
```

下载所需源码
```shell
// 下载nginx-rtmp源码
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
// 修改压缩包的名字
mv master.zip module.zip
```

```shell
// 下载nginx
wget https://github.com/nginx/nginx/archive/master.zip
```

```shell
// 下载nginx的依赖pcre源码
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
```

下载完成之后使用unzip命令进行解压
```shell
unzip master.zip 
unzip module.zip
tar -zxvf pcre-8.39.tar.gz
```
配置编译文件，准备编译安装
```shell
// 先进入nginx-master目录
cd nginx-master/
```
在nginx-master目录下有一个auto文件夹，里面有一个名为configure的配置文件，我们先来通过它进行一些配置
```shell
// prefix:指定安装目录
// add-module:指定模块文件夹
auto/configure --prefix=/usr/local/nginx --with-pcre=../pcre-8.39 --with-http_ssl_module --with-http_v2_module --with-http_flv_module --with-http_mp4_module --add-module=../nginx-rtmp-module-master/
```
配置好之后会多出一个Makefile文件(一种配置文件，定义了一系列的规则来指定编译操作)与objs文件夹
![](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/1.png)

```shell
// 编译
make
// 安装
sudo make install
```

当你make后，看到则代表编译成功
![make成功](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/2.png)
再执行【sudo make install】，看到这个则代表安装完成
![安装完成](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/3.png)

现在我们去测试一下
```shell
 cd /usr/local/nginx/sbin/
sudo ./nginx -t
```
看到successful说明配置文件正确！，如果是failed的话看看你是不是没加sudo
![配置文件正确](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/4.png)

启动nginx服务器
```shell
sudo ./nginx
```
默认端口是80，所以直接到浏览器中直接敲入本地地址 127.0.0.1，显示【Welcome to nginx!】就代表nginx已经成功安装
![](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/5.png)

## 配置nginx
创建推流存放文件夹
```
// 为里以 /usr/local/var/www/hls 为例
cd /usr/local
sudo mkdir -p var/www/hls
```
进入nginx的conf目录，使用vim编辑nginx.conf文件
```shell
cd /usr/local/nginx/conf
sudo vim nginx.conf
```
配置Nginx，支持http协议拉流
```shell
location /hls {
  types {
    application/vnd.apple.mpegurl    m3u8;
    video/mp2t ts;
  }
  root /usr/local/var/www;
   add_header Cache-Control    no-cache;
}
```

配置Nginx，支持rtmp协议推流
```shell
rtmp {
    server {
        listen 1935;
        application rtmplive {
            live on;
            max_connections 1024;
        }
        application hls{
            live on;
            hls on;
            hls_path /usr/local/var/www/hls;
            hls_fragment 1s;
        }
    }
}
```

![hls](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/6.png)

![rtmp](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/7.png)

重启nginx服务器
```shell
cd /usr/local/nginx/sbin/
sudo ./nginx -s reload
```

如果执行【sudo ./nginx -s reload】出现下面这个问题
```shell
nginx: [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
```
就使用nginx -c的参数指定nginx.conf文件的位置，接着再reload一下就好了
```shell
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```


我Ubuntu地址为192.168.123.191

推流至RTMP到服务器  [rtmp://192.168.123.191:1935/rtmplive/lxf](rtmp://192.168.123.191:1935/rtmplive/lxf)
```
ffmpeg -re -i 异形.契约.mp4 -vcodec libx264 -vprofile baseline -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 rtmp://192.168.123.191:1935/rtmplive/lxf
```
推流至HLS到服务器  http://192.168.123.191/hls/lxf.m3u8
```
ffmpeg -re -i 异形.契约.mp4 -vcodec libx264 -vprofile baseline -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 rtmp://192.168.123.191:1935/hls/lxf
```

如果出现如下错误说明你的电脑没安装ffmpeg
```
-bash: ffmpeg: command not found
```
使用Homebrew来安装FFmpeg
```shell
// 安装Homebrew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
// 安装FFmpeg
brew install ffmpeg
```

开始推流，终端上就开始不断的刷新推流信息
![开始推流](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/8.png)


我们可以用电脑上的VLC这个软件来测试是否推流成功
![VLC](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/9.png)

![打开流](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/10.png)

OK，rmtp打开正常，hls就不演示了，一样的
![rmtp打开成功](https://linxunfeng.github.io/images/2017/09/Ubuntu安装nginx来搭建推流服务器/11.png)