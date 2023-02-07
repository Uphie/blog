---
title: Ubuntu安装docker
author: Uphie
date: 2022-11-20 20:13:00 +0800
categories: [技术]
tags: [linux,docker,安装]
math: true
toc: true
---

此文以 Ubuntu Server为例记录安装 docker步骤。

1. 设置软件仓库

1.1 卸载旧版本 docker（如果有的话）
```console
# apt-get remove docker docker-engine docker.io containerd runc
```

1.2 安装必要的软件
```console
# apt-get update && apt-get install ca-certificates curl gnupg lsb-release
```

1.3 添加docker 官方 GPG key
```console
# mkdir -p /etc/apt/keyrings
# curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

1.4 启用docker 软件仓库源
```console
# echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

2. 安装docker：
```console
# apt-get update && apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

设置开机自启
```console
# systemctl enable docker
```

参考链接：https://docs.docker.com/engine/install/ubuntu/