---
title: ansible-多台服务器管理工具
date: 2018-05-03 14:33:40
tags:
categories:
---

## ansible-多台服务器管理工具

最近爬虫又遇到了麻烦事，起因是在顺（jian）利（nan）搭起多台服务器执行爬虫任务时发现每次代码更新后，都需要进入每台服务器执行git拉取命令，而且执行任务时需要每台都执行一遍相同的命令如：`scrapy crawl xxx`很是费力，尤其当出现问题时还要重新输入一遍，这时经过多方百度，查到了ansible这个运维工具，

<!--more-->

## 安装ansible

安装可以通过apt直接安装：

```shell
$ sudo apt-get update
$ sudo apt-get install ansible 
```

## 配置ansible

1. 首先将服务器的ip地址加入 到`/etc/ansible/hosts`文件中
   ![](http://otzriul1v.bkt.clouddn.com/18-5-3/93227311.jpg)

2. 生成rsa密钥

   ```shell
   $ ssh-keygen -t rsa
   ```

3. 将密钥复制到服务器上以便免密登录

   ```shell
   $ ssh-copy-id -i ~/.ssh/id_rsa.pub '服务器用户名'@服务器ip'
   ```

4. 测试一下

   ![](http://otzriul1v.bkt.clouddn.com/18-5-5/94494715.jpg)

   可以看到四台服务器均成功执行


## ansible常用参数

