---
title: mysql查询语句类型转换
date: 2019-11-18 16:09:31
tags:
categories: MySQL
---

（历史文章归档）<!--more-->

执行`show`语句看一下一个普通表的结构:

```mysql
SHOW CREATE TABLE test.user;
```
输出:

```mysql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `name` varchar(20) NOT NULL,
  `age` int(11) DEFAULT NULL,
  `sex` varchar(10) NOT NULL,
  `card` bigint(20) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `card` (`card`),
  KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户信息表'
```
注意到这里`id`字段是`bigint`类型,那么如果我查询记录时`where`条件判断`id`等于一个字符串会怎样呢?来试一下:

```mysql
SELECT * FROM test.user WHERE id = '1-fad';
```
输出:

![](http://upload-images.jianshu.io/upload_images/1890983-96ad2e6c5a1bfca7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以发现竟然查出来了,而且简单对比一下,感觉查出来的`id`就是字符串中的`1`啊,那么是为什么呢,其实mysql针对这种需要类型转换的时候进行了隐式转换,像字符串转为`int`呢方法就是将字符串从前向后截取到非数字的地方,然后把后面全部改成0并求和,相当于转化的值就是第一个非数字的位置前面代表的数字.
这里后来也发现了一个其他的问题,就是在查看执行计划时发现如果截取的数字超过字段对应的类型的话是不会查询的,来比较一下:
在范围内的情况:

```mysql
EXPLAIN SELECT * FROM test.user WHERE id = '1-fad';
```
![](http://upload-images.jianshu.io/upload_images/1890983-47be301fea4f9815.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
超过范围的情况:

```mysql
EXPLAIN SELECT * FROM test.user WHERE id = '1231231231231-fad';
```
![](http://upload-images.jianshu.io/upload_images/1890983-a06e4886bc244d87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
结果显而易见,各项都是空,mysql直接返回结果