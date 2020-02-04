---
layout: post
title: 在IOS设备上抓包 tcp udp等协议
categories: [IOS,Unity]
description: 
keywords: 开发, 
---

1. 官网下载软件Wireshark  [https://www.wireshark.org/download.html]( https://www.wireshark.org/download.html)
2. 连接IOS设备，打开ITunes，在概要页面查看 Itunes 查看 UDID 并复制。
3. 打开Wireshark。此时没有手机的接口。
4. 打开Shell ，输入`rvictl -s [UUID]` 
5. 此时在 Wireshark 中会出现 刚刚创建的名为 rvi0 的接口，双击进去抓包分析。