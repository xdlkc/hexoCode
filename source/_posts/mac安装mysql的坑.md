---
title: mac安装mysql的坑
date: 2019-11-05 11:09:20
tags:
categories: mysql
---

## 卸载MySQL

```shell
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /private/var/db/receipts/mysql
```

<!--more-->