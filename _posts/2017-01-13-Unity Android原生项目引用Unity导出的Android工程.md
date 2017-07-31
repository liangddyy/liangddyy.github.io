---
layout: post
title: "Unity Android原生项目引用Unity导出的Android工程"
keywords: 
description: ""
category: Unity
tags: Unity Android
---

<!--markdown-->1.    从Unity Build%Run 选择Google Android Project 导出。导出的项目为Adt下的Android项目。  

2.    在AndroidStudio项目中，在module设置中添加一个module，选择从Eclipse Adt Project导入。在module设置/Dependencies/添加导入的项目为库。

3.    将导入的项目`build.gradle` 文件里的  
`apply plugin: 'com.android.application'`  
改为:  
 `apply plugin: 'com.android.library'`  
并注释掉applicationId
`//applicationId "com.liangddyy.UToAUnity"`  
  
  
  
4.   然后在build.gradle文件里添加引用 `compile project(':uToAUnity')` 。中间的是module名。  
    此时如果提示AndroidManifest.xml 合并失败，如：  
  
            Error:Execution failed for task ':app:processDebugManifest'. > Manifest merger failed with multiple errors, see logs  
  
      可能的原因：  
  
      1. 两个项目sdk版本不相同  
  
      2. 两个项目的AndroidManifest.xml有重合的属性设置  
  
         * 如注释掉引用项目的这两行：  
  
           `android:theme="@style/UnityThemeSelector"`  
  
           `android:icon="@drawable/app_icon"`  
  
5.    可选步骤：  
  
         在引用项目中AndroidManifest.xml中注释掉：  
  
```  
<intent-filter>  
    <action android:name="android.intent.action.MAIN" />  
    <category android:name="android.intent.category.LAUNCHER" />  
    <category android:name="android.intent.category.LEANBACK_LAUNCHER" />  
</intent-filter>  
```  
否则桌面会有两个图标。一个是从原生项目中启动。一个是从引用项目也就是Unity部分启动。  
  
6.    然后就可以从原生代码跳转到 `UnityPlayerActivity `也就是Unity项目了。  
  
7.    返回键监听。结束当前UnityPlayerActivity，返回原生Activity。  
无效的方法：  
```  
@Override  
    public void onBackPressed() {  
    ...  
}  
```  
重写 onBackPressed() 测试过，没效果。需要重写 onKeyDown();  
为了方便管理，不直接跳转到UnityPlayerActivity.java。而是新建一个ExtendActivity 继承自UnityPlayerActivity.java（ExtendActivity 需要在AndroidManifest 里注册（直接写在UnityPlayerActivity也可以）。  
  
      package com.example.utoaandroidnative;  
            import android.os.Bundle;  
            import android.view.KeyEvent;  
            
            import com.unity3d.player.UnityPlayerActivity;  
            /**  
              Created by liang on 2017/1/13.  
              */  
            	  
            public class ExtendActivity extends UnityPlayerActivity {  
               @Override  
               protected void onCreate(Bundle savedInstanceState) {  
                   super.onCreate(savedInstanceState);  
               }  
               @Override  
               public boolean onKeyDown(int keyCode, KeyEvent event) {  
                   if (keyCode == KeyEvent.KEYCODE_BACK) {  
                       finish();  
                       mUnityPlayer.quit();  
                   }  
                   return super.onKeyDown(keyCode, event);  
               }  
            }  
 
  
  
![UnityToAndroid1](http://539go.com/usr/uploads/2017/01-13/UnityToAndroid01.png)  
  
跳转到Unity后按返回键可返回到上一个原生activity界面  
  
![UnityToAndroid12](http://539go.com/usr/uploads/2017/01-13/UnityToAndroid02.png)  
