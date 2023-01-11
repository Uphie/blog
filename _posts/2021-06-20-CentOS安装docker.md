---
title: CentOS 安装 docker
author: Uphie
date: 2021-06-20 21:20:00 +0800
categories: [技术]
tags: [linux,安装,docker]
math: true
toc: true
---

在 CentOS 中安装 docker，主要有两种方式：
- 设置 docker 软件仓库，从此仓库下载，推荐
- 下载 rpm 包，并手动安装和升级


接下来介绍第一种方法安装 docker。


# 1. 安装 yum-utils

`yum-utils`，用于管理软件仓库和扩展包，如果已安装可忽略：
```console
[uphie@centos7-server ~]$ sudo yum install -y yum-utils
[sudo] uphie 的密码：
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: ftp.sjtu.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 yum-utils.noarch.0.1.1.31-54.el7_8 将被 安装
--> 正在处理依赖关系 python-kitchen，它被软件包 yum-utils-1.1.31-54.el7_8.noarch 需要
--> 正在处理依赖关系 libxml2-python，它被软件包 yum-utils-1.1.31-54.el7_8.noarch 需要
--> 正在检查事务
---> 软件包 libxml2-python.x86_64.0.2.9.1-6.el7_9.6 将被 安装
---> 软件包 python-kitchen.noarch.0.1.1.1-5.el7 将被 安装
--> 正在处理依赖关系 python-chardet，它被软件包 python-kitchen-1.1.1-5.el7.noarch 需要
--> 正在检查事务
---> 软件包 python-chardet.noarch.0.2.2.1-3.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

=====================================================================================
 Package                架构           版本                    源               大小
=====================================================================================
正在安装:
 yum-utils              noarch         1.1.31-54.el7_8         base            122 k
为依赖而安装:
 libxml2-python         x86_64         2.9.1-6.el7_9.6         updates         247 k
 python-chardet         noarch         2.2.1-3.el7             base            227 k
 python-kitchen         noarch         1.1.1-5.el7             base            267 k

事务概要
=====================================================================================
安装  1 软件包 (+3 依赖软件包)

总下载量：863 k
安装大小：4.3 M
Downloading packages:
(1/4): libxml2-python-2.9.1-6.el7_9.6.x86_64.rpm              | 247 kB  00:00:00
(2/4): python-chardet-2.2.1-3.el7.noarch.rpm                  | 227 kB  00:00:00
(3/4): yum-utils-1.1.31-54.el7_8.noarch.rpm                   | 122 kB  00:00:00
(4/4): python-kitchen-1.1.1-5.el7.noarch.rpm                  | 267 kB  00:00:00
-------------------------------------------------------------------------------------
总计                                                    1.9 MB/s | 863 kB  00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : python-chardet-2.2.1-3.el7.noarch                                1/4
  正在安装    : python-kitchen-1.1.1-5.el7.noarch                                2/4
  正在安装    : libxml2-python-2.9.1-6.el7_9.6.x86_64                            3/4
  正在安装    : yum-utils-1.1.31-54.el7_8.noarch                                 4/4
  验证中      : python-kitchen-1.1.1-5.el7.noarch                                1/4
  验证中      : yum-utils-1.1.31-54.el7_8.noarch                                 2/4
  验证中      : libxml2-python-2.9.1-6.el7_9.6.x86_64                            3/4
  验证中      : python-chardet-2.2.1-3.el7.noarch                                4/4

已安装:
  yum-utils.noarch 0:1.1.31-54.el7_8

作为依赖被安装:
  libxml2-python.x86_64 0:2.9.1-6.el7_9.6     python-chardet.noarch 0:2.2.1-3.el7
  python-kitchen.noarch 0:1.1.1-5.el7

完毕！
```

# 2. 增加 docker 仓库

