---
title: linux命令组合
date: 2018-05-03 17:12:17
tags:
categories:
---

# Linux命令组合

本篇文章主要记录一些linux的命令组合来输出一些想要的结果(持续更新)

<!--more-->

1. 获取eth0的ip地址

   ```shell
   $ ifconfig eth0 | grep "inet" | awk '{ print $2}' 
   ```

   ​

