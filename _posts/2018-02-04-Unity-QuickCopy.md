---
layout: post
title: UnityEditor 编辑器间快速复制粘贴
categories: [Unity]
description: UnityEditor 编辑器间快速复制粘贴
keywords: Unity,UnityEditor,Unity编辑器开发 
---

经常需要从Assets面板复制文件，每次都要 右键 - Show in Exlorer ，然后复制，巨麻烦。写了个工具，直接右键Asset面板在Unity不同项目间快速复制粘贴，或者直接复制到剪切板。

主要分成两种，一种是从一个编辑器到另外一个编辑器拷贝，一种是想拷贝到系统剪切板。

![menu](/images/Unity/Editor/QuickCopy/menu.png)

## 从编辑器到剪切板

Unity编辑器里没办法像C# Winform 一样直接向系统剪切板添加文件夹，只能复制文本，但是PowerShell可以，在UnityEditor里又可以执行Powershell。所以通过执行PowerShell来向系统剪切板加入要复制的文件列表。powershell使用剪切板 见 [官方文档](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-clipboard?view=powershell-5.1) 。

执行powershell

```
public static bool RunCommand(string command)
{
    using (Process process = new Process())
    {
        process.StartInfo.FileName = "powershell";
        process.StartInfo.Arguments = command;
        process.StartInfo.CreateNoWindow = true; // 不显示窗口
        process.StartInfo.ErrorDialog = true;
        process.StartInfo.UseShellExecute = false;
        try
        {
            process.Start();
            process.WaitForExit();
            process.Close();
        }
        catch (Exception e)
        {
            Debug.LogError(e);
            return false;
        }
    }
    return true;
}
```

然后加入Assets菜单，获取选中的目录拼接一串命令运行即可。

```
[MenuItem("Assets/复制 - 剪切板复制", false, 21)]
private static void CopyToClipboard()
{
    StringBuilder stringBuilder = new StringBuilder("Set-Clipboard -Path ");
    for (int i = 0; i < Selection.assetGUIDs.Length; i++)
    {
        string path = AssetDatabase.GUIDToAssetPath(Selection.assetGUIDs[i]);
        if (i != 0) stringBuilder.Append(",");
        stringBuilder.Append("\"");
        stringBuilder.Append(AssetPath2FullPath(path));
        stringBuilder.Append("\"");
    }
    RunCommand(stringBuilder.Replace("/", "\\").ToString());
}
```

注意事项：Assets路径需要转换成绝对路径。这样粘贴时才有效。

## 从编辑器到编辑器

虽然可以通过powershell可以加入文件列表到剪切板， 但是没发现通过powershell粘贴，这边比较坑爹。不过可以通过代码获取到复制的文本:

```
GUIUtility.systemCopyBuffer
```

所以采取的办法是，如果在编辑器间复制，把文件列表路径，存起来序列化后复制到文本，然后粘贴时读取系统的剪切板 然后解析下路径列表，再通过C#执行复制粘贴擦操作。

此外，有时需要通过导出导入包来复制，以便识别到依赖。道理相同，在一边导出，把列表文本写到剪切板。

### 直接复制

```
[MenuItem("Assets/复制 - 编辑器复制", false, 21)]
private static void CopyToEditor()
{
    ClipItem item = new ClipItem(ContentType.File);
    foreach (var guiD in Selection.assetGUIDs)
    {
        string path = AssetDatabase.GUIDToAssetPath(guiD);
        item.Values.Add(AssetPath2FullPath(path));
    }
    CopyClipboardItem(item);
    Debug.Log("已复制" + Selection.assetGUIDs.Length + "条数据，可在其他 Unity 编辑器里粘贴！");
}
```

序列化并复制文本

```
public static void CopyClipboardItem(ClipItem item)
{
    BinaryFormatter formatter = new BinaryFormatter();
    using (MemoryStream stream = new MemoryStream())
    {
        formatter.Serialize(stream, item);
        TextEditor te = new TextEditor {text = Convert.ToBase64String(stream.ToArray())};
        te.OnFocus();
        te.Copy();
    }
}
```

### 通过导包

和上面唯一的区别就是导出一个包到临时目录存起来

