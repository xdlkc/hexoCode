---
title: Linux & Mac常用命令集锦
date: 2018-05-03 17:12:17
tags:
categories: Linux
---

# Linux & Mac常用命令集锦

此篇是个人经常用到的常用命令(持续更新)

<!--more-->

# 系统相关

```bash
# 获取eth0的ip地址
ifconfig eth0 | grep "inet" | awk '{ print $2}'
# 查看nfs相关依赖
rpm -qa |grep nfs-utils
# 安装nfs工具包
yum -y install nfs-utils
# 启动nfs
systemctl start nfs-utils
systemctl enable  nfs-utils
```

# Git相关

```bash
# 获取当前项目git代理
git config --local http.proxy
# 设置当前git项目https代理
git config --local https.proxy 'socket5://127.0.0.1:1080
# 设置当前git项目http代理
git config --local http.proxy 'socket5://127.0.0.1:1080'
# 设置全局git项目https代理
git config --global https.proxy 'socket5://127.0.0.1:1080
# 设置全局git项目http代理
git config --global http.proxy 'socket5://127.0.0.1:1080'
# 取消当前git项目http代理
git config --local --unset http.proxy
# 取消当前git项目https代理
git config --local --unset https.proxy
```

# 文件相关

```bash
# 查找大于8M的文件
find . -type f -size +8M -print0 | xargs -0 ls -lh
```

# 文本相关

```bash
# 搜索某个目录下所有文件是否包含stretch
grep -R stretch /etc/*
# 搜索当前目录下所有文件是否包含apple，并显示行号
grep -nr apple *
cat family_portal_access.log.20190708 | grep "GET / HTTP/1.1" | wc -l
# 查看日志每秒并发情况
cat a.log | awk -F '[' '{print $2}' | awk '{print $1}' | sort | uniq -c |sort -k1,1nr
# 统计多个日志匹配的行数
for i in {1..3}; do cat access.log.2019070$i | grep -E "/index|/user" | wc -l; done
# 查询日志并显示行数
nl a.log | grep "1564401537563" -A 50000 | tail -n 10
```

# 网络相关

```bash
netstat -ano
ip addr
```

# 软件相关

```bash
# 升级oh-my-zsh
upgrade_oh_my_zsh
```

## 进程相关

```bash
# 查看线程占句柄数
ulimit -a
# 查看系统打开句柄最大数量
more /proc/sys/fs/file-max
# 查看打开句柄总数
lsof|awk '{print $2}'|wc -l
# 根据打开文件句柄的数量降序排列，其中第二列为进程ID：
lsof|awk '{print $2}'|sort|uniq -c|sort -nr|more
```



# Mac相关

```bash
# 更新系统时间
sudo sntp -sS time.apple.com
defaults write com.apple.BluetoothAudioAgent "Apple Bitpool Min (editable)" 35
defaults write com.apple.BluetoothAudioAgent "Apple Initial Bitpool Min (editable)" 53
defaults write com.apple.BluetoothAudioAgent "Apple Initial Bitpool (editable)" 35
pipreqs . --force
# 修改mac游标移动速度
defaults write NSGlobalDomain KeyRepeat -int 1
```

