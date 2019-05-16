---
layout: post
title: 关于MacOS中硬链接的问题
categories: [其他]
description: some word here
keywords: 
---

有时候开发时，需要引用自己之前的代码库，同时希望引用的代码在原来位置的改动或者在新位置的改动能同步。虽然有同步软件能实现，但使用起来麻烦，实际上用系统自带的硬链接功能，即可解决这个问题。

所以之前在Windows下，就做了个在 Unity中 快速建立硬链接文件夹硬链接的工具 [UnityKitModule](https://github.com/liangddyy/UnityKitModule)

然后现在基本在Mac上办公了，看了 [mac 系统中文件的软链接、硬链接](https://slarker.me/mac-file-link/) 里的介绍，想着在Mac上也弄个类似的。

实际操作下来，发现苹果似乎取消了 `ln [源文件夹] [目标文件夹]`，而只能用 `ln [源文件] [目标文件]`。所以写了个工具遍历源文件夹所有文件 链接到目标文件夹去。等弄完后发现，ln 链接的文件夹 居然不能同步！！！

而后发现Mac的软链接和Windows的软链接不一样，在Windows下，软链接就是快捷方式，根本没办法把实际链接到新目录去读取操作。而Mac的软链接居然能被Unity读取，同时修改可以同步！！！也就是说软链接可以达到Windows下硬链接的效果！

而Mac软链接同时支持文件或文件夹 `ln -s [源文件夹/源文件][目标文件夹/目标文件]`

：） 

所以大概苹果是想表达MacOS 的软链接已经很强大了，所以就把硬链接废了吧~

