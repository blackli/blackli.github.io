---
layout: post
title: 高负载 Linux 服务器 TCP TIME-WAIT 状态处理
category: 开发
description: 这是一篇翻译的文章，最近在处理一个网络问题，原因是由于开启了 net.ipv4.tcp_tw_recycle，使用tcp_tw_recycle来处理TIME-WAIT一直是一个误区，这篇文章全面的总结了各种处理 TIME-WAIT 的方法。
tags: ["tcp"]
---

文章原文：http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html
文章作者：Vincent Bernat

一句话回答就是：不要开启 `net.ipv4.tcp_tw_recycle`。

Linux内核文档提供的信息很模糊：

> Enable fast recycling TIME-WAIT sockets. Default value is 0. It should not be changed without advice/request of technical experts.

另外一个类似选项`net.ipv4.tcp_tw_reuse`说得更详细点，不过描述的差不多：

> Allow to reuse TIME-WAIT sockets for new connections when it is safe from protocol viewpoint. Default value is 0. It should not be changed without advice/request of technical experts.

文档说的很含糊，导致现在很多性能调优建议，为了减少`TIME-WAIT`数量，可以把这两个选项设置成1。但是，`tcp(7)`文档中说开启`net.ipv4.tcp_tw_recycle`将会导致NAT设备后的两个连接请求可能会失败，这个问题很难发现，一不小心就会掉到坑里去。

> Enable fast recycling of TIME-WAIT sockets. Enabling this option is not recommended since this causes problems when working with NAT (Network Address Translation).

这篇文章给这个问题提供一个尽可能详细的解释，避免大家犯同样的错误。

需要说明下，虽然我们这里都说的是`ipv4`，配置名字也是`net.ipv4.tcp_tw_recycle`，单其实也适用于`IPv6`。

# `TIME-WAIT`状态介绍

首先回顾下`TIME-WAIT`状态，看一下面的TCP状态变迁图：

只有首先关闭连接的一方才会进入`TIME-WAIT`状态，另外一方可以很快的回收连接。

用`ss-tan`看一下当前的连接状态：

	$ ss -tan | head -5
	LISTEN     0  511             *:80              *:*     
	SYN-RECV   0  0     192.0.2.145:80    203.0.113.5:35449
	SYN-RECV   0  0     192.0.2.145:80   203.0.113.27:53599
	ESTAB      0  0     192.0.2.145:80   203.0.113.27:33605
	TIME-WAIT  0  0     192.0.2.145:80   203.0.113.47:50685

## 目的

保持`TIME-WAIT`状态有两个目的：


