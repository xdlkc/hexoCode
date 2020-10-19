---
title: MySQL时间类型
date: 2019-11-18 16:10:29
tags:
categories: MySQL
---

Mysql关于时间有三种存储类型,是date,datetime,timestamp,三种的大小,表示格式,注意事项也有所不同,业务上碰到了有疑惑所以记录一下不同（历史文章归档）<!--more-->

- date

  date所表示的时间是只精确到天,也就是`YYYY-MM-DD`这样,范围就是 `1000-01-01` to `9999-12-31`,存储空间占用3个字节,其他的就没什么重点了

- datetime

  datetime所表示的时间就精确到了秒,即`YYYY-MM-DD HH:MM:SS`,而且范围是`1000-01-01 00:00:00` to `9999-12-31 23:59:59`,存储空间占用8个字节,时间的值的存储跟时区无关

- timestamp

  timestamp所表示的时间也精确到秒,即`YYYY-MM-DD HH:MM:SS`,但是范围稍微小一些,是`1970-01-01 00:00:01` UTC to `2038-01-19 03:14:07` UTC,可想而知存储空间就少了一下,占4个字节,时间的值是按照UTC时间来存储的

- 接下来重点说一下timestamp,因为timestamp有一个自动更新的特点,这也是我写这份笔记的原因,timestamp在设置字段时会指定字段是否会在插入和修改时修改timestamp字段为当前时间,这个特点有很大的便利点,因为很多时候我们在数据有问题时会去查数据库,如果这时数据库中有一个字段存了这条记录什么时候被修改那么会节省很多时间,所以这个类型还是挺有用的,接下来说一下它的实际用法

  ```mysql
  TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  ```

  	这条语句代表当前字段在记录创建或者修改时都会更新为当前时间,但是注意如果更新的记录修改的值与更新前一样,那么timestamp并不会更新

  ```mysql
  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  ```

  	这条语句代表当前字段创建时会被设置,但是以后就不能被修改了

  ```mysql
  TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  ```

  	这条语句代表设置时设成0,在每次更新时会更新成当前时间

  