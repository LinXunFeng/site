---
title: AppRTC实战
date: 2018-06-07 22:49:36
categories: "Linux"
tags:
  - iOS
  - Android
  - Ubuntu
  - Linux
---

<Excerpt in index | 首页摘要> 

记录AppRTC的搭建过程，实现iOS、Android、Browser同异设备的视频通信

<!-- more -->

<The rest of contents | 余下全文>

> 1、记录AppRTC的搭建过程，实现iOS、安卓、browser同异设备的视频通信
> 2、以下直接以root身份进行操作，所有的需要下载的文件均放置于`/root`目录下，需要的话，可以自行决定存放位置，但是要注意修改相关的配置路径～

## 一、设备配置
- 阿里云ESC服务器 Ubuntu 16.04 64位
- 腾讯云域名

## 二、相关环境

### 1、JDK
```shell
add-apt-repository ppa:openjdk-r/ppa 
apt-get update 
apt-get install openjdk-8-jdk
```

### 2、nodejs
[nodejs官网](https://nodejs.org/en/)
```shell
// 这里的版本8.x可以按自己的需求去修改
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
apt-get install -y nodejs
```
至此已经安装了最新版的nodejs和npm了,可以使用`-v`来查看当前版本

```shell
node -v
npm -v
```

安装`grunt-cli`，后面需要`grunt`来构建房间服务器

```shell
npm -g install grunt-cli
```

### 3、python与python-webtest
```shell
apt-get install python 
apt-get install python-webtest
```



### libevent
```shell
apt-cache search libevent
apt-get install libevent-dev
```

<hr>

## 三、Room Server 房间服务器
```
git clone  https://github.com/webrtc/apprtc.git   
cd apprtc
npm install
```

### 1、配置 constants.py

```shell
# 当前目录 -- apprtc
vim src/app_engine/constants.py
```

```python
# 这部分为 添加
TURN_BASE_URL = 'https://linxunfeng.top' # 修改为你自己当前服务器的域名，下面的亦是如此
TURN_URL_TEMPLATE = '%s/turn.php?username=%s&key=%s' #如果turn.php未实现，可使用默认配置
CEOD_KEY = 'lxf' # 这个很重要，后面配置turn时需要用到

# 这部分为 修改
ICE_SERVER_BASE_URL = 'https://linxunfeng.top'
ICE_SERVER_URL_TEMPLATE = '%s/iceconfig.php?key=%s' #如果iceconfig.php未实现，可用默认配置，但是Android Apk会有问题
ICE_SERVER_API_KEY = os.environ.get('ICE_SERVER_API_KEY')

# Dictionary keys in the collider instance info constant.
WSS_INSTANCE_HOST_KEY = 'linxunfeng.top:8089' #信令服务器端口号8089  
WSS_INSTANCE_NAME_KEY = 'vm_name'
WSS_INSTANCE_ZONE_KEY = 'zone'
WSS_INSTANCES = [{
    WSS_INSTANCE_HOST_KEY: 'linxunfeng.top:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
    WSS_INSTANCE_HOST_KEY: 'linxunfeng.top:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]
```

### 2、编译
```shell
# 当前目录 -- apprtc
grunt build
```
编译好后`apprtc`目录下就会多出一个名为`out`的目录，里面存放的就是编译好的`room server`

### 3、GoogleAppEngine的安装与配置

官网：[GoogleAppEngine](https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Python)

可以在此路径找最新版本

> GoogleAppEngine -> Python -> Download and install the original App Engine SDK for Python.

目前最新版本为：[google_appengine_1.9.70.zip](https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.70.zip)

- 下载 GoogleAppEngine

```shell
wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.70.zip
```

-  解压

```shell
unzip google_appengine_1.9.70.zip
```
- 设置环境变量

```shell
vim /etc/profile
```
```shell
export PATH="$PATH:/root/google_appengine/"
```

- 应用环境变量

```shell
source /etc/profile
```

### 4、开启 Room Server

- 基本命令

```shell
# 当前路径 -- /root/google_appengine
./dev_appserver.py --host=linxunfeng.top ../apprtc/out/app_engine/
```

**如果你使用的是阿里云服务器，这里就不能用域名`linxunfeng.top`来启动`room server`，而是使用本地网卡地址**，否则就会提示

> raise BindError('Unable to bind %s:%s' % self.bind_addr)
> google.appengine.tools.devappserver2.wsgi_server.BindError: Unable to bind linxunfeng.top:8080

- 查看网卡

```shell
ifconfig
```

```shell
root@xxx:~/google_appengine# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:16:3e:08:b4:02  
          inet addr:172.18.141.108  Bcast:172.18.143.255  Mask:255.255.240.0
          ...
```

```shell
./dev_appserver.py --host=172.18.141.108 ../apprtc/out/app_engine/
```

这样就好了吗？不，虽然没有报错，但是你用浏览器打开你的域名看看...
这里直接给出最终命令，具体原因看 [疑难杂症 - 1](#1、Request-host-is-not-whitelist-enabled)

- 最终命令

```shell
./dev_appserver.py --enable_host_checking=false --host=172.18.141.108 ../apprtc/out/app_engine/

# 如果想直接后台运行，则使用如下命令
nohup ./dev_appserver.py --enable_host_checking=false --host=172.18.141.108 ../apprtc/out/app_engine/ &
```

- 访问 room server

```
http://域名:8080
```

## 四、Collider Server 信令服务器

### 1、拷贝collider源码

```shell
# 当前路径 -- /root
mkdir -p goWorkspace/src
```

把`apprtc/src/collider/`目录下的三个目录（collider、collidermain、collidertest）复制到`goWorkspace/src/`目录下

```shell
cp -rf apprtc/src/collider/* /goWorkspace/src
```

### 2、修改代码

- 修改房间服务器的地址

```shell
# goWorkspace/src/collidermain/main.go
var roomSrv = flag.String("room-server", "https://域名", "The origin of the room server")
```

- 修改网站证书路径

```shell
# goWorkspace/src/collider/collider.go

e = server.ListenAndServeTLS("/etc/letsencrypt/live/域名/fullchain.pem", "/etc/letsencrypt/live/域名/privkey.pem")

# 如：
e = server.ListenAndServeTLS("/etc/letsencrypt/live/linxunfeng.top/fullchain.pem", "/etc/letsencrypt/live/linxunfeng.top/privkey.pem")
```

相关的SSL证书`fullchain.pem`和`privkey.pem`在后面的nginx配置中会提到，这里先写上

### 3、安装与配置环境

- 下载GO语言环境

FQ 到 [GO官网](https://golang.org/dl/)上下载最新版本

当前最新版本：[go1.10.2.linux-amd64.tar.gz](https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz)

```shell
# 当前路径 -- /root
wget https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz
```

- 修改profile

```shell
vim /etc/profile
```

打开`profile`后添加如下内容

```shell
export GOROOT=/root/go
export GOPATH=/root/goWorkspace
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

- 应用环境变量

```shell
source /etc/profile
```

### 4、编译

进入目录 `goWorkspace/src/`

```
go get collidermain
go install collidermain
```

这里的编译过程需要翻墙，如果无法翻墙，请看以下内容，如果可以则直接跳至第5小点

```shell
# 当前路径 -- /root/goWorkspace/src
mkdir -p golang.org/x 
cd golang.org/x/
git clone https://github.com/golang/net
```

`git clone`成功后再执行上面的两行编译命令

### 5、开启 Collider Server

进入`goWorkspace`下的`bin`目录，执行命令

```shell
# 当前路径 -- /root/goWorkspace/bin
# -tls=true : 指需要数字证书
./collidermain -port=8089 -tls=true

# 如果想直接后台运行，则使用如下命令
nohup ./collidermain -port=8089 -tls=true &
```

## 五、STUN\TURN服务器

### 1、安装`coturn`

```
apt-get install coturn
```

### 2、修改配置

- coturn

```shell
vim /etc/default/coturn
```

把TURNSERVER_ENABLED=1的注释去掉

- turnserver.conf

```shell
vim /etc/turnserver.conf
```

```shell
listening-device=eth0 #此处eth0是电脑网卡名称
listening-port=3478 #turn服务器的端口号
relay-device=eth0 #此处eth0是电脑网卡名称
min-port=49152
max-port=65535
Verbose
fingerprint
lt-cred-mech
use-auth-secret
static-auth-secret=lxf #此处要和房间服务器配置时constants.py文件中的CODE_KEY保持一致。
user=lxf:0x8638170519dd1309044bca55319ff929
user=lxf:lxf
stale-nonce
cert=/usr/local/etc/turn_server_cert.pem
pkey=/usr/local/etc/turn_server_pkey.pem
no-loopback-peers
no-multicast-peers
mobility
no-cli
```

上述文件中 0x8638170519dd1309044bca55319ff929： 

`turnadmin -k -u lxf -r north.gov -p lxf`

> -k 表示生成一个long-term credential key 
> -u 表示用户名 
> -p 表示密码 
> -r 表示Realm域

coturn的证书生成（即配置文件中cert和pkey)

```shell
openssl req -x509 -newkey rsa:2048 -keyout /usr/local/etc/turn_server_pkey.pem -out /usr/local/etc/turn_server_cert.pem -days 99999 -nodes
```

### 3、启动coturn服务器

```shell
service coturn start
```

用浏览器打开
```
http://域名:3478
```

显示如下内容则说明成功开启服务

> TURN Server 
> use https connection for the admin session

## 六、配置nginx服务器

### 1、生成SSL证书

这里使用let’s encrypt颁发的免费SSL证书 [certbot.eff.org](https://certbot.eff.org)

- 安装软件

```shell
sudo apt-get update  
apt-get install software-properties-common  
add-apt-repository ppa:certbot/certbot  
apt-get update 
apt-get install python-certbot-nginx
```

- 生成证书

```shell
certbot --nginx certonly
```
SSL证书生成的目录为：`/etc/letsencrypt/live/域名/`，里面存放着四个文件`cert.pem,chain.pem,fullchain.pem,privkey.pem`

### 2、安装php和php-fpm

- 安装php和php-fpm

```shell
apt-get install php  
apt-get install php7.0-fpm
```


### 3、安装与配置nginx

- 安装nginx

```shell
apt-get install nginx
```

- 修改nginx配置

```shell
vim /etc/nginx/sites-available/default
```

将下面的`linxunfeng.top`修改为你自己的域名

```shell
upstream roomserver {
       server linxunfeng.top:8080;
}
server {
       listen 80 ;
       server_name linxunfeng.top; 
       return  301 https://$server_name$request_uri;
}
server {
        #listen 80 default_server;
        #listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        listen 443;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name linxunfeng.top; # 添加域名，如不添加，生成SSL证书时可能会有问题

        #location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
        #       try_files $uri $uri/ =404;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }
        location / { 
                proxy_pass http://roomserver$request_uri;
                proxy_set_header Host $host;
        }

        ssl on;
        ssl_certificate /etc/letsencrypt/live/linxunfeng.top/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/linxunfeng.top/privkey.pem;
        
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}
```

### turn.php 与 iceconfig.php

在`/var/www/html/`目录下创建两个文件

```shell
touch turn.php iceconfig.php
```

#### turn.php

```php
<?php 
    $request_username = $_GET["username"];
    if (empty($request_username)) {
        echo "username == null";
        exit;
    }
    $request_key = $_GET["key"];
    $time_to_live = 600;
    $timestamp = time() + $time_to_live;//失效时间
    $response_username = $timestamp.":".$_GET["username"];
    $response_key = $request_key;
    if (empty($response_key)) {
        $response_key = "code_key";
    } //constants.py中CEOD_KEY
    $response_password = getSignature($response_username, $response_key);
    $jsonObj = new Response();
    $jsonObj->username = $response_username;
    $jsonObj->password = $response_password;
    $jsonObj->ttl = 86400;
    //此处需配置自己的服务器
    $jsonObj->uris= array("stun:linxunfeng.top:3478","turn:linxunfeng.top:3478?transport=udp","turn:linxunfeng.top?transport=tcp");
    echo json_encode($jsonObj);
/**
 * 使用HMAC-SHA1算法生成签名值
 *
 * @param $str 源串
 * @param $key 密钥
 *
 * @return 签名值
 */
function getSignature($str, $key)
{
    $signature = "";
    if (function_exists('hash_hmac')) {
        $signature = base64_encode(hash_hmac("sha1", $str, $key, true));
    } else {
        $blocksize = 64;
        $hashfunc = 'sha1';
        if (strlen($key) > $blocksize) {
            $key = pack('H*', $hashfunc($key));
        }
        $key = str_pad($key, $blocksize, chr(0x00));
        $ipad = str_repeat(chr(0x36), $blocksize);
        $opad = str_repeat(chr(0x5c), $blocksize);
        $hmac = pack(
'H*',
    $hashfunc(
($key ^ $opad) . pack(
'H*',
    $hashfunc(
($key ^ $ipad) . $str
)
)
)
);
        $signature = base64_encode($hmac);
    }
    return $signature;
}
    class Response
    {
        public $username = "";
        public $password = "";
        public $ttl = "";
        public $uris = array("");
    }
?> 
```

- iceconfig.php

```php
<?php 
    $request_username = "lxf";  //配置成自己的turn服务器用户名
    if (empty($request_username)) {
        echo "username == null";
        exit;
    }
    $request_key = "lxf";  //配置成自己的turn服务器密码
    $time_to_live = 600;
    $timestamp = time() + $time_to_live;//失效时间
    $response_username = $timestamp.":".$_GET["username"];
    $response_key = $request_key;
    if (empty($response_key)) {
        $response_key = "CEOD_KEY";
    }//constants.py中CEOD_KEY
    $response_password = getSignature($response_username, $response_key);
    $arrayObj = array();
    $arrayObj[0]['username'] = $response_username;
    $arrayObj[0]['credential'] = $response_password;
    //配置成自己的stun/turn服务器
    $arrayObj[0]['urls'][0] = "stun:linxunfeng.top:3478";
    $arrayObj[0]['urls'][1] = "turn:linxunfeng.top:3478?transport=tcp";
    $arrayObj[0]['uris'][0] = "stun:linxunfeng.top:3478";
    $arrayObj[0]['uris'][1] = "turn:linxunfeng.top:3478?transport=tcp";
    $jsonObj = new Response();
    $jsonObj->lifetimeDuration = "300.000s";
    $jsonObj->iceServers = $arrayObj;
    echo json_encode($jsonObj);
/**
 * 使用HMAC-SHA1算法生成签名值
 *
 * @param $str 源串
 * @param $key 密钥
 *
 * @return 签名值
 */
function getSignature($str, $key)
{
    $signature = "";
    if (function_exists('hash_hmac')) {
        $signature = base64_encode(hash_hmac("sha1", $str, $key, true));
    } else {
        $blocksize = 64;
        $hashfunc = 'sha1';
        if (strlen($key) > $blocksize) {
            $key = pack('H*', $hashfunc($key));
        }
        $key = str_pad($key, $blocksize, chr(0x00));
        $ipad = str_repeat(chr(0x36), $blocksize);
        $opad = str_repeat(chr(0x5c), $blocksize);
        $hmac = pack(
'H*',
    $hashfunc(
($key ^ $opad) . pack(
'H*',
    $hashfunc(
($key ^ $ipad) . $str
)
)
)
);
        $signature = base64_encode($hmac);
    }
    return $signature;
}
    class Response
    {
        public $username = "";
        public $password = "";
        public $ttl = "";
        public $uris = array("");
    }
?> 
```

- 重启Nginx服务器和php7.0-fpm

```shell
service nginx restart 
service php7.0-fpm restart
```

## 七、各端运行测试

### 1、Browser

浏览器直接打开即可，如果不能访问摄像头之类的，请查看[疑难杂症 - 2](#2、浏览器无法访问摄像头)

```
https://域名
```

### 2、Android

[webrtc-android-demo-apprtc](https://github.com/duqian291902259/webrtc-android-demo-apprtc)

直接安装手机，打开软件，点击右上角的扳手跳转至设置界面，滚到界面最下方，找到`Room server URL.`，将其修改为刚刚搭建好的服务器域名即可，如

```
https://linxunfeng.top
```

### 3、iOS

[GitHub - ISBX/apprtc-ios](https://github.com/ISBX/apprtc-ios)

打开`AppRTC`项目，分别修改以下两个文件

`ARTCVideoChatViewController.m`

```objc
#define SERVER_HOST_URL @"https://linxunfeng.top"
```

`Pods -> Development Pods -> AppRTC -> Lib -> ARDAppClient.m`

```objc
static NSString *kARDRoomServerHostUrl =
    @"https://linxunfeng.top";
static NSString *kARDDefaultSTUNServerUrl =
    @"stun:linxunfeng.top:3478";
static NSString *kARDTurnRequestUrl =
    @"https://linxunfeng.top"
    @"/turn?username=lxf&key=lxf";
```

```objc
// 搜索 kARDDefaultSTUNServerUrl 
- (RTCICEServer *)defaultSTUNServer {
  NSURL *defaultSTUNServerURL = [NSURL URLWithString:kARDDefaultSTUNServerUrl];
  return [[RTCICEServer alloc] initWithURI:defaultSTUNServerURL
                                  username:@"lxf"
                                  password:@"lxf"];
}
```


## 疑难杂症
### 1、Request host is not whitelist enabled

> 具体提示
> Request host is not whitelist enabled for this server. Please use the --host command-line flag to whitelist a specific host (recommended) or use --enable_host_checking to disable host checking. See the command-line flags help text for more information. 

[参考链接](https://stackoverflow.com/questions/47988810/use-ngrok-url-as-callback-url-for-facebook-webhook-but-is-recognized-not-white)

执行dev_appserver.py时，加上如下参数
```shell
 --enable_host_checking=false
```

完整指令
```shell
./dev_appserver.py --enable_host_checking=false --host=172.18.141.108 ../apprtc/out/app_engine/
```

### 2、浏览器无法访问摄像头

浏览器访问设备的摄像头是需要使用`https`链接或者`localhost`来访问

### 3、端口
阿里云等国内大厂提供的服务器基本上都有一个叫安全组的玩意儿，我们搭建AppRTC服务器的所有服务所需的端口均要添加至安全组

### 4、Failed to execute ‘pushState’ on ‘History’

> Failed to start signaling: Failed to execute ‘pushState’ on ‘History’: A history state object with URL ‘http://linxunfeng.top/r/598600855’ cannot be created in a document with origin ‘https://linxunfeng.top’ and URL ‘https://linxunfeng.top/

解决方法有两种



- 解决方法1： 

>  房间服务器编译完成后，在/root/apprtc/out/app_engine/js/apprtc.debug.js文件中找到window.history.pushState({‘roomId’: roomId, ‘roomLink’: roomLink}, roomId, roomLink)，把这句话注释掉，重新运行即可。（如果重新编译，需要重新修改）



- 解决方法2： 

>  在/root/apprtc/src/web_app/js/appcontroller.js文件中找到window.history.pushState({‘roomId’: roomId, ‘roomLink’: roomLink}, roomId, roomLink)，把这句话注释掉，然后重新编译，重新运行房间服务器即可。