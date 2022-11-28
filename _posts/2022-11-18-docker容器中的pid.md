---
title: docker容器中的PID
author: Uphie
date: 2022-11-18 21:30:20 +0800
categories: [技术]
tags: [linux,docker,namespace,进程]
math: true
toc: true
---
如果有意观察的话，会发现 docker容器内初始进程的进程号 PID 很小，似乎就是一个独立的机器，是如何实现的呢？

以 mysql8 容器为例，我们先在容器中先安装下 ps 命令：
```console
$ docker exec -it mysql8.0.21 bash
root@8b28ee20e307:/# apt update
root@8b28ee20e307:/# apt install procps
```

查看下当前进程：
```console
root@8b28ee20e307:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
mysql        1     0  0 06:44 pts/0    00:04:53 mysqld
root        85     0  0 06:44 pts/1    00:00:00 /bin/sh
root       532     0  0 16:28 pts/2    00:00:00 bash
root       539   532  0 16:28 pts/2    00:00:00 ps -ef
```

我们发现进程 `mysqld` 的 PID 为1，最下面的两个进程其实是刚刚的 `ps -ef` 进程和其父进程 `bash`。进程号都很小，看上去这些进程是运行在一个和宿主机完全独立的机器中。

我们知道 Linux 中进程启动都会被分配到一个进程号，即 PID，用于唯一标识这个进程，进程退出 PID 就会被回收。 但实际上上面的情况是个“障眼法”，容器中的进程在宿主机操作系统中分配的 PID 与容器内的 PID 不同，容器内 PID 为1，在宿主机中可能就是 500。实现这个“障眼法”的就是 Linux 的 namespace 机制。

Linux 中创建进程/线程的一个系统调用是：
```C++
int pid=clone(main_function,stack_size,SIGCHLD,NULL);
```
使用系统调用时如果传入 `CLONE_NEWPID` 参数，即：
```C++
int pid=clone(main_function,stack_size,CLONE_NEWPID|SIGCHLD,NULL)
```
新创建的进程会得到一个全新的进程空间，在这个进程空间里新进程的 PID 是1。是不是猜到什么了？在宿主机的进程空间中，PID 还是原来的值，不是1。

如果多次执行这个函数的话，得到的进程在其进程空间内看不到其他进程空间的进程，都“认为”自己的 PID 是1。

这就是 Linux 容器最基本的实现原理，容器只是一种特殊的进程。

那在宿主机中，进程的 PID 是什么呢？可以这样查看：
```console
$ docker top mysql8.0.21
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
999                 7774                7749                0                   06:44               ?                   00:04:48            mysqld
root                8045                7749                0                   06:44               ?                   00:00:00            /bin/sh
```
可以看到 1 号进程的 `mysqld` 的真实 PID 是7774。

类似的，Linux 还提供了 `Mount`、`Network`、`UTS`、`IPC`、`User` 这些 namespace，用于互相隔离环境，例如 Mount Namespace 能让被隔离的进程只能看到当前 namespace 下的磁盘挂载信息。

看上去 Docker 使用的 Linux Namespace 非常棒，能隔离进程、文件、网络、用户等等，但仍有缺点。由于容器内进程在宿主机上都是普通的宿主机进程只是被 namespace 隔离了而已，那么这些进程就要共用相同的操作系统内核，那么低版本内核的宿主机不能运行高版本内核的容器。理论上Windows、Mac OS 上也不能运行 Linux 的容器，因为没有 Linux 的宿主环境。为了解决这个问题，Windows 和 mac OS上的 Docker Desktop 软件使用了不同的虚拟化技术启动了Linux虚拟机，Linux 容器实际上是跑在了 Linux 虚拟机上，有兴趣的可以参阅 http://t.zoukankan.com/jing1617-p-9474464.html 。

另外一个 Linux Namespace 无法隔离的是时间，如果一个容器修改了系统时间，那么宿主机时间和其他容器时间也会被改变。