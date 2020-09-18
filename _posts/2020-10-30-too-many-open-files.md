---
title: too many open files; retrying in 5ms
author: Uphie
date: 2020-10-30 09:50:10 +0800
categories: [技术]
tags: [进程,文件描述符,go]
math: true
toc: true
---

在用 `iris` 和 `gin` 做压测，压测结果不怎么理想。发现两个程序都有以下的日志输出：
```
[HTTP Server] http: Accept error: accept tcp [::]:9000: accept4: too many open files; retrying in 5ms
[HTTP Server] http: Accept error: accept tcp [::]:9000: accept4: too many open files; retrying in 10ms
[HTTP Server] http: Accept error: accept tcp [::]:9000: accept4: too many open files; retrying in 5ms
```

看日志是说有 server 进程打开的文件太多，那么进程应该打开了最大限制量的文件描述符，无法再打开新的，在这里即网络连接。

想到压测是用 5000 的并发量测试的，查看系统限制：
```
[root@Uphie ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 62790
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 62790
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

[root@Uphie ~]# cat /proc/18178/limits
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            10485760             unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             62790                62790                processes
Max open files            1024                 4096                 files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       62790                62790                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us
```
可以看到系统限制了 server 进程可打开的文件描述符的数量为 1024（软限制），最大不能超过 4096（硬限制），限制是默认的没被修改过。

而我在别的机器上也做了相同并发的测试，没有出现上面的错误，查看其进程的文件描述符软硬限制已经被改为了102400，完全够了。

从系统层面修改限制
```
[root@Uphie ~]# vim /etc/security/limits.conf
```
在末尾修改：
```
* soft nofile 102400
* hard nofile 102400
```
重新登录，启动程序测试，问题解决。