```
[MenuItem("Assets/复制 - 导出包复制", false, 21)]
private static void CopyAsPackage()
{
    string[] assetPaths = new string[Selection.assetGUIDs.Length];
    for (int i = 0; i < Selection.assetGUIDs.Length; i++)
    {
        assetPaths[i] = AssetDatabase.GUIDToAssetPath(Selection.assetGUIDs[i]);
    }
    string outPath = Path.Combine(Application.temporaryCachePath,
        Random.Range(0, 1024) + ".unitypackage");
    AssetDatabase.ExportPackage(assetPaths, outPath,
        ExportPackageOptions.Recurse | ExportPackageOptions.IncludeDependencies);
    ClipItem item = new ClipItem(ContentType.Package);
    item.Values.Add(outPath);
    CopyClipboardItem(item);
}
```

### 粘贴

```
[MenuItem("Assets/粘贴", false, 21)]
private static void Paste()
{
    ClipItem item;
    try
    {
        byte[] bytes = Convert.FromBase64String(GUIUtility.systemCopyBuffer);
        BinaryFormatter formatter = new BinaryFormatter();
        using (MemoryStream stream = new MemoryStream(bytes))
        {
            item = formatter.Deserialize(stream) as ClipItem;
        }
    }
    catch (FormatException e)
    {
        throw new FormatException("没有从剪切板解析到有效的数据!");
    }
    string assetPath = AssetDatabase.GUIDToAssetPath(Selection.assetGUIDs[0]);
    switch (item.Type)
    {
        case ContentType.File:
            CopyListFileInEditor(item.Values, AssetPath2FullPath(assetPath));
            break;
        case ContentType.Package:
            if (item.Values.Count > 0 && Path.GetExtension(item.Values[0]).Equals(".unitypackage"))
                Package2Folder.ImportPackageToFolder(item.Values[0], assetPath, true);
            break;
        default:
            break;
    }
}


public static void CopyListFileInEditor(List<string> sourcePaths, string targetPath)
{
    bool isAuto = EditorPrefs.GetBool(KeyAutoRefresh, true);
    if (isAuto) EditorPrefs.SetBool(KeyAutoRefresh, false);
    foreach (var path in sourcePaths)
    {
        string destName = Path.Combine(targetPath, Path.GetFileName(path));
        if (File.Exists(path))
        {
            File.Copy(path, destName);
        }
        else
        {
            CopyDir(path, destName);
        }
    }
    if (isAuto) EditorPrefs.SetBool(KeyAutoRefresh, true);
    AssetDatabase.Refresh();
}

public static void CopyDir(string sourcePath, string destinationPath)
{
    DirectoryInfo info = new DirectoryInfo(sourcePath);
    if (!Directory.Exists(destinationPath))
        Directory.CreateDirectory(destinationPath);
    foreach (FileSystemInfo fsi in info.GetFileSystemInfos())
    {
        string destName = Path.Combine(destinationPath, fsi.Name);

        if (fsi is FileInfo)
        {
            File.Copy(fsi.FullName, destName);
        }
        else
        {
            Directory.CreateDirectory(destName);
            CopyDir(fsi.FullName, destName);
        }
    }
}
```

需要注意的是，在编辑器里直接通过脚本粘贴之前，需要禁用掉Unity的自动刷新，否则可能没复制完线程便中断了。Unity设置里的自动刷新配置保存在 EditorPrefs，Key：kAutoRefresh。

---

以上的代码并不完整，另外还有一些细节，比如导入包时 可以导入指定Assets目录等一些细节上的没贴出来。代码在这里 [QuickCopy.cs](https://github.com/liangddyy/UnityClipboard/blob/master/UnityQuickCopyModule/UnityQuickCopyModule/QuickCopy.cs) ，直接放到项目中Editor文件夹下就能用。

## 无关紧要的部分

从一个项目复制到另一个项目，都得需要上面的代码，意味着每次都得先拷贝下这个代码才行，也相当麻烦。可以做成模块化，这样自己电脑上随便打开一个项目不用复制代码也能用了。参见之前的博客：[UnityEditor Unity的模块](https://539go.com/2017/10/20/UnityEditor-Unity%E7%9A%84%E6%A8%A1%E5%9D%97/)



为了方便自己，打包了一份。在这里 [UnityQuickCopyModule.dll](https://github.com/liangddyy/UnityClipboard/blob/master/UnityQuickCopyModule.dll) ，把该文件放到任意Unity项目，用管理员权限启动Unity，菜单栏安装即可，然后就可以全局通用了。

![menu2](/images/Unity/Editor/QuickCopy/menu2.png)



以上所有脚本和文件 [https://github.com/liangddyy/UnityClipboard](https://github.com/liangddyy/UnityClipboard)