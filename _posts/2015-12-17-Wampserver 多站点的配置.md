---
layout: post
title: "Wampserver 多站点的配置"
keywords: 
description: ""
category: 
tags: 
---

<!--markdown-->我被这东西困扰了一晚上最后不得已卸载了wamp重装是不是说明我变笨了.  
版本:WampServer 2.5  
  
## 1. 配置httpd-vhosts.conf文件  

E:\wamp\bin\apache\apache2.4.9\conf\extra\httpd-vhosts.conf

其中E:\wamp是你的Wampserver安装目录
  
打开httpd-vhosts.conf文件后,会发现如下内容:  
  
    <VirtualHost *:80>  
        ServerAdmin webmaster@dummy-host.example.com  
        DocumentRoot "c:/Apache24/docs/dummy-host.example.com"  
        ServerName dummy-host.example.com  
        ServerAlias www.dummy-host.example.com  
        ErrorLog "logs/dummy-host.example.com-error.log"  
        CustomLog "logs/dummy-host.example.com-access.log" common  
    </VirtualHost>  
  
    <VirtualHost *:80>  
        ServerAdmin webmaster@dummy-host2.example.com  
        DocumentRoot "c:/Apache24/docs/dummy-host2.example.com"  
        ServerName dummy-host2.example.com  
        ErrorLog "logs/dummy-host2.example.com-error.log"  
        CustomLog "logs/dummy-host2.example.com-access.log" common  
    </VirtualHost>  
  
拷贝其中的一份在下面.并做如下编辑:  
  
    <VirtualHost *:80>  
        DocumentRoot "D:/_workspace/_phpweb/test01/"  
        ServerName test01.com  
        <Directory "D:/_workspace/_phpweb/test01/">  
            Options Indexes FollowSymLinks  
            AllowOverride all  
            Require all granted  
        </Directory>  
    </VirtualHost>  
  
其他的杂项没什么用不管.  
  
Documentroot 为网页的目录.  
  
ServerName 是站点信息.  
  
Directory 是配置站点权限.  
  
即意味着我的网页需要保存在 D:/_workspace/_phpweb/test01/ 并用 test01.com 访问  
  
**注意事项:**  
  
我从网页目录的文件管理菜单复制的文件目录是这样的  
  
    D:\_workspace\_phpweb\test01  
  
而实际上在配置文件中需要使用"/"而不是"\".另外在末尾需要添加"/";  
  
即　`D:/_workspace/_phpweb/test01/`  
  
## 2. 配置httpd.conf文件  
  
![blob.png](/usr/uploads/2015/12/17/1450316765462892.png "1450316765462892.png")  
  
打开httpd.conf文件,查找下面三个内容并注释掉(去掉前面的&quot;#&quot;号)  
  
    LoadModule php5_module "d:/wamp/bin/php/php5.5.12/php5apache2_4.dll"  
    PHPIniDir d:/wamp/bin/php/php5.5.12  
    Include conf/extra/httpd-vhosts.conf  
  
如果有注释就去掉,没有就不用管了.我只发现了第三项有注释.  
  
## 3. 编辑hosts文件  
  
找到如下目录中的hosts文件.  
`C:\Windows\System32\drivers\etc`  
  
打开文件找到最后的127.0.0.1    localhost内容.在后面添加  
  
127.0.0.1 &nbsp; &nbsp; &nbsp; test01.com  
  
其中 "test01.com" 就是刚才设置的站点.  
  
> 直接修改也许会失败,这时可以将 hosts文件拷贝到其他位置,修改后再拷贝回去覆盖原文件.  
  
至此,多站点已经配置完成.重启WampServer所有服务.即可打开新站点了.  
