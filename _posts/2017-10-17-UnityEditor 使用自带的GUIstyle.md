---
layout: post
title: UnityEditor 使用自带的GUIstyle
categories: [Unity]
description: 
keywords: 开发, Unity 
---

在Unity编辑器开发时，而GUI的各个组件基本上都有个GUIstyle的参数用于我们自定义不同的样式。可以用skin中已有的一些样式，如`GUILayout.Label("label标签",GUI.skin.label);`，也可以自行创建一个skin使用自定义不同的样式。

但是Unity自带了各种自定义好的样式，能满足很多的应用场景，但却经常被忽略。如图。

![GUIStyleView](http://539go.com\Img\Unity\Editor\GUIStyleView.png)

这些style藏在`GUI.skin.customStyles`中。写了个简单的工具方便查看和搜索这些样式。效果图如上图，代码：

```
using UnityEngine;
using UnityEditor;

public class GUIStyleViewer : EditorWindow
{
    private Vector2 scrollVector2 = Vector2.zero;
    private string search = "";

    [MenuItem("UFramework/GUIStyle查看器")]
    public static void InitWindow()
    {
        EditorWindow.GetWindow(typeof(GUIStyleViewer));
    }

    void OnGUI()
    {
        GUILayout.BeginHorizontal("HelpBox");
        GUILayout.Space(30);
        search = EditorGUILayout.TextField("", search, "SearchTextField", GUILayout.MaxWidth(position.x / 3));
        GUILayout.Label("", "SearchCancelButtonEmpty");
        GUILayout.EndHorizontal();
        scrollVector2 = GUILayout.BeginScrollView(scrollVector2);
        foreach (GUIStyle style in GUI.skin.customStyles)
        {
            if (style.name.ToLower().Contains(search.ToLower()))
            {
                DrawStyleItem(style);
            }
        }
        GUILayout.EndScrollView();
    }

    void DrawStyleItem(GUIStyle style)
    {
        GUILayout.BeginHorizontal("box");
        GUILayout.Space(40);
        EditorGUILayout.SelectableLabel(style.name);
        GUILayout.FlexibleSpace();
        EditorGUILayout.SelectableLabel(style.name, style);
        GUILayout.Space(40);
        EditorGUILayout.SelectableLabel("", style, GUILayout.Height(40), GUILayout.Width(40));
        GUILayout.Space(50);
        if (GUILayout.Button("复制到剪贴板"))
        {
            TextEditor textEditor = new TextEditor();
            textEditor.text = style.name;
            textEditor.OnFocus();
            textEditor.Copy();
        }
        GUILayout.EndHorizontal();
        GUILayout.Space(10);
    }
}
```

上面的搜索框，便是使用了"SearchTextField"，"SearchCancelButtonEmpty"两个样式。

