---
title: MySQL 执行计划中的 extra 详解
author: Uphie
date: 2022-12-03 17:25:00 +0800
categories: [技术]
tags: [mysql]
math: true
toc: true
---

上篇文章介绍了 MySQL 执行计划中的 type，这篇文章介绍执行计划中的 Extra，并且沿用上篇文章中的数据库表。

Extra 描述了执行计划中附加的更加具体的执行计划信息，能帮助开发人员了解更详尽的sql执行方案。


# Using where

不是读取表的所有数据，或者不是仅仅通过索引就可以获取所有需要的数据。

```sql
mysql> explain select * from sku where id<50;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |   48 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
```

# Using index

说明查询时使用到了覆盖索引。

```sql
mysql> explain select price from sku;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sku   | NULL       | index | NULL          | idx_price | 5       | NULL | 8393 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
1 row in set (0.01 sec)
```

# Using index condition

表示先按条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行。

```sql
mysql> explain select * from sku where cat_1=2 and cat_3=2;
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | sku   | NULL       | ref  | idx_cat       | idx_cat | 4       | const |   81 |    10.00 | Using index condition |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
1 row in set (0.01 sec)
```

# Using index for group-by

使用索引进行分组操作，包括 `distinct`、`group by` 操作。

```sql
mysql> explain select distinct cat_1 from sku;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | sku   | NULL       | range | idx_cat       | idx_cat | 4       | NULL |  101 |   100.00 | Using index for group-by |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
1 row in set (0.02 sec)
 
mysql> explain select cat_1 from sku group by cat_1;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | sku   | NULL       | range | idx_cat       | idx_cat | 4       | NULL |  101 |   100.00 | Using index for group-by |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
1 row in set (0.00 sec)
```

# Using union

发生在同时使用多个索引的情况。

如：
```sql
mysql> explain select * from sku where name='小白菜' or alias='小白菜';
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
| id | select_type | table | partitions | type        | possible_keys           | key                     | key_len   | ref  | rows | filtered | Extra                                             |
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
|  1 | SIMPLE      | sku   | NULL       | index_merge | idx_uniq_name,idx_alias | idx_uniq_name,idx_alias | 1022,1023 | NULL |    2 |   100.00 | Using union(idx_uniq_name,idx_alias); Using where |
+----+-------------+-------+------------+-------------+-------------------------+-------------------------+-----------+------+------+----------+---------------------------------------------------+
1 row in set (0.02 sec)
```

# Using filesort

发生在使用文件进行排序的时候，这个时候可能是因为没有可排序的索引或者索引失效而使用了扫描全表，将获得的数据借助文件存储进行排序。这种排序效率往往不高。

```sql
mysql> explain select * from sku where cat_1=1 order by create_time;
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+----------------+
|  1 | SIMPLE      | sku   | NULL       | ref  | idx_cat       | idx_cat | 4       | const |   93 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+----------------+
1 row in set (0.00 sec)
mysql> explain select * from sku order by price;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 8393 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set (0.01 sec)
```

# Backward index scan

指的是倒序索引扫描，具体来说是对于按正序创建的索引，MySQL（从 MySQL8开始）会支持倒序索引扫描。

```sql
mysql> explain select price from sku order by price desc;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+----------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                            |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+----------------------------------+
|  1 | SIMPLE      | sku   | NULL       | index | NULL          | idx_price | 5       | NULL | 8302 |   100.00 | Backward index scan; Using index |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+----------------------------------+
1 row in set (0.00 sec)
mysql> explain select producer_id,batch_num from sku order by producer_id desc,batch_num desc;
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+----------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key           | key_len | ref  | rows | filtered | Extra                            |
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+----------------------------------+
|  1 | SIMPLE      | sku   | NULL       | index | NULL          | idx_batch_num | 1028    | NULL | 8302 |   100.00 | Backward index scan; Using index |
+----+-------------+-------+------------+-------+---------------+---------------+---------+------+------+----------+----------------------------------+
1 row in set (0.00 sec)
```

# Using temporary

表示使用到了临时表。

使用 group by ：
```sql
mysql> explain select cat_1,sum(1) as cnt from sku group by cat_1 order by cnt;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                                        |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------+
|  1 | SIMPLE      | sku   | NULL       | index | idx_cat       | idx_cat | 12      | NULL | 8393 |   100.00 | Using index; Using temporary; Using filesort |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------+
1 row in set (0.00 sec)

```

对非索引字段（包括全文索引字段）进行去重：
```sql
mysql> explain select distinct supplier_id from sku;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 8302 |   100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set (0.00 sec)
mysql> explain select distinct remark from sku;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | sku   | NULL       | ALL  | idx_remark    | NULL | NULL    | NULL | 8302 |   100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set (0.00 sec)
```

# Select tables optimized away

表示查询时根据索引字段优化了查询。

如查询索引字段 price 的最小值：
```sql
mysql> explain select min(price) from sku;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set (0.01 sec)
```

# Impossible WHERE

where 语句不可能查到。这是无效的查询。

如：
```sql
mysql> explain select * from sku where cat_1 is null;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | Impossible WHERE |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
1 row in set (0.00 sec)
```
注：字段 cat_1 不可为null。


# Impossible HAVING

HAVING子句始终为false,不会命中任何行。

笔者未找到这种情况下的 sql 示例，待补充。


# Not exists

MySQL 能对 LEFT JOIN 优化，在找到符合 LEFT JOIN 的行后，不会为上一行组合中检查此表中的更多行。
```sql
mysql> explain select sku.* from sku left join sku_stock on sku.id=sku_stock.id where sku_stock.id is null;
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------+------+----------+--------------------------------------+
| id | select_type | table     | partitions | type   | possible_keys | key     | key_len | ref         | rows | filtered | Extra                                |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------+------+----------+--------------------------------------+
|  1 | SIMPLE      | sku       | NULL       | ALL    | NULL          | NULL    | NULL    | NULL        | 8302 |   100.00 | NULL                                 |
|  1 | SIMPLE      | sku_stock | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.sku.id |    1 |   100.00 | Using where; Not exists; Using index |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------+------+----------+--------------------------------------+
2 rows in set (0.00 sec)
```

# no matching row in const table

存在一个空表，或者没有行能够满足唯一索引条件。

关联查询：
```sql
mysql> explain select sku.* from sku left join sku_stock on sku.id=sku_stock.id where sku_stock.id=10;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | no matching row in const table |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
1 row in set (0.00 sec)
```
注：sku_stock 表中不存在 id=10 的数据。

MyISAM 引擎下空表查询：
```
mysql> explain select * from book;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | no matching row in const table |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+--------------------------------+
1 row in set (0.00 sec)
```
注：book 表为空表，MyISAM 引擎。InnoDB 引擎下空表查询不会是 `no matching row in const table`。

# No matching min/max row

是指没有匹配的数据行来执行 `min`、`max` 查询。

```sql
mysql> explain select max(price) from sku where price>30;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                   |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | No matching min/max row |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------------------+
1 row in set (0.01 sec)
```

注：不存在 price>30 的数据行。

# Zero limit

使用了 `limit 0`。这是无效的查询。
```sql
mysql> explain select * from sku limit 0;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra      |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | Zero limit |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
1 row in set (0.01 sec)
```

# No tables used

查询未使用到表。

如：
```sql
mysql> explain select now();
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | No tables used |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set (0.00 sec)
```


参考连接：[https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_extra](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_extra)