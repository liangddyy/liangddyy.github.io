---
layout: post
title: "Unity CommandInvokationFailure Unable to resolve build tools directory解决"
keywords: 
description: ""
category: Unity
tags: 
---

<!--markdown-->```
CommandInvokationFailure: Unable to resolve build tools directory. See the Console for more details. 
C:/_green/jdk1.8\bin\java.exe -Xmx2048M -Dcom.android.sdkmanager.toolsdir="E:/SDKLocal/SDK\tools" -Dfile.encoding=UTF8 -jar "C:\Program Files\Unity5.4.3f1\Editor\Data\PlaybackEngines\AndroidPlayer/Tools\sdktools.jar" -

stderr[
Error:E:\SDKLocal\SDK\tools is not a directory. calculated from system property com.android.sdkmanager.toolsdir
]
stdout[

]
UnityEditor.Android.Command.Run (System.Diagnostics.ProcessStartInfo psi, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandInternal (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandSafe (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.HostView:OnGUI()

```

主要是最新版的tools的问题。

解决办法：

1. 重命名或者删除SDK目录下的tools文件夹
2. [http://dl-ssl.google.com/android/repository/tools_r25.2.5-windows.zip](http://dl-ssl.google.com/android/repository/tools_r25.2.5-windows.zip) 下载旧版本的tools，解压到SDK根目录。  
