---
title: TCP/IP协议笔记
date: 2020-10-14 17:53:08
tags: TCP
categories: TCP/IP
---

SYN重试：指数回退，第1次3秒，第2次6秒，第3次12秒……

net.ipv4.tcp_sync_retries：表示一次主动打开中尝试重新发送SYN报文段的最大次数

net.ipv4.tcp_syncack_retries：表示在响应对方的一个主动打开请求时尝试重新发送SYN+ACK报文段的最大次数

TIME_WAIT状态存在的原因：TCP/IP协议里描述为“允许任何受制于一条关闭连接的数据报被丢弃”，

## TCP超时与重传

超时重传计数器：RTO

二进制指数回退

两个阈值（R1和R2）：R1表示TCP向IP层传递“消极建议”之前，愿意尝试重传的次数。R2表示TCP应放弃当前连接的时机。

RTO计算方法：SRTT← α(SRTT)十(1一α) RTTs

## Q&A