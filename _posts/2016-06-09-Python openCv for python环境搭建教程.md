---
layout: post
title: "Python openCv for python环境搭建教程"
keywords: 
description: ""
category: Python
tags: 
---

<!--markdown-->因为要完成 opencv课程的期末大作业，又不想人云亦云，所以想用 python 来做。  

基础环境：
  
Python2.7 + opencv2.4.13 + win10x64(系统虽然是64位，但是后面装插件却要装32位的，切记)  
  
安装 Python 和安装 opencv 就不多说了，网上有很多关于opencv在python下使用的教程。包括一些用pip来安装的方法，装各种插件，最后发现，有的教程真的是误人子弟。步骤其实很简单。  
  
## 1. 安装numpy  
  
[下载地址](https://pypi.python.org/pypi/numpy/)  
  
必须下载对应的python版本。可以看到，官网都是 whl 文件，如何安装这种文件，后面会说到。  
  
这里我下载的是一个 exe文件，更加便捷的安装。[Numpy-1.9.2-win32-superpack-python2.7.exe  下载地址](http://download.csdn.net/detail/corfox_liu/9155347)  
  
## 2. 添加拓展(opencv2.4.13)  
  
将 `opencv\build\python\2.7\x86\`目录下的 cv2.pyd 文件复制到 `\Python27\Lib\site-packages\`下，这里尝试用了64位，不能使用，但用32位的可以。  
  
此时，opencv便可以使用了。  
  
示例代码：  
  
```  
# _*_ coding:utf-8 _*_  
import cv2  
  
img = cv2.imread('E:\\2.jpg',cv2.IMREAD_COLOR)  
cv2.imshow("img", img)  
cv2.waitKey(100000)  
```  
  
 ![QQ截图20160609033502](http://539go.com/usr/uploads/2016/06-08/QQ截图20160609033502.png)  
  
## 附加  
  
### 安装setuptools  
  
[下载 ez_setup.py](https://pypi.python.org/pypi/setuptools) 下载Windows (simplified)版  
  
在 ez_setup.py 文件目录下，运行cmd命令 `python ez_setup.py`  
  
### 安装pip  
  
[下载get-pip.py](https://pip.pypa.io/en/latest/installing/#id7)  
  
同样运行cmd命令 `python get_pip.py`  
  
安装好后，可以在 `\Python27\Scripts\`目录下 看到pip.exe ，而此时还不能使用。需要添加 `Python27\Scripts`环境变量，如：`C:\Python27\Scripts`添加到Path。  
  
在cmd上 运行cmd命令 `pip help` 有相应的内容则表示成功了。  
  
### pip使用  
  
主要用于安装 python 拓展  
  
安装 `pip install + 文件名` 如 `pip install numpy-1.10.4+mkl-cp27-cp27m-win32.whl`  
