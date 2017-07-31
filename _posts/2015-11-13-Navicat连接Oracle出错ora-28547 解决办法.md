---
layout: post
title: "Navicat连接Oracle出错ora-28547 解决办法"
keywords: 
description: ""
category: 
tags: 
---

<!--markdown-->    ora-28547 connection to server failed,probable oracle net admin error  

    Navicat是一个可以连接多种数据库的管理工具。
  
    今天连接Oracle的时候提示 ora-28547 connection to server failed,probable oracle net admin error  
  
解决办法  
  
1，打开 Navicat permium——工具——选项——OCI  
  
2，在OCI bilrary那一栏需要个 oci.dll  
  
        需要去Oracle官网下载：  
  
[http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html)  
  
        对应自己的系统版本下载即可。Oracle的官网下载需要注册，而且资料填老多。提供windows的下载吧，刚才自己用的时候下的。 [http://yunpan.cn/cLYIW6XEStjRb](http://yunpan.cn/cLYIW6XEStjRb)  访问密码 7814  
  
3，对应32或者64位下载后，解压到某个地方。比如C盘 C:\instantclient_12_1  
  
    然后 如上所说 Navicat permium——工具——选项——OCI 选择 C:\instantclient_12_1\oci.dll  
  
即可。  
