---
layout: post
title: UnityEditor Unity的模块
categories: [Unity]
description: 
keywords: 开发, 
---

有时写了点编辑器工具，希望在每个项目中都用到，但是又不希望每次都把脚本拷贝一下。这时，就可以用到这个功能，让脚本成为Unity的“一部分”，任何项目可用。

比如平时用VsCode作为Unity的编辑器时，经常需要用到VsCode这个插件，下面以Vscode为例，让Unity打开所有项目时，都自带VsCode插件。

### Step1. 

在Unity的安装目录 `Unity5.4.3\Editor\Data\UnityExtensions\Unity` (我的版本是5.4.3)下，有一些Unity自带的模块，比如 Networking、GUISystem等，所有的模块根目录都有一个`ivy.xml`文件。如：

```
<?xml version="1.0" encoding="utf-8"?>
<ivy-module version="2.0">
  <info version="5.4.3" organisation="Unity" module="UNetHLAPI" e:packageType="UnityExtension" e:unityVersion="5.4.3f1" xmlns:e="http://ant.apache.org/ivy/extra" />
  <publications xmlns:e="http://ant.apache.org/ivy/extra">
    <artifact name="UnityEngine.Networking" type="dll" ext="dll" e:guid="870353891bb340e2b2a9c8707e7419ba" />
    <artifact name="Editor/UnityEditor.Networking" type="dll" ext="dll" e:guid="5f32cd94baa94578a686d4b9d6b660f7" />
  </publications>
</ivy-module>
```

其中：version、unityVersion均为当前Unity的版本号；artifact name 标签为我们的DLL文件的相对路径。这两项是需要我们修改的(在Step3)。

### Step2. 生成dll

打开Visual Studio 新建一个Dll类库。

文件 - 新建 - 项目 - Visual C# - 类库(.NET Framework) 

![dll](\Img\Unity\Editor\module\dll.png)

- 这里需要将目标框架设置为framework3.5。（项目属性 - 应用程序 - 目标框架 - framework3.5 。），否则可能报下面的错误

```
TypeLoadException: Could not load type 'System.Runtime.Versioning.TargetFrameworkAttribute' from assembly '...
```

由于我们需要用到Unity的一些Api，所以还需要为Vs项目引用两个Dll，UnityEngine.dll和UnityEditor.dll。

![添加引用](\Img\Unity\Editor\module\添加引用.png)

这两个引用在`Unity5.4.3\Editor\Data\Managed\` 目录下，当然，也可以引用其他的dll文件。

将Unity插件VsCode的脚本拷贝到这个项目中，并生成一下解决方案，可以得到我们需要的dll了。

![生成dll](\Img\Unity\Editor\module\生成dll.png)

### Step3. 打包模块使用

从Unity自带的模块中拷贝一份 ivy.xml 文件放置好。VSCodeModule文件夹下ivy.xml文件、并且将生成的

VSCodeModule.dll、VSCodeModule.pdb 文件放置到Editor下（如果是非Editor脚本则放在根目录）。

> 这里除了要拷贝主要的VSVSCodeModule.dll，如果有引用其他dll库，也需要一并复制进来。Unity已有的库则不用再拷贝（如UnityEditor.dll...等）。

![結構](\Img\Unity\Editor\module\結構.png)

然后修改ivy.xml文件，如：

```
<?xml version="1.0" encoding="utf-8"?>
<ivy-module version="2.0">
  <info version="5.4.3" organisation="Unity" module="VSCodeModule" e:packageType="UnityExtension" e:unityVersion="5.4.3" xmlns:e="http://ant.apache.org/ivy/extra" />
  <publications xmlns:e="http://ant.apache.org/ivy/extra">
    <artifact name="Editor/VSCodeModule" type="dll" ext="dll" e:guid="80a3616ca19596e4da0f10f14d241e9f" />
  </publications>
</ivy-module>
```

修改了 `version="5.4.3"` `e:unityVersion="5.4.3"` `module="VSCodeModule"` `artifact name="Editor/VSCodeModule"` 主要是Unity的版本号，模块名称和dll位置。

然后将 VSCodeModule 文件夹复制到 `\Editor\Data\UnityExtensions\Unity\`下。新建一个项目，空空如也，依然可用vscode插件。

![vscode](\Img\Unity\Editor\module\vscode.png)

> 参考：[http://homans.nhlrebel.com/2014/12/09/unity-4-6-and-modules/](http://homans.nhlrebel.com/2014/12/09/unity-4-6-and-modules/)
>
> Unity VsCode插件：[https://github.com/dotBunny/VSCode](https://github.com/dotBunny/VSCode)
>
> 本文示例工程：[https://github.com/liangddyy/UnityDemos/tree/master/VSCodeModule](https://github.com/liangddyy/UnityDemos/tree/master/VSCodeModule)
