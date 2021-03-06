---
layout: post
title: Unity中的曲线插值CatmullRom
categories: [Unity, 算法]
description: 
keywords: 开发, Unity, 插值, 算法
---

之前写了个[动画插件](https://github.com/liangddyy/TweenUtil)，有个需要曲线插值的功能。给定一些点的位置，物体成一条平滑曲线依次通过这些点。

Bezier曲线是在Unity里比较常用的,但是不适合这里的需求。因为Bezier无法通过所有的点,它需要有另外的点来构造切线。如下图：

图1

![Bezeir](\Img\Unity\Spline\Bezeir.png)

查了资料发现CatmullRom曲线可以平滑的通过所有点，满足需求，如下图：

图2

![CatmullRom](\Img\Unity\Spline\CatmullRom.png)

可以看这里的[介绍](https://en.wikipedia.org/wiki/Centripetal_Catmull%E2%80%93Rom_spline)。

在Unity中C#脚本的实现：


```
/// <summary>
/// Catmull-Rom 曲线插值
/// </summary>
/// <param name="p0"></param>
/// <param name="p1"></param>
/// <param name="p2"></param>
/// <param name="p3"></param>
/// <param name="t">0-1</param>
/// <returns></returns>
public static Vector3 CatmullRomPoint(Vector3 p0, Vector3 p1, Vector3 p2, Vector3 p3, float t)
{
    return p1 + (0.5f * (p2 - p0) * t) + 0.5f * (2f * p0 - 5f * p1 + 4f * p2 - p3) * t * t +
            0.5f * (-p0 + 3f * p1 - 3f * p2 + p3) * t * t * t;
}
```

需要注意的是，接受参数p0-p3四个点后，所得的曲线为p1和p2之间的。所以要得到多个点构造的曲线，首位部分需要特殊处理下，比如：`CatmullRomPoint(p0,p0,p1,p2,t);` 这样可以得到p0和p1之间的曲线。

在Unity中绘制出来：

![CatmullRom-Unity](\Img\Unity\Spline\CatmullRom-Unity.png)

