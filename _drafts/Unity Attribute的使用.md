---
layout: post
title: title
categories: [cate1, cate2]
description: 
keywords: 开发, 
---

AddComponentMenu(添加Component菜单)

不能放在Editor下。名称(TestComponent)不能和类名相同。

```
[AddComponentMenu("TestMenu/TestComponent")]
public class ComponentMenuTest : MonoBehaviour {

}
```

MenuItem

Project面板右键菜单示例

```
    #region 右键菜单 导出资源包（选中资源时有效）

    [MenuItem("Assets/导出Unity资源包", true)]
    static bool ExportPackageValidation()
    {
        for (var i = 0; i < Selection.objects.Length; i++)
        {
            if (AssetDatabase.GetAssetPath(Selection.objects[i]) != "")
                return true;
        }

        return false;
    }

    [MenuItem("Assets/导出Unity资源包")]
    static void ExportPackage()
    {
        var path = EditorUtility.SaveFilePanel("Save unitypackage", "", "", "unitypackage");
        if (path == "")
            return;
        UnityEngine.Debug.Log("路径：" + path);
        var assetPathNames = new string[Selection.objects.Length];
        for (var i = 0; i < assetPathNames.Length; i++)
        {
            assetPathNames[i] = AssetDatabase.GetAssetPath(Selection.objects[i]);
        }

        assetPathNames = AssetDatabase.GetDependencies(assetPathNames);

        AssetDatabase.ExportPackage(assetPathNames, path,
            ExportPackageOptions.Interactive | ExportPackageOptions.Recurse | ExportPackageOptions.IncludeDependencies);
    }

    #endregion
```

Hierarchy右键菜单

```
    [MenuItem("GameObject/MenuTest/MenuTest1", false, -2)]
    static void RightHierarchy()
    {
        Debug.Log("Hierarchy右键菜单");
    }
```

