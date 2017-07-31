---
layout: post
title: Unity VsCode与Unity完美结合
keywords: 
description: ""
category: Unity
tags: Unity VsCode
---

​	之前开发都是在vs上装了插件resharper，以此来保持用Androidstudio上的一些操作习惯和功能。由于最近又用起了Mac，Mac上的vs又没有resharper，所以只好自己折腾一个自己喜欢的开发环境。

​	比较重视的几点：

* 重构方便
* 查看文档、上下文引用便捷​

## 插件/拓展

* Debugger for Unity

* Unity3d-pack
  * C# 

    对 C# 代码和工程提供全面支持。

  * C# FixFormat

  * C# Snippets

  * C# XML Documentation Comments

  * Debugger for Unity

  * Shader languages support

  * Unity Snippets Modified

  * Unity Tools 用来查看Unity文档等

* UnitySnippets

  可以快速创建 Start、Update 等常用方法的片段。

* MonoDebug

* ESLint

* IntelliJ IDEA Keybindings

* Material Icon Theme

* Visual Studio Dark Theme

* vscode-icon

* Document This  强大的注释工具

## 插件/补充

* Git History

  使用. ctrl + shift + p 快捷工具中输入 `view history`

* Setting Sync 同步配置到Github

  alt + shift + U	上传配置（键入：sync update）
  alt + shift + D 	下载配置

  vscode  同步gist ID：56fb89fc501808e410da4d8fac402adb

   ​

## 一些设置

首选项 - 设置

```
    "editor.fontSize": 16,
    "workbench.iconTheme": "vscode-icons",

    // 控制已更新文件的自动保存。
    // 接受的值:“off”、"afterDelay”、
    // “onFocusChange”(编辑器失去焦点)、“onWindowChange”(窗口失去焦点)。如果设置为“afterDelay”，则可在 "files.autoSaveDelay" 中配置延迟。
    "files.autoSave": "onFocusChange",
    
    // 控制在多少毫秒后自动保存更改过的文件。仅在“files.autoSave”设置为“afterDelay”时适用。
    "files.autoSaveDelay": 500,

    // 控制键入时是否应自动显示建议
    "editor.quickSuggestions": {
        "other": true,
        "comments": false,
        "strings": true
    },

    // 启用时，会在打开文件时尝试猜测字符集编码
    "files.autoGuessEncoding": true,

    // 控制折行方式。可以选择： - “off” （禁用折行），
    // - “on” （视区折行）， - “wordWrapColumn”（在“editor.wordWrapColumn”处折行）或 - “bounded”（在视区与“editor.wordWrapColumn”两者的较小者处折行）。
    "editor.wordWrap": "on"
```

## 一些快捷键

快捷键修改：首选项 - 键盘快捷方式（比如将『editor.action.format』的快接机方式改成『Ctrl + alt + f 』，这个是代码格式化）

以下的快捷键均是映射了『IntelliJ IDEA Keybindings』之后的。

* ctrl + shift + p	键入。快捷操作 
  * 比如搜索『文件图标主题』，即可将图标改成上面下载的插件『vscode-icon』或者『Material Icon Theme』
* shift + f6 重命名

错误提示

`The .NET CLI tools cannot be located. .NET Core debugging will not be enabled. Make sure .NET CLI tools are installed and are on the path.`

