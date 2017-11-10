---
layout: post
title: 极路由Go免Root安装Shadowsocks插件
categories: [Other]
description:  极路由Go免Root安装Shadowsocks插件
keywords:  
---

[](http://539go.com/2016/01/20/%E6%90%AD%E5%BB%BA%E5%B8%A6%E6%B5%81%E9%87%8F%E9%99%90%E5%88%B6%E7%9A%84shadowsocks/)

以前在这篇文章[搭建带流量限制的shadowsocks](http://539go.com/2016/01/20/%E6%90%AD%E5%BB%BA%E5%B8%A6%E6%B5%81%E9%87%8F%E9%99%90%E5%88%B6%E7%9A%84shadowsocks/) 的末尾提过在极路由上装shadowsocks插件，由于需要root极路由，申请开发者权限，不仅流程麻烦，而且会失去保修，非常不便。

后面又入了个极路由Go，下面说下怎么在极路由Go（其他极路由设备也可以）上免root安装ss插件。

## 安装

在电脑上打开极路由的后台是这样的

![home](/Img/Other/hiwifi/home.png)


完全手机的后台，此时只需要将地址栏的`admin_mobile` 改成 `admin_web` 就可以进电脑版的web后台了。

![adminpage](/Img/Other/hiwifi/adminpage.png)

然后

智能插件 - 去往插件市场 - 全部插件

然后随便在插件市场选一个插件 

地址是类似于是这样的 

```
https://app.hiwifi.com/store.php?m=plugins&a=install&rid=r1593903593&sid=382974826
```

将后面的一串数字(382974826)改成 `163116535` ，就可以看到ss插件了。安装即可。

## 使用

![setting](/Img/Other/hiwifi/setting.png)

安装好以后，在配置参数页面配置好自己的账号密码以及加密方式。

然后白名单模式需要填写一个名单，名单内的网站会走代理。代理的网址可以在github上找到相应的项目。（当然，也可以全局模式）

这里我已经提取了一份[gfwlist.txt](/Img/Other/gfwlist.txt) （不是最新的），将内容复制到域名列表即可。

启用插件。