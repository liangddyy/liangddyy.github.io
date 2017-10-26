---
layout: post
title: "Unity 导出Android文件并与原生项目合并遇到的问题记录"
keywords: 
description: ""
category: Unity
tags: 
---

#### 问题一

Failure to initialize! Your hardware does not support this application, sorry!

首先从自己的Android项目中添加Unity导出的工程为 module，具体可以参考：[Unity Android原生项目引用Unity导出的Android工程](http://539go.com/index.php/archives/201701/310.html)  

项目正常编译后，当跳转到Unity界面时，首先是这个报错（如下图）：Failure to initialize! Your hardware does not support this application, sorry! 

![Screenshot_2017-05-18-17-08-34-571_com.fjut.qr102](http://539go.com/usr/uploads/2017/05-20/Screenshot_2017-05-18-17-08-34-571_com.fjut.qr102.png)  

找了很久才发现原来是.so文件丢失。（大神指点的，其实仔细一想，“不支持的设备”，很明显是.so文件的问题，我以前都还遇到过类似的。自个儿还瞎折腾了半天）  

> 解决办法  

这里.so文件缺失可能有两种原因，不幸的是在不断尝试的过程中我两种都遇上了。  

- 解决1

一种是在导入Module时.so文件丢失了，解决的办法很简单。先用AndroidStudio导入项目的形式打开Unity导出的Android工程文件。将其从eclipse/adt工程转为studio工程。这时再从我们需要的项目里添加Module，此时导入时需要选import Gradle Project (通常会提示需要重命名Module）。这样导入之后.so文件不会丢失。  

- 解决2

还有一种情况（绝大多数），.so文件没有丢失，但还是报上面的错误。这时因为.so文件有多种不同的平台，而Unity导出时一般只有 armeabi-v7a和x86（jniLibs文件夹下）。解决的办法是在build.gradle（app）文件中只声明这两种平台。（build.gradle / defaultConfig）  

```  
ndk {  
    moduleName "jnilib"  
    abiFilters "armeabi-v7a", "x86"  
}  
```

如果有需要，还需要配置so文件目录（可选）。  

```  
sourceSets {  
    main {  
        jniLibs.srcDirs = ['libs']  
    }  
}  
```

示例 build.gradle（app)  

```  
apply plugin: 'com.android.application'  
android {  
    compileSdkVersion 23  
    buildToolsVersion "23.0.3"  
  
    defaultConfig {  
        applicationId "com.liangdy.aqrUnity"  
        minSdkVersion 19  
        targetSdkVersion 21  
//        multiDexEnabled true  
        versionCode 1  
        versionName "1.0"  
        ndk {  
            moduleName "jnilib"  
            abiFilters "armeabi-v7a", "x86"  
        }  
    }  
    sourceSets {  
        main {  
            jniLibs.srcDirs = ['libs']  
        }  
    }  
    ...  
```

- 解决3

2017.10.26补充：

如果aar文件里有Unity不支持的平台的.so库，也会产生这个错误。删掉armeabi-v7a和x86以外的文件夹可以解决此问题。

#### 问题二  

在Unity中引用了第三方项目，和原生项目合并后提示包名错误。切记不用重构原生项目包名。只需要改build.gradle(app)中的 applicationId 即可。  

```  
applicationId "Unity中项目的包名"  
```
