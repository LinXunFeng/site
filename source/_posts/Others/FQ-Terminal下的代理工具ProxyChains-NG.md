---
title: FQ-Terminal下的代理工具ProxyChains-NG
date: 2019-01-01 17:11:51
categories: "Others"
tags:
  - FQ
  - Terminal
---

<Excerpt in index | 首页摘要> 

是不是你也遇到过，在 `Shadowsocks` 使用 `全局` 模式下，终端依旧无法 `ping` 通谷歌？这是因为 `Shadowsocks` 仅针对代理应用软件，但是一些终端下执行的命令是无法代理的。所以本篇就来介绍一下如何使你的终端也走代理进行访问。  

<!-- more -->

<The rest of contents | 余下全文>



> 是不是你也遇到过，在 `Shadowsocks` 使用 `全局` 模式下，终端依旧无法 `ping` 通谷歌？这是因为 `Shadowsocks` 仅针对代理应用软件，但是一些终端下执行的命令是无法代理的。所以本篇就来介绍一下如何使你的终端也走代理进行访问。



## 方案一：终端下的all_proxy

> 这里以 `zshrc` + `Shadowsocks` 为例
>
> - 打开 `Shadowsocks`，模式选为 `PAC自动模式` 或 `全局模式` 
>
> - 如果不是使用 `.zshrc` 就 编辑 `~/.bashrc`，下面的同理



### 1、打开 `.zshrc`

```shell
vim ~/.zshrc
```



### 2、添加命令

```shell
alias proxy='export all_proxy=socks5://127.0.0.1:1086'
alias unproxy='unset all_proxy'
```



### 3、使用

先应用一下配置

```shell
source ~/.zshrc
```

终端下敲入

```shell
proxy
```

这样就应用上代理了，使用 `curl` 获取一下 `cip.cc` 来查看当前所使用的 `ip`

```
curl cip.cc
```

如果不想使用代理了，就使用如下命令

```shell
unproxy
```



### 4、总结

这种方式我个人是亲测无效的，不知道是不是我人品问题，还是我的 MAC 有问题，有兴趣的小伙伴可以试试。



## 方案二：ProxyChains-NG

proxychains-ng是proxychains的加强版，主要有以下功能和不足：

- 支持http/https/socks4/socks5
- 支持认证
- 远端dns查询
- 多种代理模式
- 不支持udp/icmp转发
- 少部分程序和在后台运行的可能无法代理

详情可见 [GitHub地址](https://github.com/rofl0r/proxychains-ng)



### 环境

这里以 `MAC` + `homebrew` 为例，如果你还没有安装的话可以参考下方命令，详细可见官网 [Homebrew](https://brew.sh/index_zh-cn.html)

```shell
# 将命令粘贴至终端并回车进行安装
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```



### 关闭SIP

macOS 10.11 后下由于开启了 SIP（System Integrity Protection） 会导致命令行下 proxychains-ng 代理的模式失效，如果你要使用 proxychains-ng 这种简单的方法，就需要先关闭 SIP。

具体的关闭方法如下：

- 部分关闭 SIP

> 重启Mac，按住Option键进入启动盘选择模式，再按⌘ + R进入Recovery模式。
> 实用工具（Utilities）-> 终端（Terminal）。
> 输入命令`csrutil enable --without debug`运行。
> 重启进入系统后，终端里输入 csrutil status，结果中如果有 Debugging Restrictions: disabled 则说明关闭成功。

- 完全关闭 SIP

> 重启Mac，按住Option键进入启动盘选择模式，再按⌘ + R进入Recovery模式。
> 实用工具（Utilities）-> 终端（Terminal）。
> 输入命令`csrutil disable`运行。
> 重启进入系统后，终端里输入 csrutil status，结果中如果有 System Integrity Protection status:disabled. 则说明关闭成功。



### 安装

```shell
brew install proxychains-ng
```



### 配置

使用 Homebrew 安装完成后的配置文件路径为 `/usr/local/etc/proxychains.conf`

打开它，找到 `[ProxyList]`

```shell 
[ProxyList]
socks5  127.0.0.1 1086
```



proxychains-ng支持多种代理模式,默认是选择 strict_chain。

- dynamic_chain ：动态模式,按照代理列表顺序自动选取可用代理
- strict_chain ：严格模式,严格按照代理列表顺序使用代理，所有代理必须可用
- round_robin_chain ：轮询模式，自动跳过不可用代理
- random_chain ：随机模式,随机使用代理



给proxychains4增加一个别名，在 ~/.zshrc或~/.bashrc末尾加入如下行：

```shell
alias pc='proxychains4'
```

这样就可以使用 `pc` 来 指代 `proxychains4`，简化输入。

```shell
pc curl cip.cc
```



如果你使用 `iTerm` 的话可以配置快捷键来实现前缀补全功能

```
在 iTerm -> Preferences -> Profiles -> Keys 中，新建一个快捷键，例如 ⌥ + p ，Action 选择 Send Hex Code，键值为 0x1 0x70 0x63 0x20 0xd，保存生效。
```

更多的Hex Code可以到 [manytricks](https://manytricks.com/keycodes/) 上查找。

使用场景：敲了一长串的命令后想使用代理功能时，就可以直接使用快捷键 `⌥ + p` ，这样就会自动在命令的最前面加上 `pc ` 



### 测试

```shell
proxychains4 curl cip.cc

// 如果你设置了别名的话可以使用 pc 指代 proxychains4
pc curl cip.cc
```

可以看到这就代理上了

```
IP	: xxx.xxx.xxx.xxx
地址	: 美国  加利福尼亚州  洛杉矶
运营商	: it7.net

数据二	: 美国

数据三	: 美国加利福尼亚洛杉矶

URL	: http://www.cip.cc/xxx.xxx.xxx.xxx
```

