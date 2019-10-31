---
title: linux命令组合
date: 2018-05-03 17:12:17
tags:
categories: linux
---

# Linux命令组合

此篇是个人经常用到的常用命令(持续更新)

<!--more-->

1. 获取eth0的ip地址

   ```shell
   $ ifconfig eth0 | grep "inet" | awk '{ print $2}' 
   ```

   

2. 更新系统时间

   ```shell
   sudo sntp -sS time.apple.com
   ```

   