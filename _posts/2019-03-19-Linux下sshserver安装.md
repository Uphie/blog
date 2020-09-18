---
title: Linux下sshserver安装
author: Uphie
date: 2019-03-19 00:00:00 +0800
categories: [技术]
tags: [linux,安装]
math: true
toc: true
---

以下以 Ubuntu 系统为例
```shell
# 系统更新
$ sudo apt-get update

# 安装
$ sudo apt-get install -y openssh-server

# 修改配置...（可选）
$ sudo vim /etc/ssh/ssh_config

# 末尾添加以下内容，以支持Windows下SecureShell能够远程（可选）
Ciphers aes128-cbc,aes192-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,3des-cbc,arcfour128,arcfour256,arcfour,blowfish-cbc,cast128-cbc

MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160,hmac-sha1-96,hmac-md5-96

KexAlgorithms diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group-exchange-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group1-sha1,curve25519-sha256@libssh.org

# 重启sshserver
$ service sshd restart
```
