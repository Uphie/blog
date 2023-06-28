---
title: error while loading shared libraries libcrypto.so.1.1
author: Uphie
date: 2023-06-28 21:03:00 +0800
categories: [技术]
tags: [linux,mongo,openssl]
math: true
toc: true
---

在 Ubuntu 22.04.2 LTS 上，mongodb 5.0.17 启动时报错，
```
root@node1:/home/uphie# systemctl start mongo
Job for mongo.service failed because the control process exited with error code.
See "systemctl status mongo.service" and "journalctl -xeu mongo.service" for details.
```

按照提示查看错误信息：
```
root@node1:/home/uphie# journalctl -xeu mongo.service
```

从输出中发现几处错误：
```
Jun 28 10:38:37 node1 mongod[2026]: /opt/mongodb/bin/mongod: error while loading shared libraries: libcrypto.so.1.1: cannot open shared o>Jun 28 10:38:37 node1 systemd[1]: mongo.service: Control process exited, code=exited, status=127/n/a
```

依赖库缺失，查看下 mongod 依赖情况：
```
root@node1:/home/uphie# ldd /opt/mongodb/bin/mongod
        linux-vdso.so.1 (0x00007ffc1bff3000)
        libcurl.so.4 => /lib/x86_64-linux-gnu/libcurl.so.4 (0x00007fc06c21b000)
        liblzma.so.5 => /lib/x86_64-linux-gnu/liblzma.so.5 (0x00007fc06c1f0000)
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007fc06c1dc000)
        libcrypto.so.1.1 => not found
        libssl.so.1.1 => not found
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fc06c1d5000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fc06c1d0000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fc06c0e9000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fc06c0c9000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fc06c0c4000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc06be9c000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc071a8f000)
        libnghttp2.so.14 => /lib/x86_64-linux-gnu/libnghttp2.so.14 (0x00007fc06be70000)
        libidn2.so.0 => /lib/x86_64-linux-gnu/libidn2.so.0 (0x00007fc06be4f000)
        librtmp.so.1 => /lib/x86_64-linux-gnu/librtmp.so.1 (0x00007fc06be30000)
        libssh.so.4 => /lib/x86_64-linux-gnu/libssh.so.4 (0x00007fc06bdc3000)
        libpsl.so.5 => /lib/x86_64-linux-gnu/libpsl.so.5 (0x00007fc06bdaf000)
        libssl.so.3 => /lib/x86_64-linux-gnu/libssl.so.3 (0x00007fc06bd0b000)
        libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x00007fc06b8c7000)
        libgssapi_krb5.so.2 => /lib/x86_64-linux-gnu/libgssapi_krb5.so.2 (0x00007fc06b873000)
        libldap-2.5.so.0 => /lib/x86_64-linux-gnu/libldap-2.5.so.0 (0x00007fc06b814000)
        liblber-2.5.so.0 => /lib/x86_64-linux-gnu/liblber-2.5.so.0 (0x00007fc06b803000)
        libzstd.so.1 => /lib/x86_64-linux-gnu/libzstd.so.1 (0x00007fc06b734000)
        libbrotlidec.so.1 => /lib/x86_64-linux-gnu/libbrotlidec.so.1 (0x00007fc06b726000)
        libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007fc06b708000)
        libunistring.so.2 => /lib/x86_64-linux-gnu/libunistring.so.2 (0x00007fc06b55e000)
        libgnutls.so.30 => /lib/x86_64-linux-gnu/libgnutls.so.30 (0x00007fc06b373000)
        libhogweed.so.6 => /lib/x86_64-linux-gnu/libhogweed.so.6 (0x00007fc06b32b000)
        libnettle.so.8 => /lib/x86_64-linux-gnu/libnettle.so.8 (0x00007fc06b2e5000)
        libgmp.so.10 => /lib/x86_64-linux-gnu/libgmp.so.10 (0x00007fc06b261000)
        libkrb5.so.3 => /lib/x86_64-linux-gnu/libkrb5.so.3 (0x00007fc06b196000)
        libk5crypto.so.3 => /lib/x86_64-linux-gnu/libk5crypto.so.3 (0x00007fc06b167000)
        libcom_err.so.2 => /lib/x86_64-linux-gnu/libcom_err.so.2 (0x00007fc06b161000)
        libkrb5support.so.0 => /lib/x86_64-linux-gnu/libkrb5support.so.0 (0x00007fc06b153000)
        libsasl2.so.2 => /lib/x86_64-linux-gnu/libsasl2.so.2 (0x00007fc06b136000)
        libbrotlicommon.so.1 => /lib/x86_64-linux-gnu/libbrotlicommon.so.1 (0x00007fc06b113000)
        libp11-kit.so.0 => /lib/x86_64-linux-gnu/libp11-kit.so.0 (0x00007fc06afd8000)
        libtasn1.so.6 => /lib/x86_64-linux-gnu/libtasn1.so.6 (0x00007fc06afc0000)
        libkeyutils.so.1 => /lib/x86_64-linux-gnu/libkeyutils.so.1 (0x00007fc06afb9000)
        libffi.so.8 => /lib/x86_64-linux-gnu/libffi.so.8 (0x00007fc06afaa000)
```

