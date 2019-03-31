---
layout: post
title: VisualStudio实现自动保存 
categories: [其他]
description: VisualStudio实现自动保存
keywords: 开发, VisualStudio, Vs, 自动保存
---


环境：VisualStudio2017 + win10

用惯了vscode的自动保存，切换回vs后没有这个功能有点不太习惯。

### 安装插件

地址：[Visual Commander](https://vlasovstudio.com/visual-commander/) 

或者直接在 `工具 - 拓展和更新 - 联机` 搜索 Visual Commander 即可。

安装后需要重启visualstudio

### 配置插件

安装好插件后，菜单栏 VCmd - Extension - Add - Language (选C#4.0) 

然后粘贴以下代码

```
public class E : VisualCommanderExt.IExtension
{
    public void SetSite(EnvDTE80.DTE2 DTE_, Microsoft.VisualStudio.Shell.Package package)
    {
        DTE = DTE_;
        System.Windows.Application.Current.Deactivated += OnDeactivated;
    }

    public void Close()
    {
        System.Windows.Application.Current.Deactivated -= OnDeactivated;
    }

    private void OnDeactivated(object sender, System.EventArgs e)
    {
        try
        {
            DTE.ExecuteCommand("File.SaveAll");
        }
        catch (System.Exception ex)
        {
        }
    }

    private EnvDTE80.DTE2 DTE;
}
```

依次 Save -  Compile - Install

> 此时实现的功能为 当 VisualStudio 失去焦点时自动保存。

### 切换标签自动保存

如果想要实现当标签页失去焦点时自动保存，可以使用下面的代码。

```
using EnvDTE;
using EnvDTE80;

public class E : VisualCommanderExt.IExtension
{
    private EnvDTE.Events events;
    private EnvDTE.WindowEvents windowEvents;
    private Microsoft.VisualStudio.Shell.Interop.IVsStatusbar statusBar;
    
    private EnvDTE80.DTE2 DTE;
    public void SetSite(EnvDTE80.DTE2 DTE_, Microsoft.VisualStudio.Shell.Package package)
    {
        DTE = DTE_;

	 events = DTE.Events;
		windowEvents = events.WindowEvents;
		windowEvents.WindowActivated += OnWindowActivated;
		System.IServiceProvider serviceProvider = package as System.IServiceProvider;
		statusBar = serviceProvider.GetService(
			typeof(Microsoft.VisualStudio.Shell.Interop.SVsStatusbar)) as 
				Microsoft.VisualStudio.Shell.Interop.IVsStatusbar;
    }

    public void Close()
    {
	 windowEvents.WindowActivated -= OnWindowActivated;
    }

    private void OnWindowActivated(EnvDTE.Window gotFocus, EnvDTE.Window lostFocus)
    {
       try
        {
            DTE.ExecuteCommand("File.SaveAll");
        }
        catch (System.Exception ex)
        {
        }
    }
}
```

如果需要实现失去焦点或切换标签时都可以自动保存，将上面的两处脚本合并成一个，然后编译安装即可。

参考：https://vlasovstudio.com/visual-commander/extensions.html#StatusBar