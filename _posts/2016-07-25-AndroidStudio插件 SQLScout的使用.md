---
layout: post
title: "AndroidStudio插件 SQLScout的使用"
keywords: 
description: ""
category: Android
tags: 
---

<!--markdown-->SQLScout插件 Android开发时用于调试Sqlite的神器。  

## 安装  
  
Setting --> Plugings --> Browse Repositories  搜索 SQLScout 安装即可。

如果像我一样安装失败，可以直接从 [官网](http://www.idescout.com) 下载，然后从AndroidStudio里选择本地文件安装 。  
  
## 使用  
  
1. 安装 SQLScout 插件。  
  
2. 打开需要调试的工程，在整个工程的 build.gradle 文件中 添加：  
  
   ```  
     allprojects {  
         repositories {  
             jcenter()  
             maven {  
                 url 'http://www.idescout.com/maven/repo/'  
             }  
         }  
     }  
   ```  
  
3. 在module 中的 build.gradle 中添加依赖：  
  
   ```  
   compile 'com.idescout.sql:sqlscout-server:1.0'  
   ```  
  
4. 在相应的Activity的 oncreate 方法中，加入下面一行代码：  
  
   ```  
   SqlScoutServer.create(this, getPackageName());  
   ```  
   此时，已经可以使用这个插件了。  
  
5. 简单操作  
  
   在工程的右侧，有 SQLiteExplorer （装好插件才有）  
  
    ![2016-07-25_200500](http://539go.com/usr/uploads/2016/07-25/2016-07-25_200500.png)  
  
   然后选择 + 号中的第一项。  
  
    ![2016-07-25_200833](http://539go.com/usr/uploads/2016/07-25/2016-07-25_200833.png)  
  
   直接确定就好。  
  
    ![2016-07-25_200956](http://539go.com/usr/uploads/2016/07-25/2016-07-25_200956.png)  
  
   此时，就可以看到所选表中的字段了。选中相应的表，点击上面类似表格的按钮，就可以以表格的形式看到数据库中的信息了。  
  
    ![2016-07-25_201255](http://539go.com/usr/uploads/2016/07-25/2016-07-25_201255.png)  
  
     ![2016-07-25_201225](http://539go.com/usr/uploads/2016/07-25/2016-07-25_201225.png)  
  
  
当然，还有一些其他的功能。  
  
这个插件非常的好用，但是是付费的，暂时也没发现什么破解的方法。  
  
  
  
