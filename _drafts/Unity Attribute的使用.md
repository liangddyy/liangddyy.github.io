---
layout: post
title:  Unity中自带的Attribute及其使用
categories: [Unity]
description: Unity中自带的Attribute及其使用
keywords: Unity,  Attribute
---

### 菜单相关

#### PreferenceItem

使用该属性可以定制Unity的Preference界面,如Vscode插件。官方示例：

```
public class ExampleScript : MonoBehaviour
{
    // Have we loaded the prefs yet
    private static bool prefsLoaded = false;

    // The Preferences
    public static bool boolPreference = false;

    // Add preferences section named "My Preferences" to the Preferences Window
    [PreferenceItem("My Preferences")]
    public static void PreferencesGUI()
    {
        // Load the preferences
        if (!prefsLoaded)
        {
            boolPreference = EditorPrefs.GetBool("BoolPreferenceKey", false);
            prefsLoaded = true;
        }

        // Preferences GUI
        boolPreference = EditorGUILayout.Toggle("Bool Preference", boolPreference);

        // Save the preferences
        if (GUI.changed)
            EditorPrefs.SetBool("BoolPreferenceKey", boolPreference);
    }
}
```

#### AddComponentMenu

(添加Component菜单).名称(TestComponent)不能和类名相同。

```
[AddComponentMenu("TestMenu/TestComponent")]
public class ComponentMenuTest : MonoBehaviour {

}
```

#### MenuItem

##### 普通菜单项

##### Project面板右键菜单示例

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

##### Hierarchy右键菜单

```
[MenuItem("GameObject/MenuTest/MenuTest1", false, -2)]
static void RightHierarchy()
{
Debug.Log("Hierarchy右键菜单");
}
```

##### ContextMenu

可以在对应脚本的Inspector的中增加右键菜单选项。

```
[ContextMenu("菜单项")]
void DoSomething()
{
    Debug.Log("脚本上的菜单项");
}
```



### 初始化

#### InitializeOnLoad

在Class上使用，可以在Unity启动的时候，运行Editor脚本。需要该Class拥有静态的构造函数。

```
[InitializeOnLoad]
public class ScriptExecuteOrderSetting
{
    static ScriptExecuteOrderSetting()
    {
        UnityEngine.Debug.Log("ScriptExecuteOrderSetting");
    }
}
```

#### InitializeOnLoadMethod

在Method上使用，是InitializeOnLoad的Method版本。Method必须是static的。

### Editor回调相关

以下三个Callback的属性，都需要方法为static。

#### OnOpenAsset

在打开一个Asset后被调用，如打开脚本。

```
[OnOpenAsset(1)]
public static bool step1(int instanceID, int line)
{
    string name = EditorUtility.InstanceIDToObject(instanceID).name;
    Debug.Log("Open Asset step: 1, (" + name + ")");
    return false; //使用默认方式打开
}
```

#### PostProcessBuild

该属性是在build完成后，被调用的callback。同时具有多个的时候，可以指定先后顺序。

```
[PostProcessBuild(1)]
public static void OnPostprocessBuild(BuildTarget target, string pathToBuiltProject)
{
    Debug.Log(pathToBuiltProject);
}
```

#### PostProcessScene

使用该属性的函数，在scene被build之前，会被调用。具体使用方法和PostProcessBuild类似。

### 其他

#### RPC

在方法上添加该属性，可以网络通信中对该方法进行RPC调用。