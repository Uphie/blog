---
title: MySQL 执行计划中的 type 详解
author: Uphie
date: 2022-12-03 20:20:20 +0800
categories: [技术]
tags: [mysql]
math: true
toc: true
---


在开发过程中，我们尝尝使用 `explain` 来查看 sql 语句的性能情况，如：
```console
mysql> explain select * from test where f1=2;
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key       | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | test  | NULL       | ref  | idx_f1_f2     | idx_f1_f2 | 4       | const |    3 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
1 row in set (0.00 sec)
```

我们以上面示例中的 `type` 为角度，了解下 MySQL 的执行计划设计。

在此之前我们先预定义下测试数据库表，读者阅读时可往前翻阅：

```sql
CREATE TABLE `sku` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '商品名',
  `cat_1` int NOT NULL COMMENT '一级分类',
  `cat_2` int NOT NULL COMMENT '二级分类',
  `cat_3` int NOT NULL COMMENT '三级分类',
  `alias` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '别名',
  `remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '备注',
  `price` decimal(10,2) NOT NULL COMMENT '单价',
  `supplier_id` int NOT NULL COMMENT '供应商 ID',
  `producer_id` int DEFAULT NULL COMMENT '生产厂家 ID',
  `batch_num` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '生产批次号',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `idx_uniq_name` (`name`),
  UNIQUE KEY `idx_batch_num` (`producer_id`,`batch_num`),
  KEY `idx_price` (`price`),
  KEY `idx_alias` (`alias`),
  KEY `idx_cat` (`cat_1`,`cat_2`,`cat_3`),
  FULLTEXT KEY `idx_remark` (`remark`)
) ENGINE=InnoDB AUTO_INCREMENT=8491 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

// sku 表数据，略

