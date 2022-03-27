---
layout: post
title: Unity的一个小BUG
categories: [Unity]
description: some word here
keywords: UnityEditor, UnityBUG
---

一个Hieraychy面板的右键菜单的验证问题。如下：

```
    [MenuItem("GameObject/示例-Hierarchy", false, 12)]
    private static void HierarchyMenu()
    {
        Debug.Log(Selection.activeGameObject.name);
    }

    [MenuItem("GameObject/示例-Hierarchy", true, 12)]
    private static bool CopyValide()
    {
        return Selection.activeGameObject != null;
    }
```

添加一个菜单，第二个是验证方法。当我没选择 Hierarchy 面板里任何物体时，菜单栏的 GameObject/示例-Hierarchy 菜单应该是不可点击状态。事实也是如此。

但是当我直接在 Hierarchy 面板 中右键的上下文菜单时，无论有没有选中物体，`示例-Hierarchy`菜单 始终是可以点击状态。但是当没有选中物体 也就是 `CopyValide()` 为FALSE 时，点击菜单后却没有执行 本来的 `HierarchyMenu()` 方法；有选中物体时，正常执行。

也就是说验证方法 `CopyValide()` 被用来 在点击后才去调用了。

![菜单-未选](/images/Unity/Editor/HierarchyBUG/菜单-未选.png)

![菜单-未选](/images/Unity/Editor/HierarchyBUG/菜单-未选.png)

![上下文-未选](/images/Unity/Editor/HierarchyBUG/上下文-未选.png)

![上下文-已选](/images/Unity/Editor/HierarchyBUG/上下文-已选.png)

当然，由于验证函数还是执行了，虽然在上下文菜单中没有起作用，但是点击之后仍然有调用。当不满足条件时，是不会执行对应的方法的。这个问题尽管不会导致出错，但也不正确。