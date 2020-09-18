---
title: redis中的string
author: Uphie
date: 2020-09-28 08:05:00 +0800
categories: [技术]
tags: [redis]
math: true
toc: true
---


# 内部编码

## int

8个字节的长整型，即64位。

当值小于 `OBJ_SHARED_INTEGERS` （在 server.h 中定义的大小为10000）时，从共享的整数字符串取值，并对共享字符串的引用+1。这个方式类似于 Python 对“小”整型的处理，对实际使用较频繁的小整型提前创建好以供直接使用。

当值处于“大”整型范围，即 LONG_MIN ~ LONG_MAX 时，创建保存。

当值超过“大”整型范围，则尝试用 embstr 编码，若 embstr 不行，则继续换用 raw 编码。

> https://github.com/redis/redis/blob/5.0/src/object.c

```C
/* Create a string object from a long long value. When possible returns a
 * shared integer object, or at least an integer encoded one.
 *
 * If valueobj is non zero, the function avoids returning a a shared
 * integer, because the object is going to be used as value in the Redis key
 * space (for instance when the INCR command is used), so we want LFU/LRU
 * values specific for each key. */
robj *createStringObjectFromLongLongWithOptions(long long value, int valueobj) {
    robj *o;

    if (server.maxmemory == 0 ||
        !(server.maxmemory_policy & MAXMEMORY_FLAG_NO_SHARED_INTEGERS))
    {
        /* If the maxmemory policy permits, we can still return shared integers
         * even if valueobj is true. */
        valueobj = 0;
    }

    if (value >= 0 && value < OBJ_SHARED_INTEGERS && valueobj == 0) {
        incrRefCount(shared.integers[value]);
        o = shared.integers[value];
    } else {
        if (value >= LONG_MIN && value <= LONG_MAX) {
            o = createObject(OBJ_STRING, NULL);
            o->encoding = OBJ_ENCODING_INT;
            o->ptr = (void*)((long)value);
        } else {
            o = createObject(OBJ_STRING,sdsfromlonglong(value));
        }
    }
    return o;
}
```

示例：
```shell
127.0.0.1:6379> set demo 11
OK
127.0.0.1:6379> object encoding demo
"int"
```

如果设置的值不是合法的长整数的话，将不会使用 int 编码。

```shell
# 不能以0开头
127.0.0.1:6379> set demo 011
OK
127.0.0.1:6379> object encoding demo
"embstr"

# 不能是整数
127.0.0.1:6379> set demo 123.5
OK
127.0.0.1:6379> object encoding demo
"embstr"

# 超出范围
127.0.0.1:6379> set demo 12345678901234567890
OK
127.0.0.1:6379> object encoding demo
"embstr"

127.0.0.1:6379> set demo 12345678901234567890123456789012345678901234567890
OK
127.0.0.1:6379> strlen demo
(integer) 50
127.0.0.1:6379> object encoding demo
"raw"
```


## embstr

该编码是为保存短字符串而优化设计的编码方式，内部使用 sds （简单动态字符串）结构来存储短字符串。

它可以存储长度小于等于 `OBJ_ENCODING_EMBSTR_SIZE_LIMIT`个字节的字符串，不可改变。

> https://github.com/redis/redis/blob/5.0/src/object.c

