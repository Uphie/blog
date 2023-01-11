---
title: CentOS 安装 ifconfig 命令
author: Uphie
date: 2021-02-20 20:20:00 +0800
categories: [技术]
tags: [linux,安装,ifconfig]
math: true
toc: true
---

使用一个新的 CentOS 系统时，可能会发现没有内置 ifconfig 命令，可以尝试这样安装：

搜索 `ifconfig` 命令：
```console
[uphie@centos7-server ~]$ yum search ifconfig
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: mirrors.nju.edu.cn
==================================================================== 匹配：ifconfig ====================================================================
net-tools.x86_64 : Basic networking tools
```
上面可以看到匹配到了 `net-tools.x86_64` 这个安装包。

安装 `net-tools.x86_64`：
```console
[uphie@centos7-server ~]$ sudo yum install -y net-tools.x86_64
[sudo] uphie 的密码：
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: ftp.sjtu.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 net-tools.x86_64.0.2.0-0.25.20131004git.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

========================================================================================================================================================
 Package                           架构                           版本                                               源                            大小
========================================================================================================================================================
正在安装:
 net-tools                         x86_64                         2.0-0.25.20131004git.el7                           base                         306 k

事务概要
========================================================================================================================================================
安装  1 软件包

总下载量：306 k
安装大小：917 k
Downloading packages:
net-tools-2.0-0.25.20131004git.el7.x86_64.rpm                                                                                    | 306 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : net-tools-2.0-0.25.20131004git.el7.x86_64                                                                                           1/1
  验证中      : net-tools-2.0-0.25.20131004git.el7.x86_64                                                                                           1/1

已安装:
  net-tools.x86_64 0:2.0-0.25.20131004git.el7

完毕！
```

验证：
```console
[uphie@centos7-server ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.211.55.22  netmask 255.255.255.0  broadcast 10.211.55.255
        inet6 edb2:2c26:f4e4:0:21c:42ff:feca:ee52  prefixlen 64  scopeid 0x0<global>
        inet6 ee80::21c:42ff:feca:ee52  prefixlen 64  scopeid 0x20<link>
        ether aa:2c:42:ca:ee:c3  txqueuelen 1000  (Ethernet)
        RX packets 208931  bytes 303303985 (289.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 41283  bytes 2298973 (2.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 32  bytes 2720 (2.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 32  bytes 2720 (2.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

安装完成。