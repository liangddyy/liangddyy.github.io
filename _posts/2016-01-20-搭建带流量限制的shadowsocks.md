---
layout: post
title: "搭建带流量限制的shadowsocks"
keywords: 
description: ""
category: 
tags: Shadowsocks
---

<!--markdown--><!-- index-menu -->  

## 1. Ubuntu搭建shadowsocks 带流量控制

### 1.1 Python版shadowsocks安装  
  
>参考：[ubuntu 服务器搭建 Shadowsocks 服务][1]  

更新软件源：  
  
    sudo apt-get update
  
安装 PIP 环境：  
  
    sudo apt-get install python-pip  
  
安装 shadowsocks：  
  
    sudo pip install shadowsocks  
  
### 1.2 安装shadowsocks的流量控制模块安装及配置  
  
>这是一个Github上[ss-bash][2]的项目  
  
### 1.2.1 安装及使用  
  
#### 下载软件  
  
    git clone https://github.com/hellofwy/ss-bash  
  
或者：  
  
    wget https://github.com/hellofwy/ss-bash/archive/v1.0-beta.3.tar.gz  
  
#### 首次运行时，先新建用户  
  
例如新用户端口为8388，密码为passwd，流量限制为10GB，执行：  
  
    sudo ss-bash/ssadmin.sh add 8388 passwd 10G  
#### 启动ssserver  
  
    sudo ss-bash/ssadmin.sh start  
  
### 1.2.2 自定义ssserver的配置  
  
>详细配置参考：[ss-bash][3]  
  
>详细命令（包含用户的增删改以及流量限制）：[SShelp][4]  
#### 注意事项：  
  
* 按照步骤1安装的shadowsocks则ssserver文件位置不用修改。  
* 编辑配置文件时，可能遇到权限不够。修改权限：  
  
  
    sudo chmod 600 ××× （只有所有者有读和写的权限）  
    sudo chmod 644 ××× （所有者有读和写的权限，组用户只有读的权限）  
    sudo chmod 700 ××× （只有所有者有读和写以及执行的权限）  
    sudo chmod 666 ××× （每个人都有读和写的权限）  
    sudo chmod 777 ××× （每个人都有读和写以及执行的权限）  
    (xxx为对应的文件，如 /111/shadowsocks.json)  
  
## 2. windows Server搭建shadowsocks  
  
windows Server的系统就不用敲命令行了。不过没有流量控制脚本  
  
*[shadowsocks服务端工具][5]*  
  
下载后打开 config.json 文件配置。  
.btw 最好使用notepad++编辑而不要用记事本或者写字板）  
### 多用户配置：  
(端口号，密码 加密方式，超时时间)  
  
    {  
            "port_password": {  
                    "8389": "pwd1",  
                    "8390": "pwd2",  
                    "8387": "pwd3",  
                    "8388": "pwd4"  
            },  
            "method": "aes-256-cfb",  
            "timeout": 600  
    }  
  
保存之后直接运行 shadowsocks.exe即可。  
  
## 3. 客户端  
PC客户端：  
shadowsocks客户端可以直接去[官网][6]下载  
  
Android端-影梭：  
  
官网有对应的Google Play的下载地址。不方便。  
提供个镜像 [地址1][7]  
  
## 4. 极路由shadowsocks插件安装  
>参考：  
[极路由Shadowsocks家庭无痛翻墙实践][8]  
或者  
>[极路由安装ss插件][9]  
  
一键安装ss 脚本：  
  
    cd /tmp && wget http://cdn.is26.com/file/hiwifi/shadow.sh && sh shadow.sh && rm shadow.sh  
  
一键更新路由表：  
  
    cd /etc/gw-redsocks/gw-shadowsocks && wget http://this.is26.com/download/gfw.txt && cat gfw.txt >> gw-shadowsocks.dnslist && /etc/init.d/dnsmasq restart  
  
安装好后重启生效。  
  
***  
>其他：[Shadowsocks Python版一键安装脚本][10]  
  
  [1]: https://github.com/hellofwy/ss-bash  
  [2]: https://github.com/hellofwy/ss-bash  
  [3]: http://www.ilucong.net/lulu/windows-shadowsocks.html  
  [4]: http://www.ilucong.net/lulu/windows-shadowsocks.html  
  [5]: http://www.ilucong.net/lulu/windows-shadowsocks.html  
  [6]: https://shadowsocks.org/  
  [7]: http://www.zhangzhe.info/2014/12/shadowsocks-apk-android/  
  [8]: https://luolei.org/hiwifi-shadowsocks/  
  [9]: https://rogerchen.info/install-shadowsocks-plugin/  
  [10]: https://teddysun.com/342.html  
