---
layout: post
title: 活用C#中的析构函数回收脚本
categories: [Unity, C#]
description: 
keywords: 
---

活用C#中的析构函数回收脚本

公司旧项目里有一个庞大的类，使用的地方还特别多，有的页面甚至一进去就new几百次，导致大量的gc。

一进某个页面的时候是这样子的 (旁边的大粗线就是了)   T_T

![resurrection01](/images/Unity/scripts/resurrection01.png)

![resurrection02](/images/Unity/scripts/resurrection02.png)



于是乎尝试优化下。

主要两个方向：

1. 增加缓存池（减少new的使用）。
2. 减轻类结构。

大概看了下相关代码，数据类的代码实在有点多，而且有点错综复杂，加上还有个服务端用的也很庞大的数据类。同时，改数据类的关联结构肯定牵一发动全身，所以先考虑增加缓存池看看情况。

于是把所有的原来的 `new ShipDesignVO(WarShipPlanningInfo wspInfo)` （WarShipPlanningInfo 就是那个很大的服务端用的数据类了 T_T） 换成了 

```
public static ShipDesignVO Get(WarShipPlanningInfo wspInfo)
{
	return StackObjectPool<ShipDesignVO>.Get().Init(wspInfo);
}
```

好在创建的地方并不多，7、8处的样子。

然后在所有覆盖或销毁 `ShipDesignVO` 的地方，使用 `StackObjectPool<ShipDesignVO>.Push();`

但是此时有个问题，虽然创建的地方不多，但是使用创建该类脚本的地方却有几百个，同时Unity里脚本还可能随物体销毁，不可能挨个检查过去。

最后发现用析构函数可以解决。

在主要的地方仍然手动push到缓存池，对于遗漏的脚本，在GC延迟回收时，会触发析构函数，此时用`GC.ReRegisterForFinalize` 方法重新注册，并重新推入缓存池，最终完美解决。

```
public class ShipDesignVO : IStackObject
{
	~ShipDesignVO()
	{
		GC.ReRegisterForFinalize(this);
		StackObjectPool<ShipDesignVO>.Push(this);
	}
}
```

然后

![03](/images/Unity/scripts/03.png)

T_T变细变矮了~

![04](/images/Unity/scripts/04.png)



最后，`GC.ReRegisterForFinalize` 该方法请慎用，小心造成内存泄漏。



附上老东家那边同事给的参考链接~

https://stackoverflow.com/questions/22155347/in-c-sharp-can-i-stop-an-object-from-being-garbage-collected-from-the-finalizer?tdsourcetag=s_pctim_aiomsg

https://stackoverflow.com/questions/3680281/usages-of-object-resurrection