CREATE TABLE `sku_stock` (
  `id` int NOT NULL COMMENT 'sku id',
  `stock` decimal(10,2) NOT NULL COMMENT '库存',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

// sku_stock 表数据，略
```

本文以MySQL8.0.21进行讲解。


# 常见类型
## ALL

type 为 ALL 发生在查询在不使用索引的情况下进行了全表扫描。

```console
mysql> explain select * from sku;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 8465 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set (0.00 sec)
mysql> explain select * from sku where supplier_id=10;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 8465 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
```

## system

官方表述：
> The table has only one row (= system table). This is a special case of the const join type.

表中只有一行数据（等于系统表），是 `const` 类型的一种特例。

笔者曾用这样的结构与数据尝试
```
CREATE TABLE `manager` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL COMMENT '管理员名称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO `manager` (`id`, `name`) VALUES (1, '张三');
```

但结果始终不是 system：
```sql
mysql> explain select * from manager where id=1;
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | manager | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set (0.01 sec)
mysql> explain select * from manager where name='张三';
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | manager | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
```

后面发现在 InnoDB 引擎下，**InnoDB 不能可靠地维护表大小**，因此查询优化器不能确定表正好有 1 行，但是在 MyISAM 引擎下可以。

改成用 MyISAM 引擎试下：
```sql
CREATE TABLE `manager` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL COMMENT '管理员名称',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO `manager` (`id`, `name`) VALUES (1, '张三');
```
explain 验证：
```sql
mysql> explain select * from manager where id=1;
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table   | partitions | type   | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | manager | NULL       | system | PRIMARY       | NULL | NULL    | NULL |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
1 row in set (0.00 sec)
mysql> explain select * from manager where name='张三';
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table   | partitions | type   | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | manager | NULL       | system | NULL          | NULL | NULL    | NULL |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+--------+---------------+------+---------+------+------+----------+-------+
1 row in set (0.01 sec)
```

## index

`index`，表示查询直接扫描了**整个**索引，这时候没有回表，即使用了覆盖索引。

如：
```console
mysql> explain select name from sku;
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key           | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | index | NULL          | idx_uniq_name | 1022    | NULL | 8393 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
```

当没有使用覆盖索引时，type 不能为 `index`：
```sql
mysql> explain select name,create_time from sku;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 8393 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set (0.00 sec)
```

当覆盖索引且查询条件有索引时，就不是当前要表示的情况了，它是扫描了部分索引，如
```sql
mysql> explain select name from sku where name='小白菜';
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key           | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | const | idx_uniq_name | idx_uniq_name | 1022    | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------------+
1 row in set (0.00 sec)
```

## index_merge

`index_merge` 指的是查询同时使用了**多个索引**，如 `or` 两端是两个索引的查询

```sql
mysql> explain select * from sku where name='小白菜' or alias='小白菜';
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
| id | select_type | table | partitions | type        | possible_keys           | key                     | key_len   | ref  | rows | filtered | Extra                                             |
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
|  1 | SIMPLE      | sku   | NULL       | index_merge | idx_uniq_name,idx_alias | idx_uniq_name,idx_alias | 1022,1023 | NULL |    2 |   100.00 | Using union(idx_uniq_name,idx_alias); Using where |
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
1 row in set (0.02 sec)
```


## const

`const` 发生在使用唯一性索引进行查询时，如主键、唯一索引

主键：
```sql
mysql> explain select * from sku where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set (0.00 sec)
```

单键唯一索引：
```sql
mysql> explain select * from sku where name='小白菜';
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key           | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | const | idx_uniq_name | idx_uniq_name | 1022    | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------+------+----------+-------+
1 row in set (0.00 sec)
```

复合唯一索引：
```sql
mysql> explain select price from sku where producer_id=1 and batch_num='202201010021';
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key           | key_len | ref         | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | const | idx_batch_num | idx_batch_num | 1028    | const,const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------------+---------+-------------+------+----------+-------+
1 row in set (0.00 sec)
```

## ref

`ref` 指的是非唯一索引扫描，返回匹配的所有行，发生在非唯一索引或唯一索引的非唯一的前缀匹配。

非唯一索引匹配：
```sql
mysql> explain select * from sku where price=3;
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key       | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | ref  | idx_price     | idx_price | 5       | const |   14 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+-----------+---------+-------+------+----------+-------+
1 row in set (0.01 sec)
```

唯一索引的非唯一前缀的匹配：
```sql
mysql> explain select price from sku where producer_id=1;
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key           | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | sku   | NULL       | ref  | idx_batch_num | idx_batch_num | 5       | const |  198 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
1 row in set (0.01 sec)
```

## ref_or_null

`ref_or_null` 类似于 ref，在 `ref` 发生条件基础上还要求包含为 null 的情况

具体来说是下面这样的情况：
```sql
mysql> explain select * from sku where alias='小白菜' or alias is null;
+----+-------------+-------+------------+-------------+---------------+-----------+---------+-------+------+----------+-----------------------+
| id | select_type | table | partitions | type        | possible_keys | key       | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------------+---------------+-----------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | sku   | NULL       | ref_or_null | idx_alias     | idx_alias | 1023    | const |    4 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------------+---------------+-----------+---------+-------+------+----------+-----------------------+
1 row in set (0.01 sec)
```

## eq_ref

`eq_ref` 发生在被驱动表使用唯一性索引进行联表查询

```sql
mysql> explain select * from sku inner join sku_stock on sku.id=sku_stock.id;
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------+------+----------+-------+
| id | select_type | table     | partitions | type   | possible_keys | key     | key_len | ref               | rows | filtered | Extra |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------+------+----------+-------+
|  1 | SIMPLE      | sku_stock | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL              |    2 |   100.00 | NULL  |
|  1 | SIMPLE      | sku       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.sku_stock.id |    1 |   100.00 | NULL  |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------+------+----------+-------+
2 rows in set (0.00 sec)
```

上面示例中，sku 是被驱动表，sku_stock 是驱动表。

## fulltext

`fulltext` 发生在查询使用了全文索引

```sql
mysql> explain select * from sku where match(remark) against('白菜');
+----+-------------+-------+------------+----------+---------------+------------+---------+-------+------+----------+-------------------------------+
| id | select_type | table | partitions | type     | possible_keys | key        | key_len | ref   | rows | filtered | Extra                         |
+----+-------------+-------+------------+----------+---------------+------------+---------+-------+------+----------+-------------------------------+
|  1 | SIMPLE      | sku   | NULL       | fulltext | idx_remark    | idx_remark | 0       | const |    1 |   100.00 | Using where; Ft_hints: sorted |
+----+-------------+-------+------------+----------+---------------+------------+---------+-------+------+----------+-------------------------------+
1 row in set (0.01 sec)
```

当全文索引与普通索引的查询条件同时出现时，MySQL 优先使用全文索引：
```
mysql> explain select * from sku where match(remark) against('小白菜') and price=10;
+----+-------------+-------+------------+----------+----------------------+------------+---------+-------+------+----------+-------------------------------+
| id | select_type | table | partitions | type     | possible_keys        | key        | key_len | ref   | rows | filtered | Extra                         |
+----+-------------+-------+------------+----------+----------------------+------------+---------+-------+------+----------+-------------------------------+
|  1 | SIMPLE      | sku   | NULL       | fulltext | idx_price,idx_remark | idx_remark | 0       | const |    1 |     5.00 | Using where; Ft_hints: sorted |
+----+-------------+-------+------------+----------+----------------------+------------+---------+-------+------+----------+-------------------------------+
1 row in set (0.00 sec)
```

但当全文索引与唯一性索引的查询条件同时出现时，MySQL 优先使用唯一性索引：
```sql
mysql> explain select * from sku where match(remark) against('小白菜') and name='小白菜';
+----+-------------+-------+------------+-------+--------------------------+---------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys            | key           | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+--------------------------+---------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | const | idx_uniq_name,idx_remark | idx_uniq_name | 1022    | const |    1 |    11.11 | Using where |
+----+-------------+-------+------------+-------+--------------------------+---------------+---------+-------+------+----------+-------------+
1 row in set (0.00 sec)
```

## range

`range` 发生在索引范围扫描，如 `<`、`<=`、`>`、`>=`、`!=`、`in`、`between`

```sql
mysql> explain select * from sku where id<300;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |  298 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
mysql> explain select * from sku where id in (1,3,5,7,9);
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    5 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
mysql> explain select * from sku where id!=3;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL | 4198 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set (0.00 sec)
mysql> explain select * from sku where price between 3 and 5;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | sku   | NULL       | range | idx_price     | idx_price | 5       | NULL | 1735 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
1 row in set (0.00 sec)
```

# 小结

1. 通常情况下，上述几种类型在查询性能上比较如下，越靠前性能越好：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range >index > All

2. 上面讲述的情况与示例，并不是百分百必现的，原因在于 MySQL 优化器不是单纯的检查索引定义情况，而是综合考虑到了表数据量、数据分布等情况决定最终查询计划的。

如，请看下如下查询的执行计划：
```sql
mysql> explain select * from sku where price<3;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | sku   | NULL       | range | idx_price     | idx_price | 5       | NULL | 2517 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
1 row in set (0.01 sec)
 
mysql> explain select * from sku where price<100;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | idx_price     | NULL | NULL    | NULL | 8393 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set (0.00 sec)
```
我们可以看到同样的查询语句执行计划不同，只是 where 条件稍有不同。原因在于当 `price<100` 时该条件下已经覆盖了绝大部分数据了，使用全表扫描性能由于索引扫描，也就是这时发生了索引失效。这也是我们使用 MySQL 时需要注意的一点。