可以看到缺少动态链接库 `libcrypto.so.1.1` 和 `libssl.so.1.1`，查阅资料得知是 Ubuntu 22 上 openssl 版本太心，没有这两个库。这样我们再编译下 openssl 1.1.1 拿到动态链接库就好了。

安装编译工具：
```
root@node1:/home/uphie# apt-get install -y gcc make
```

安装 openssl：
```
root@node1:/home/uphie# wget https://www.openssl.org/source/openssl-1.1.1u.tar.gz
root@node1:/home/uphie# tar -zxvf openssl-1.1.1u.tar.gz
root@node1:/home/uphie# cd openssl-1.1.1u
root@node1:/home/uphie/openssl-1.1.1u# ./config
root@node1:/home/uphie/openssl-1.1.1u# make
```

不需要 `make install`，以免覆盖安装造成其他问题。

此时当前目录下已经编译出了 `libcrypto.so.1.1` 和 `libssl.so.1.1`，拷贝到 `/lib/x86_64-linux-gnu`:
```
root@node1:/home/uphie/openssl-1.1.1u# cp libssl.so.1.1 /lib/x86_64-linux-gnu/
root@node1:/home/uphie/openssl-1.1.1u# cp libcrypto.so.1.1 /lib/x86_64-linux-gnu/
```

再检查 mongod 依赖，没有缺失的了：
```
root@node1:/home/uphie# ldd /opt/mongodb/bin/mongod
```

这时再重启mongo，顺利启动！
```
root@node1:/home/uphie# systemctl restart mongo
root@node1:/home/uphie# systemctl status mongo
● mongo.service - mongodb service
     Loaded: loaded (/etc/systemd/system/mongo.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-06-28 13:44:04 UTC; 5s ago
       Docs: https://docs.mongodb.com/manual/
    Process: 24011 ExecStart=/opt/mongodb/bin/mongod -f /opt/mongodb/conf/mongo.yaml (code=exited, status=0/SUCCESS)
   Main PID: 24014 (mongod)
      Tasks: 48 (limit: 9347)
     Memory: 69.4M
        CPU: 254ms
     CGroup: /system.slice/mongo.service
             └─24014 /opt/mongodb/bin/mongod -f /opt/mongodb/conf/mongo.yaml

Jun 28 13:44:04 node1 systemd[1]: Starting mongodb service...
Jun 28 13:44:04 node1 mongod[24011]: about to fork child process, waiting until server is ready for connections.
Jun 28 13:44:04 node1 mongod[24014]: forked process: 24014
Jun 28 13:44:04 node1 mongod[24011]: child process started successfully, parent exiting
Jun 28 13:44:04 node1 systemd[1]: Started mongodb service.
```