---
title: CROSSSLOT Keys in request don't hash to the same slot
author: Uphie
date: 2020-12-29 18:52:00 +0800
categories: [技术]
tags: [redis,hash-slot,hash-tag]
math: true
toc: true
---

# 问题

跨 slot 的 key 没有被散列到相同的 slot 中。

```shell
127.0.0.1:8000>setbit a 3 1
-> Redirected to slot[5084] located at 10.11.0.1:8000
(integer) 0
127.0.0.1:8000>setbit b 2 1
-> Redirected to slot[2703] located at 10.11.0.2:8000
(integer) 0
127.0.0.1:8000>bitop and c a b
(error) CROSSSLOT Keys in request don't hash to the same slot
```

# 解决

## hash slot
首先要先说明下上面涉及到的一个概念 `hash slot`，redis 集群中的 key 空间被划分为 16384 个 hash slot，节点数量也被限制为最多16384个（建议1000以内）。每个节点负责一部分的 hash slot，key 划分到 hash slot的规则如下：

```
HASH_SLOT = CRC16(key) mod 16384
```

hash slot 可以帮助集群的横向伸缩，增减节点时只需要将节点上的 slot 转移到其他节点。

**在集群模式中，同一个请求中涉及多个key的操作，会被限制在一个 slot 中，不能跨 slot 执行。**
比如：
- 多 key 命令： `mset`、`mget`、`bitop` 等
- 涉及到多 key 的**事务**
- 涉及到多 key 的**Lua脚本**

上面的情形中 `a` 和 `b` 就被映射到了不同的 slot 上面。

但是** Redis Enterprise 没有这个问题，只是开源版有这个问题**，不知道以后开源版会不会修改这个bug。

该异常参见源码：[https://github.com/redis/redis/blob/5.0/src/cluster.c](https://github.com/redis/redis/blob/5.0/src/cluster.c)
```C
/* If it is not the first key, make sure it is exactly
 * the same key as the first we saw. */
if (!equalStringObjects(firstkey,thiskey)) {
    if (slot != thisslot) {
        /* Error: multiple keys from different slots. */
        getKeysFreeResult(keyindex);
        if (error_code)
            *error_code = CLUSTER_REDIR_CROSS_SLOT;
        return NULL;
    } else {
        /* Flag this request as one with multiple different
         * keys. */
        multiple_keys = 1;
    }
}
```

## hash tag
hash tag 可以影响 hash slot 的生成，相同 hash tag 的 key 会被分配到相同的 hash slot。hash tag使用 `{...}` 形式。对于包含 hash tag 的 key，redis只会对 `{}` 内的字符串计算 hash，从而相同 hash tag 的 key 会计算得到相同的 hash slot。

一个有效的 hash tag 应该是key中首个 `{` 和首个 `}`（在首个 `{` 之后） 之间有字符存在。

示例：
- `{user1000}.following`、 `{user1000}.follower` 有相同 hash tag，会被分到相同的 slot
- `foo{}{bar}` 没有有效的 hash tag，会按照 `foo{}{bar}` 进行 hash 计算
- `foo{{bar}}zap` 有有效的 hash tag，会按照 `{bar` 进行 hash 计算
- `foo{bar}{zap}` 有有效的 hash tag，会按照 `bar` 进行 hash 计算


注意，hash tag 要合理使用，避免大量的 key 被分配到相同 slot 里导致数据存储和访问倾斜。

## 解决方法

根据前面的讲到的 hash tag 可以这样解决：
```shell
127.0.0.1:8000>setbit {sometext}a 3 1
(integer) 0
127.0.0.1:8000>setbit {sometext}b 2 1
(integer) 0
127.0.0.1:8000>bitop and c {sometext}a {sometext}b
1
```

参考资料：

[https://redis.io/topics/cluster-spec](https://redis.io/topics/cluster-spec)

[https://redislabs.com/blog/redis-clustering-best-practices-with-keys/](https://redislabs.com/blog/redis-clustering-best-practices-with-keys/)