```console
[uphie@centos7-server ~]$ sudo yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
已加载插件：fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

# 3. 安装 docker（社区版）
```console
[uphie@centos7-server ~]$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: ftp.sjtu.edu.cn
docker-ce-stable                                              | 3.5 kB  00:00:00
(1/2): docker-ce-stable/7/x86_64/updateinfo                   |   55 B  00:00:00
(2/2): docker-ce-stable/7/x86_64/primary_db                   |  91 kB  00:00:00
正在解决依赖关系
--> 正在检查事务
---> 软件包 containerd.io.x86_64.0.1.6.15-3.1.el7 将被 安装
--> 正在处理依赖关系 container-selinux >= 2:2.74，它被软件包 containerd.io-1.6.15-3.1.el7.x86_64 需要
---> 软件包 docker-ce.x86_64.3.20.10.22-3.el7 将被 安装
--> 正在处理依赖关系 docker-ce-rootless-extras，它被软件包 3:docker-ce-20.10.22-3.el7.x86_64 需要
---> 软件包 docker-ce-cli.x86_64.1.20.10.22-3.el7 将被 安装
--> 正在处理依赖关系 docker-scan-plugin(x86-64)，它被软件包 1:docker-ce-cli-20.10.22-3.el7.x86_64 需要
---> 软件包 docker-compose-plugin.x86_64.0.2.14.1-3.el7 将被 安装
--> 正在检查事务
---> 软件包 container-selinux.noarch.2.2.119.2-1.911c772.el7_8 将被 安装
---> 软件包 docker-ce-rootless-extras.x86_64.0.20.10.22-3.el7 将被 安装
--> 正在处理依赖关系 fuse-overlayfs >= 0.7，它被软件包 docker-ce-rootless-extras-20.10.22-3.el7.x86_64 需要
--> 正在处理依赖关系 slirp4netns >= 0.4，它被软件包 docker-ce-rootless-extras-20.10.22-3.el7.x86_64 需要
---> 软件包 docker-scan-plugin.x86_64.0.0.23.0-3.el7 将被 安装
--> 正在检查事务
---> 软件包 fuse-overlayfs.x86_64.0.0.7.2-6.el7_8 将被 安装
--> 正在处理依赖关系 libfuse3.so.3(FUSE_3.2)(64bit)，它被软件包 fuse-overlayfs-0.7.2-6.el7_8.x86_64 需要
--> 正在处理依赖关系 libfuse3.so.3(FUSE_3.0)(64bit)，它被软件包 fuse-overlayfs-0.7.2-6.el7_8.x86_64 需要
--> 正在处理依赖关系 libfuse3.so.3()(64bit)，它被软件包 fuse-overlayfs-0.7.2-6.el7_8.x86_64 需要
---> 软件包 slirp4netns.x86_64.0.0.4.3-4.el7_8 将被 安装
--> 正在检查事务
---> 软件包 fuse3-libs.x86_64.0.3.6.1-4.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

=====================================================================================
 Package                   架构   版本                        源                大小
=====================================================================================
正在安装:
 containerd.io             x86_64 1.6.15-3.1.el7              docker-ce-stable  33 M
 docker-ce                 x86_64 3:20.10.22-3.el7            docker-ce-stable  22 M
 docker-ce-cli             x86_64 1:20.10.22-3.el7            docker-ce-stable  30 M
 docker-compose-plugin     x86_64 2.14.1-3.el7                docker-ce-stable  10 M
为依赖而安装:
 container-selinux         noarch 2:2.119.2-1.911c772.el7_8   extras            40 k
 docker-ce-rootless-extras x86_64 20.10.22-3.el7              docker-ce-stable 8.5 M
 docker-scan-plugin        x86_64 0.23.0-3.el7                docker-ce-stable 3.8 M
 fuse-overlayfs            x86_64 0.7.2-6.el7_8               extras            54 k
 fuse3-libs                x86_64 3.6.1-4.el7                 extras            82 k
 slirp4netns               x86_64 0.4.3-4.el7_8               extras            81 k

事务概要
=====================================================================================
安装  4 软件包 (+6 依赖软件包)

