---
title: MySQL知识点
date: 2019-08-27 19:56:22
tags:
---

### JOIN

三种算法：

- Nested Loop Join：常用
- Hash Join
- Merge Sort Join

类型：

- JOIN: 如果表中有至少一个匹配，则返回行
- LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行
- RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
- FULL JOIN: 只要其中一个表中存在匹配，就返回行

### 查询优化

left join 优化：引擎使用nested loop join算法，通过驱动表的结果集作为循环基础数据，一条条地通过该结果集作为过滤条件到下一个表中查询数据，然后合并结果。left join左边的表一般为大表，所以以其为驱动表时查询效率会很低，方案是使用join，引擎一般会选择小表作为驱动表来查询。

## 索引

三星：将相关的记录放在一起，索引的数据顺序与查询的排列顺序一致，索引中的列包含了查询中需要的全部列

全文索引，哈希索引

### 聚簇索引

叶子结点存储的是整行数据，且InnoDB只支持用主键做聚簇索引，非主键的索引的叶子结点的值是主键的值

Innodb_autoinc_lock_mode

### 覆盖索引

## 主从复制

bin_log

master IO线程，slave IO线程，SQL线程

![image-20190903122054120](https://tva1.sinaimg.cn/large/006y8mN6ly1g6m7mllo88j30um0diank.jpg)