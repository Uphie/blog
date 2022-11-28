---
title: redis中的list
author: Uphie
date: 2020-09-29 22:35:00 +0800
categories: [技术]
tags: [redis]
math: true
toc: true
---

最多存储 $${2^{32}}-1$$ 个元素。

# 内部编码

## ziplist

压缩列表。有以下必须满足的条件：

1. 当元素个数小于 list-max-ziplist_entries（默认512）。
2. 列表中每个元素的值都小于 list-max-ziplist-value（默认64字节）。

ziplist 是一个能维持数据项先后顺序的列表，且内存紧凑，但不适合修改操作，每次数据变动都会引起内存的 realloc，每次 realloc 会导致大量的数据拷贝，降低性能。


## linkedlist

链表。无法满足 ziplist 条件时，会使用改编码实现。

redis链表是一个双向无环链表结构，很多发布订阅、慢查询、监视器功能都是使用到了链表来实现。
每个链表的节点由一个listNode结构来表示，每个节点都有指向前置节点和后置节点的指针，同时表头节点的前置和后置节点都指向NULL。


# 常用命令

## 添加类
| 命令               | 说明                                   | 时间复杂度                              |
| ------------------ | -------------------------------------- | -------------------------------------- |
| lpush key element [element...] | 从左侧添加元素                         | O(k)，k为元素个数 |
| rpush key element [element...] | 从右侧添加元素                         | O(k)，k为元素个数 |
| linsert key before/after pivot element | 找到pivot元素，并在其前面或后面插入元素 | O(n)，n为pivot距离头或尾的距离 |
| lpushx key element [element...] | 向左侧添加元素，只有当key存在时，即只追加不创建 | O(k)，k为元素个数 |
| rpushx key element [element...] | 向右侧添加元素，只有当key存在时，即只追加不创建 | O(k)，k为元素个数 |

## 查询类

| 命令                                     | 说明                                                         | 时间复杂度 |
| ---------------------------------------- | ------------------------------------------------------------ | ---------- |
| lrange key start end                     | 查询指定区间 [start,end] 的元素                              | O(n)       |
| lindex key index                         | 获取指定索引上的元素，index为负数时，从由往左数，如-1表示倒数第一个 | O(n)       |
| llen key                                 | 求长度                                                       | O(1)       |
| lpox key element [rank] [count] [maxlen] | 查找元素 element的索引                                       | O(n)       |

## 删除类

| 命令                         | 说明                                                    | 时间复杂度 |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| lpop key                     | 从左侧弹出一个元素                                    | O(1)                        |
| rpop key                     | 从右侧弹出一个元素                                      | O(1) |
| blpop key [key...] [timeout] | 阻塞者从左侧弹出一个元素，若在 timeout 时间到后返回 nil | O(k)，k为key数列 |
| brpop key [key...] [timeout] | 阻塞者从右侧弹出一个元素，若在 timeout 时间到后返回 nil | O(k)，k为key数列 |
| lrem key count pivot    | 删除值为 pivot的元素，count>0，从左往右最多删除count个元素；count<0，从右往左最多删除count个元素；count=0，删除所有。 | O(n) |
| ltrim key start end |                        | O(n)，n为裁剪的元素总数 |


## 移动类
| 命令                         | 说明                                                    | 时间复杂度                                               |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| rpoplpush source_key destination_key  [timeout] | 从一个list右移除一个元素添加到另一个list的左侧 | O(1) |
| brpoplpush source_key destination_key  [timeout] | 阻塞着从一个list右移除一个元素添加到另一个list的左侧 | O(1) |



## 修改类


| 命令                   | 说明                   | 时间复杂度          |
| ---------------------- | ---------------------- | ------------------- |
| lset key index element | 设定指定位置的元素的值 | O(n)，n为索引偏移量 |

# 常用场景

- 消息队列，list作为队列用
- 数据列表
