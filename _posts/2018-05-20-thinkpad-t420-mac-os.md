---
layout: post
title: ThinkPadT420 安装黑苹果小记
categories: [瞎折腾]
description: ThinkPadT420 安装黑苹果小记
keywords: MacOS, 黑苹果
---

 周末闲的蛋疼，把一台老古董 ThinkPadT420装上了黑苹果，过程真的十分坎坷。简单的记录一下。

## 准备工作

一个U盘，一台GPT硬盘格式的Window（用于写入镜像，不知道MBR格式的是否可以。）

拔掉电池！！！（这个很重要，否则 会报错：`com.apple.driver.AppleIntelCPUPowermMangement XXX` 按照晚上的 添加`NullCPUPowerManagement.kext` 删除 CPUPowerManagement等，一点都不管用，我在这里卡了很久。

BIOS：从ThinkPad官网下载了最新的，貌似是1.5X（网上说用1.4x 没找到）

软件：镜像写入工具TransMac

### BIOS配置

启动项设置为 UEFI（Only）！！！

关掉独显！！！（参考：[Thinkpad T420机型BIOS切换独立显卡教程](http://heisetoufa.iteye.com/blog/1333309) )

## 写入镜像

其实随便找一份镜像10,11等版本的镜像都可，重要的是引导和驱动。主要参考了这里 （1） [T420完美安装OS X10.11 El Capitan超详细教程](https://forum.51nb.com/thread-1639171-1-1.html)

文章里镜像的链接已经失效了，所以我在这篇文章中找了个镜像，不过这篇教程中的驱动有问题，不建议用，我在这里也费了不少时间。（2）[【原创】我也来谈谈T420黑苹果安装](https://forum.51nb.com/thread-1579155-1-1.html) 其中镜像链接：http://bbs.pcbeta.com/viewthread-1592352-1-1.html

1. 用链接（2）的镜像 10.10.x 版本，然后用 TransMac将镜像写入U盘。

   然后拔掉U盘，重新插上可以看到一个EFI分区。（有可能重插后也无法显示EFI分区，可以进pe或者在mac上可以显示，也可以用DiskGenius 给EFI分区分配一个盘符）删除EFI分区中的

2. 删除上个步骤中EFI分区中的BOOT 和 CLOVER 两个文件夹，替换为从链接（1）里的EFI 中的BOOT 和CLOVER。（切记不要覆盖，直接删掉后拷贝新的）

## 安装

插上U盘，选 Boot OS X Install xxx xxx，进入安装。（此时安装如果有问题，记得用 -v模式，查看错误问题。）用上面的EFI文件应该是没有问题的。

我之前最多的遇到个 com.apple.driver.AppleIntelCPUPowermMangement XXX ，然后用了 NullCPUPowerManagement.kex，删掉CPUPowerManagement 等都没用。电源问题拔掉电池就可以了，装好系统后再装上去。

进度条跑完进入到安装界面基本万事大吉了，用磁盘工具分区后直接安装，其他教程里有详细操作。安装问题：

1. 安装的时候如果提示不能验证（这个是证书问题）：

   `提示:应用程序副本不能验证 它在下载过程中可能已遭破坏或篡改`  

   解决：直接打开终端 输入：`date 122014102015.30` 即可（注意区分大小写和空格。参考这里：[安装OS X，提示:应用程序副本不能验证 它在下载过程中可能已遭破坏或篡改 ](https://www.applex.net/threads/os-x.57768/) )

2. 安装进度条最后一秒要很久，我差不多等了20分钟。耐心等待即可。

3. 第一次安装进度条跑完后，会重启，发现还是没有启动选择项，别急，直接重新上一个步骤进去再装一次。（这一次不需要用磁盘工具分区等，直接装） 第二次安装很快。

装完后重启到 CLOVER界面可以看到系统了。进去配置即可。

## 收尾工作

1. 装声卡。
2. 写入引导到系统。
3. 装上拆掉的电池。

前两项参考 链接（1）里 下载音频驱动VoodooHDA 和CLOVER 按照其中的操作即可。

用到的资料截个图（这篇文章在windows上写的，新装的mac还在升级10.11版本）

![t420_mac_os](http://539go.com/Img/Other/OS/t420_mac_os.png)

## 关于升级

后面在网上下载了 10.11.6的镜像，在T420 mac上单独分一个区（如：分区B)，然后拷贝镜像进去安装即可。（安装后会重启，重启进入Clover 然后选分区B！！！进入后照常安装，不过安装盘选你的mac系统盘，资料不会丢。进度条跑完以后，重新到Clover 就可以选系统盘进入了，进入后就是升级后的系统）