---
title: MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk.
author: Uphie
date: 2023-06-08 14:03:00 +0800
categories: [技术]
tags: [redis]
math: true
toc: true
---

程序有一天突然出错，和 redis 有关：

> MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.


解决方法：
编辑 `/etc/redis.conf`，找到这个配置：
```
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
```
配置修改为
```
stop-writes-on-bgsave-error no
```

重启 redis
```
$ systemctl restart redis
```