总下载量：107 M
安装大小：398 M
Is this ok [y/d/N]: y
Downloading packages:
(1/10): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm  |  40 kB  00:00:00
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-20.10.22-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
docker-ce-20.10.22-3.el7.x86_64.rpm 的公钥尚未安装
(2/10): docker-ce-20.10.22-3.el7.x86_64.rpm                   |  22 MB  00:00:04
(3/10): containerd.io-1.6.15-3.1.el7.x86_64.rpm               |  33 MB  00:00:08
(4/10): docker-ce-rootless-extras-20.10.22-3.el7.x86_64.rpm   | 8.5 MB  00:00:02
(5/10): docker-ce-cli-20.10.22-3.el7.x86_64.rpm               |  30 MB  00:00:06
(6/10): fuse-overlayfs-0.7.2-6.el7_8.x86_64.rpm               |  54 kB  00:00:00
(7/10): slirp4netns-0.4.3-4.el7_8.x86_64.rpm                  |  81 kB  00:00:00
(8/10): docker-scan-plugin-0.23.0-3.el7.x86_64.rpm            | 3.8 MB  00:00:01
(9/10): docker-compose-plugin-2.14.1-3.el7.x86_64.rpm         |  10 MB  00:00:02
(10/10): fuse3-libs-3.6.1-4.el7.x86_64.rpm                    |  82 kB  00:00:05
-------------------------------------------------------------------------------------
总计                                                    6.3 MB/s | 107 MB  00:17
从 https://download.docker.com/linux/centos/gpg 检索密钥
导入 GPG key 0x621E9F35:
 用户ID     : "Docker Release (CE rpm) <docker@docker.com>"
 指纹       : 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 来自       : https://download.docker.com/linux/centos/gpg
是否继续？[y/N]：y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch              1/10
  正在安装    : containerd.io-1.6.15-3.1.el7.x86_64                             2/10
  正在安装    : slirp4netns-0.4.3-4.el7_8.x86_64                                3/10
  正在安装    : fuse3-libs-3.6.1-4.el7.x86_64                                   4/10
  正在安装    : fuse-overlayfs-0.7.2-6.el7_8.x86_64                             5/10
  正在安装    : docker-scan-plugin-0.23.0-3.el7.x86_64                          6/10
  正在安装    : 1:docker-ce-cli-20.10.22-3.el7.x86_64                           7/10
  正在安装    : docker-ce-rootless-extras-20.10.22-3.el7.x86_64                 8/10
  正在安装    : 3:docker-ce-20.10.22-3.el7.x86_64                               9/10
  正在安装    : docker-compose-plugin-2.14.1-3.el7.x86_64                      10/10
  验证中      : 3:docker-ce-20.10.22-3.el7.x86_64                               1/10
  验证中      : docker-scan-plugin-0.23.0-3.el7.x86_64                          2/10
  验证中      : docker-ce-rootless-extras-20.10.22-3.el7.x86_64                 3/10
  验证中      : fuse3-libs-3.6.1-4.el7.x86_64                                   4/10
  验证中      : fuse-overlayfs-0.7.2-6.el7_8.x86_64                             5/10
  验证中      : slirp4netns-0.4.3-4.el7_8.x86_64                                6/10
  验证中      : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch              7/10
  验证中      : 1:docker-ce-cli-20.10.22-3.el7.x86_64                           8/10
  验证中      : containerd.io-1.6.15-3.1.el7.x86_64                             9/10
  验证中      : docker-compose-plugin-2.14.1-3.el7.x86_64                      10/10

已安装:
  containerd.io.x86_64 0:1.6.15-3.1.el7  docker-ce.x86_64 3:20.10.22-3.el7
  docker-ce-cli.x86_64 1:20.10.22-3.el7  docker-compose-plugin.x86_64 0:2.14.1-3.el7

作为依赖被安装:
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8
  docker-ce-rootless-extras.x86_64 0:20.10.22-3.el7
  docker-scan-plugin.x86_64 0:0.23.0-3.el7
  fuse-overlayfs.x86_64 0:0.7.2-6.el7_8
  fuse3-libs.x86_64 0:3.6.1-4.el7
  slirp4netns.x86_64 0:0.4.3-4.el7_8

完毕！
```

# 4. 启动 docker

```console
[uphie@centos7-server ~]$ sudo systemctl start docker
[sudo] uphie 的密码：
```

查看是否可用：
```console
[uphie@centos7-server ~]$ sudo docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

参考链接：[https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)