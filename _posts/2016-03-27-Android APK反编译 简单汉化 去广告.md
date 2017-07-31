---
layout: post
title: "Android APK反编译 简单汉化/去广告/"
keywords: 
description: ""
category: 
tags: 
---

<!--markdown--><!-- index-menu -->  
  
> 用到的一些资源【链接】  
  
1. 一款文档内容批量搜索的软件：[DocFetcher][1]  
2. 一款非常棒的编辑器：[notepad++][2]
3. 用于获取文件资源：[APKtools][3]  
4. 用于源码文件获取：dex2jar  
5. 用于源码查看：dex2jar  
5. [修改后的APKIDE少月版][4]  
  
以上最好下载最新版本来使用。  
这里我提供个包含上面四个工具的一个[集合包][5]  
  
## 1. 比较常用的反编译方法  
1.1 工具  
* apktool （获取资源文件） 
* dex2jar（获取源码文件）  
* jd-gui  （java源码查看）  
  
#### 1.2 使用apktool得到资源文件  
打开cmd.exe文件（如果直接从开始菜单启动，记得定位到编译工具所在的目录 如 `D:\Personal\Desktop\工具\` ）  
运行命令  
`java -jar apktool_2.0.3.jar d -f D:\Personal\Desktop\工具\yx.apk -o OUTPUT`  
  
其中`D:\Personal\Desktop\工具\yx.apk`是我要反编译的apk所在的路径。`OUTPUT`是输出文件夹。 
  
  
![1 命令1.png][6]  
  
得到的文件  
![2 输出文件.png][7]  
  
#### 1.2 使用dex2jar反编译apk获取Java源码  
解压apk（不要用上面的获取资源文件的文件。另外单独解压一份，用于获取源代码）  
将 classes.dex 文件拷贝到dex2jar-2.0目录  
运行cmd.exe  
执行命令 `d2j-dex2jar classes.dex` 即可得到 classes-dex2jar.jar文件  
![3 解压后的文件.png][8]  
  
***  
OUTPUT目录  
![4 运行jar编译.png][9]  
  
#### 1.3 使用jd-GUI查看源码  
打开jd-gui，将得到的classes-dex2jar.jar拖拽到jd-gui便可以查看到java源码了  
  
![源码目录.png][10]  
  
## 2. 比较便捷的反编译  
需要用到的工具：改版APKIDE（上面的链接5）  
  
使用方法非常的便捷，直接打开APK即可反编译。反编译后在软件的work目录下，便可以看到源代码。修改后，执行 编译-编译生成APK 即可重新打包。如果有错，会编译失败。  
  
.btw 这个软件还可以自己打包签名哦  
  
## 3.  汉化  
  
这里以 印象笔记 为例，反编译后的文件目录  
图01  
  
通常我们只需要修改res文件夹下的values文件夹中的文件  
字符集  
  
    values-zh		中国地区语言包  
    values			英文包  
    values-zh-rTW	中文繁体（港澳台）  
    values-zh-rCN	中文简体（仅内陆）  
  
打开相应的包修改即可。  
需要注意的是，通常values目录下不止有英文包，还包含颜色，风格，或者id等等的一些定义，通常只需要修改下面三个文件即可。  
` arrays.xml  
 strings.xml  
 plurals.xml`  
  
## 4. 去广告  
  
一个软件若要显示广告，需要先导入SDK，并在AndroidManifest.xml中注册。 
res\layout目录内的xml文件就包含有广告界面的配置代码，修改这些代码就可以去除广告界面。 
  
另外还要屏蔽广告下载源，不然只是单单不显示广告界面而已，软件还会下载广告所需的数据，耗费流量。 
Android常用的广告供应商似乎 Admob，Google Ads，百度？ 
* Admob的广告代码为： 
  
`<com.admob.android.ads.AdView android:id=”@+id/ad” 
android:layout_width=”fill_parent” 
android:layout_height=”wrap_content” />`  
  
* Google Ads的广告代码为：  
  
`<com.google.ads.GoogleAdView android:id=”@+id/adview” 
android:layout_width=”wrap_content” android:layout_height=”wrap_content” />`  
  
广告下载源 
* Admob的广告下载源： 
`http://r.admob.com/ad_source.php http://mm.admob.com 
http://api.admob.com `  
* Google Ads的广告下载源： 
`http://pagead2.googlesyndication.com/pagead/afma_load_ads.js `  
  
#### 4.1 去除广告下载源 
解包classes.dex  
用 DocFetcher 搜索广告下载源地址，DocFetcher的使用可以参照上面的 资源链接1  
  
将广告下载源地址修改成无效的地址（例如0.0.0.0、192.168.1.1等），  
  
#### 4.2 屏蔽广告  
通常广告的id都会被设置成 ads、adsview、adview等，甚至有的代码会在`AndroidManifest.xml` 文件中出现，在`AndroidManifest.xml` 中出现的广告代码删掉即可。更多的广告代码会在 res 目录下的 layout文件夹中的布局文件中出现。  
![图 01.png][11]  
  
用DocFetcher搜索/res/laout/ 下的 .xml文件即可。通常搜索的关键字 ads、adview、googleAdView、android.ads 等  
并将其对应的标签 的宽高都改为0.0dip即可，即  
`android:layout_width="0.0dip" android:layout_height="0.0dip" />`  
  
宽高都为0，等于让广告不再显示了。。。  
  
但是事事并不总尽如人意，以万能WIFI钥匙.apk为例，反编译后，搜索layout布局文件，各种广告的关键字都搜索完了，都没有找到任何内容，不得不怀疑他们的程序猿是怎么命令的。  
  
索性直接搜索ad吧。于是...  
![图02][12]  
  
命名还是很规范，`advertise` 傻子也能看出来是广告了，用notepad++或者记事本打开文件查看源码将宽高项改为0，即  
`android:layout_width="0.0dip" android:layout_height="0.0dip`  
  
同理搜索其他类似的显示广告的组件并修改。  
  
修改好后，将反编译的源码重修打包成apk即可，打包参照步骤2  
  
  
  [1]: http://www.iplaysoft.com/docfetcher.html  
  [2]: https://notepad-plus-plus.org/  
  [3]: https://bitbucket.org/iBotPeaches/apktool/downloads  
  [4]: http://www.pd521.com/thread-818-1-1.html  
  [5]: http://pan.baidu.com/s/1qXDJQ2K  
  [6]: http://539go.com/usr/uploads/2016/03/2783495707.png  
  [7]: http://539go.com/usr/uploads/2016/03/3332203919.png  
  [8]: http://539go.com/usr/uploads/2016/03/3409335289.png  
  [9]: http://539go.com/usr/uploads/2016/03/3942320602.png  
  [10]: http://539go.com/usr/uploads/2016/03/2156543565.png  
  [11]: http://539go.com/usr/uploads/2016/03/3938760831.png  
  [12]: http://539go.com/usr/uploads/2016/03/2013633368.png  
