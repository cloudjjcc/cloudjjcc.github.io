---
author: cloudjjcc
title: tcp必知必会
date: 2023-09-19
description: tcp协议相关的问题总结
tags: [tcp]
categories: [网络原理]
---
## 连接的建立
### 三次握手
正常情况下，建立连接需要经过三次握手
![alt 三次握手](tcp_connect.webp)
## 连接的终止
### 正常终止（四次挥手）
正常情况下，断开连接需要经过四次挥手
![alt 三次握手](tcp_close.webp)

### 异常终止（RST报文）
主动终止方发送RST报文立刻终止连接

## 处于TIME_WAIT状态的连接过多怎么办？
### 如何查看连接状态？
``` shell
❯ netstat -ant|awk '/^tcp/ {++S[$NF]} END {for(a in S) print (a,S[a])}'
LISTEN 32
CLOSE_WAIT 7
TIME_WAIT 3
ESTABLISHED 119
```

### 什么情况下会产生大量的TIME_WAIT连接？
我们知道TIME_WAIT是主动正常断开连接的一方所处的最后一个状态，这个状态会经历2MSL时长（2*30s）
在Nginx反向代理中，在高并发短连接的场景下，可能导致大量的TIME_WAIT连接

### TIME_WAIT 危害
+ 对于客户端：处于TIME_WAIT状态的连接占用的端口无法被再次使用，可能会导致端口耗尽，无法建立新的连接
+ 对于服务端：大量的连接不被释放会消耗内存

### 控制最大TIME_WAIT连接数
Linux提供了tcp_max_tw_buckets参数，当TIME_WAIT连接数等于此参数值时，新的连接关闭时不会再经历TIME_WAIT状态，而是直接关闭。
### 重用连接
Linux提供了tcp_tw_reuse 参数允许复用处于TIME_WAIT状态的连接