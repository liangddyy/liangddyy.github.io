---
layout: post
title: Android zipalign优化
categories: [Android, Unity]
description: 
keywords: 开发, 
---

打包时版本号没核对好，又不想从新打包，所以用反编译工具修改下版本号。结果上传上去却悲剧了。

![zipalign](/Img/Android/package/zipalign.png)

公司的反编译工具应该是有对齐的。（不知道为什么Unity项目的谷歌包有问题 - -！）

于是需要手动 zipalign优化下。 zipalign工具在`SDK\build-tools`目录下。

检查apk是否Zipalign处理的命令：`zipalign -c -v 4 input.apk`，input.apk和zipalign.exe同级目录。

如：

```
C:\_File\SDK\build-tools\27.0.1\zipalign -c -v 4 input.apk
```

出现`Verification FAILED`表示没有优化。

![ziplign01](/Img/Android/package/ziplign01.png)

Zipalign优化：

```
C:\_File\SDK\build-tools\27.0.1\zipalign -v 4 input.apk output.apk
```

