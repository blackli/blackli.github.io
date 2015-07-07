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


