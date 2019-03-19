---
title: 科学上网解决方案
date: 2019-02-16 13:41:09
tags:
  - 科学上网
toc: true
---

## shadowsocks 服务端配置

### 一键安装脚本

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

若已安装多个版本，则卸载时也需多次运行（每次卸载一种）
使用 root 用户登录，运行以下命令：

```bash
$ ./shadowsocks-all.sh uninstall
```

### 命令

```bash
$ /etc/init.d/shadowsocks-r start | stop | restart | status
```

## 开启 bbr 加速

众所周知，Ubuntu 开启 BBR 的前提是内核必须等于高于 4.9，所以想要使用这个牛逼的玩意儿，需要先看看你的内核是否是 4.9 或者以上。

查看命令：`uname -a`

如果是 4.9 或者以上，那么恭喜你，升级内核这一步你就可以跳过了，如果在 4.9 以下，那就需要更新一下内核了；很遗憾 GCE 官方默认搭载的镜像，内核是 4.4 的，所以我必须要做一波内核升级了。

### Ubuntu 内核升级

升级过程中其实比较简单，先确定你的系统是 32 位还是 64 位的，可以用下面的命令查看

查看命令：`getconf LONG_BIT`

确定系统之后，需要下载必要的升级程序包

http://kernel.ubuntu.com/~kernel-ppa/mainline/

这个网站可以找到最新的程序包，根据自己的需要使用 wget 命令来下载到服务器；

比如我的服务器是 64 位，安装 4.10.2 的内核：

```bash
sudo wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10.2/linux-image-4.10.2-041002-generic_4.10.2-041002.201703120131_amd64.deb
```

然后切换到你的文件下载目录，执行下列命令来升级：

```bash
$ sudo dpkg -i linux-image-4.10.2-041002-generic_4.10.2-041002.201703120131_amd64.deb
```

最后，执行命令 sudo update-grub，更新 grub 引导装入程序。

一旦各方面都已完成，重启机器，你就可以准备使用了。系统重启后，打开终端窗口，执行命令 uname -a，确保你实际上是在运行你更新之后的内核。

### 开启 TCP BBR

修改系统变量：

```bash
$ echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
$ echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

重点，执行以上命令，如果显示拒绝访问可以尝试使用如下命令

```bash
$ sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
$ sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
```

保存生效
sysctl -p

执行

```bash
$ sysctl net.ipv4.tcp_available_congestion_control
```

如果返回结果
net.ipv4.tcp_available_congestion_control = bbr cubic reno
那么恭喜你 BBR 开启成功了！

也可以执行

```bash
$ lsmod | grep bbr
```

来检测 BBR 是否真的开启成功

## 终端代理

以`zsh`作为说明

```shell
➜  ~ vim ~/.zshrc
```

添加如下代理配置:

1086 是 shadowsocksx-ng 默认端口

```shell
# proxy list
alias proxy='export all_proxy=socks5://127.0.0.1:1086'
alias unproxy='unset all_proxy'
```

`:wq`保存退出

```shell
➜  ~ source ~/.zshrc
```

使用`proxy`前先查看下当前的`ip`地址：

```shell
➜  ~ curl ip.cn
当前 IP：112.64.xxx.xx 来自：上海市 联通
```

或者

```shell
~ curl cip.cc
IP	: 140.206.97.42
地址	: 中国  上海

数据二	: 上海市 | 联通

URL	: http://www.cip.cc/140.206.97.42
```

执行:

```shell
➜  ~ proxy
➜  ~ curl ip.cn
当前 IP：47.89.xx.xxx 来自：香港特别行政区 阿里云
```

如果 ip.cn 不能用，可以换个类似的站点查询

```shell
~ curl cip.cc
IP	: 45.78.47.19
地址	: 美国  加利福尼亚

数据二	: 美国 | 加利福尼亚州洛杉矶市 IT7 Networks

URL	: http://www.cip.cc/45.78.47.19
```

没问题，终端走了代理，`brew update`顺畅了- -

如果不需要走代理，执行：

```shell
➜  ~ unproxy
➜  ~ curl ip.cn
当前 IP：112.64.xxx.xx 来自：上海市 联通
```