```C
/* Create a string object with EMBSTR encoding if it is smaller than
 * OBJ_ENCODING_EMBSTR_SIZE_LIMIT, otherwise the RAW encoding is
 * used.
 *
 * The current limit of 44 is chosen so that the biggest string object
 * we allocate as EMBSTR will still fit into the 64 byte arena of jemalloc. */
#define OBJ_ENCODING_EMBSTR_SIZE_LIMIT 44
robj *createStringObject(const char *ptr, size_t len) {
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr,len);
    else
        return createRawStringObject(ptr,len);
}

/* Create a string object with encoding OBJ_ENCODING_EMBSTR, that is
 * an object where the sds string is actually an unmodifiable string
 * allocated in the same chunk as the object itself. */
robj *createEmbeddedStringObject(const char *ptr, size_t len) {
    robj *o = zmalloc(sizeof(robj)+sizeof(struct sdshdr8)+len+1);
    struct sdshdr8 *sh = (void*)(o+1);

    o->type = OBJ_STRING;
    o->encoding = OBJ_ENCODING_EMBSTR;
    o->ptr = sh+1;
    o->refcount = 1;
    if (server.maxmemory_policy & MAXMEMORY_FLAG_LFU) {
        o->lru = (LFUGetTimeInMinutes()<<8) | LFU_INIT_VAL;
    } else {
        o->lru = LRU_CLOCK();
    }

    sh->len = len;
    sh->alloc = len;
    sh->flags = SDS_TYPE_8;
    if (ptr == SDS_NOINIT)
        sh->buf[len] = '\0';
    else if (ptr) {
        memcpy(sh->buf,ptr,len);
        sh->buf[len] = '\0';
    } else {
        memset(sh->buf,0,len+1);
    }
    return o;
}
```
`OBJ_ENCODING_EMBSTR_SIZE_LIMIT` 在不同Redis版本中有差异，3.2之前为39。

使用 `set` 命令创建或更新时，若长度不超过 `OBJ_ENCODING_EMBSTR_SIZE_LIMIT` ，则其编码为 embstr。
```shell
127.0.0.1:6379> set demo snh48
OK
127.0.0.1:6379> object encoding demo
"embstr"
```

这种编码方式的初始化使用一次内存分配函数获得一个连续的空间，减少了内存碎片。

## raw

大于 **OBJ_ENCODING_EMBSTR_SIZE_LIMIT** 个字节且不超过512M的字符串。

使用 `set` 命令创建或更新时，若长度超过 `OBJ_ENCODING_EMBSTR_SIZE_LIMIT` ，则其编码为 raw。
```shell
127.0.0.1:6379> set demo abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
OK
127.0.0.1:6379> strlen demo
(integer) 52
127.0.0.1:6379> object encoding demo
"raw"
```

但是当 embstr 编码的值**修改**时，会转换编码为 raw，即使长度很短。

```shell
127.0.0.1:6379> set demo Uphie
OK
127.0.0.1:6379> object encoding demo
"embstr"
127.0.0.1:6379> append demo Studio
(integer) 11
127.0.0.1:6379> get demo
"UphieStudio"
127.0.0.1:6379> object encoding demo
"raw"
```



# 常见操作

## 字符串类操作

| 命令                      | 说明                                 | 时间复杂度          |
| ------------------------- | ------------------------------------ | ------------------- |
| append key value          | 追加字符串                           | O(1)                |
| strlen key                | 查询字符串长度                       | O(1)                |
| setrange key offset value | 对字符串指定位置设定新值             | O(1)                |
| getrange key start end    | 获取字符串指定区间的值，区间左必右闭 | O(k)，k为字符串长度 |




## 数值类操作

| 命令     | 说明 | 时间复杂度 |
| -------- | ---- | ---------- |
| incr key                         | 自增1 | O(1) |
| decr key                         | 自减1 | O(1) |
| incrby key increment             | 自增指定值 | O(1) |
| decrby key increment             | 自减指定值 | O(1) |
| incrbyfloat key increment | 自增指定浮点数 | O(1) |




## 通用类

| 命令                             | 说明                         | 时间复杂度        |
| -------------------------------- | ---------------------------- | ----------------- |
| get key                          | 取值                         | O(1)              |
| set key value                    | 设置值                       | O(1)              |
| et key value nx                  | 若键不存在，设置值，即只新建 | O(1)              |
| set key value xx                 | 若键存在，设置值，即只更新   | O(1)              |
| getset key value                 | 设置新值，返回旧值           | O(1)              |
| setnx key value                  | 若键不存在，设置值，即只新建 | O(1)              |
| del {key} [key2,key3]            | 删除，可批量                 | O(k)，key为键个数 |
| mset key value [key2 value2 ...] | 批量设置值                   | O(k)，key为键个数 |
| mget key [key2,key3...]          | 批量取值                     | O(k)，key为键个数 |
