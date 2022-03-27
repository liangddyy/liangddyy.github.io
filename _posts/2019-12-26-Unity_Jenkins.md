---
layout: post
title: Unity Jenkins使用记录配置部分
categories: [Unity]
description: Unity自动化打包
keywords: 
---

## Java

[Java SE Development Kit 8 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

## 安装Jenkins

### 1. 下载安装

[Jenkins](https://jenkins.io/zh/)

### 2. 命令行安装（Mac推荐）

[Mac下使用命令行安装 jenkins 方法](https://www.jianshu.com/p/5b5306a32fae)

重启电脑

```jsx
jenkins start
```

[http://localhost:8080/](http://localhost:8080/login?from=%2F)

```jsx
http://localhost:8080/exit //退出Jenkins
http://localhost:8080/restart //重启
http://localhost:8080/reload //重新加载
```

## 3. 插件

---

更新gradle 配置文件 .bash_profile

source ~/.bash_profile

- AnsiColor
- Timestamp
- Unity
- XCode
- Locale (中文插件）

    ```jsx
    配置【Manage Jenkins】>【Configure System】> 【Locale】
    语言填 zh_CN
    勾选强制设置语言
    ```

## 4. 配置

archive the artifacts 匹配多个文件("," 符号隔开)

[Jenkins multiple artifacts for the same build](https://stackoverflow.com/questions/32823684/jenkins-multiple-artifacts-for-the-same-build)

示例：

```jsx
builds/${BUILD_NUMBER}/*.xml,builds/${BUILD_NUMBER}/*.zip,builds/${BUILD_NUMBER}/**/*.zip,builds/${BUILD_NUMBER}/**/*.obb,builds/${BUILD_NUMBER}/**/build/outputs/apk/debug/*.apk,builds/${BUILD_NUMBER}/**/build/outputs/apk/release/*.apk
```

Unity命令行示例

```jsx
-quit  -batchmode  -projectPath  "${WORKSPACE}" -buildTarget ${platform} -executeMethod  GameBear.SmartBuild.Internal.SmartBuilderJenkins.Build -logFile "${WORKSPACE}/builds/${BUILD_NUMBER}/unity3d_defineSymbols.log"  -silent-crashes -bbBuildPath "${WORKSPACE}/builds/${BUILD_NUMBER}" -bbVersion ${version} -bbDevelopment ${development} -bbGenerateBundle ${generateBundle} -bbKeystoreName android_release.keystore -bbKeystorePass 123456 -bbIsSplit ${isObb} -bbIsBuildBundle ${isBuildBundle} -bbBundleVersion ${bundleVersion} -bbDisableGm ${disableGm} -bbEnableTuto ${enableTuto} -bbAutoLogin ${autoLogin} -bbPingServer ${pingServer} -bbEnableMusic ${enableMusic} -bbConfigName ${configName} -bbCdnType ${cdnType} cat unity.log | grep error
```

自动删除旧的归档文件

![1](/images/Unity/Jenkins/1.png)

其他

[Command line arguments](https://docs.unity3d.com/Manual/CommandLineArguments.html)

```jsx
配置全局变量：Manage Jenkins -> Configure System -> 全局属性 -> Environment
```

Please provide an auth token with USYM_UPLOAD_AUTH_TOKEN environment variable 问题修复

[unity 命令行打包，解决报错：USYM_UPLOAD_AUTH_TOKEN environment variable_游戏_muqian328的博客-CSDN博客](https://blog.csdn.net/muqian328/article/details/102388521)

[Building a Unity game for iOS with fastlane fails with missing USYM_UPLOAD_AUTH_TOKEN](https://stackoverflow.com/questions/59637428/building-a-unity-game-for-ios-with-fastlane-fails-with-missing-usym-upload-auth)