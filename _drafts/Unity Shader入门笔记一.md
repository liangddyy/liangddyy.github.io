## Properties 属性

**Properties { Property [Property ...] }**

定义属性块，其中可包含多个属性，其定义如下：

**name ("display name", Range (min, max)) =number**

定义浮点数属性，在检视器中可通过一个标注最大最小值的滑条来修改。

**name ("display name", Color) =(number,number,number,number)**

定义颜色属性

**name ("display name", 2D) = "name" {options }**

定义2D纹理属性

**name ("display name", Rect) = "name"{ options }**

定义长方形（非2次方）纹理属性

**name ("display name", Cube) = "name"{ options }**

定义立方贴图纹理属性

**name ("display name", Float) = number**

定义浮点数属性

**name ("display name", Vector) =(number,number,number,number)**

定义一个四元素的容器(相当于Vector4)属性

#### 细节说明

- 包含在着色器中的每一个属性通过name索引（在Unity中, 通常使用下划线来开始一个着色器属性的名字）。属性会将display name显示在材质检视器中，还可以通过在‘=’符号后为每个属性提供缺省值。
- 对于Range和Float类型的属性只能是单精度值。
- 对于Color和Vector类型的属性将包含4个由括号围住的数描述。
- 对于纹理(2D, Rect, Cube) 缺省值既可以是一个空字符串也可以是某个内置的缺省纹理："white", "black", "gray" or"bump"
- 随后在着色器中，属性值通过[name]来访问。

## 通道Pass中的代码写法列举

这些代码一般是写在Pass{ }中的，细节如下：

**Color Color**

设定对象的纯色。颜色即可以是括号中的四值（RGBA），也可以是被方框包围的颜色属性名。 

**Material { Material Block }**

材质块被用于定义对象的材质属性。

**Lighting On | Off**

开启光照，也就是**定义材质块中的设定是否有效**。想要有效的话必须使用Lighting On命令开启光照，而颜色则通过Color命令直接给出。

**SeparateSpecular On | Off**

开启独立镜面反射。这个命令会添加高光光照到着色器通道的末尾，因此贴图对高光没有影响。只在光照开启时有效。

**ColorMaterial AmbientAndDiffuse | Emission**

使用每顶点的颜色替代材质中的颜色集。AmbientAndDiffuse 替代材质的阴影光和漫反射值;Emission 替代 材质中的光发射值。

###  材质块Material Block 中相关代码写法列举

如下这些代码的使用的地方是在SubShader中的一个Pass{ }中新开一个Material{ }块，在这个Material{ }块中进行这些语句的书写。这些代码包含了包含材质如何和光线产生作用的一些设置。这些属性默认为值都被设定为黑色（也就是说不产生作用），也就是说他们一般情况下可以被忽略。当然，还是有很多时候需要使用到他们的。

**Diffuse Color(R,G,B,A)**

漫反射颜色构成。这是对象的基本颜色。

**Ambient Color(R,G,B,A)**

环境色颜色构成.这是当对象被RenderSettings.中设定的环境色所照射时对象所表现的颜色。

**Specular Color(R,G,B,A)**

对象反射高光的颜色。(R,G,B,A)四个分量分别代表红绿蓝和Alpha，取值为0到1之间。

**Shininess Number**

加亮时的光泽度，在0和1之间。0的时候你会发现更大的高亮也看起来像漫反射光照，1的时候你会获得一个细微的亮斑。

**Emission Color**

自发光颜色，也就是当不被任何光照所照到时，对象的颜色。(R,G,B,A)四个分量分别代表红绿蓝和Alpha，取值为0到1之间。

而打在对象上的完整光照颜色最终是：

```
FinalColor=
	Ambient * RenderSettings ambientsetting + (Light Color * Diffuse + Light Color *Specular) + Emission
```

**翻译过来的中文式子便是：**

`最终颜色=环境光反射颜色\* 渲染设置环境设置 *（灯光颜色*漫反射颜色+灯光颜色*镜面反射颜色）+自发光`

知道了这个式子，我们就知道了，在各种光的综合作用下，我们材质最终的颜色是怎么来的了。

需要注意的是：方程式的灯光部分（也就是带括号的部分）对所有打在对象上的光线都是重复使用的。而我们在写Shader的时候常常会将漫反射和环境光光保持一致（所有内置Unity着色器都是如此）。

## 子着色器标签（SubShader Tags）

子着色器使用标签来告诉渲染引擎期望何时和如何渲染对象。其语法如下：

**Tags { "TagName1" ="Value1" "TagName2" = "Value2" }**

也就是，为标签"TagName1"指定值"Value1"。为标签"TagName2"指定值"Value2"。我们可以设定任意多的标签。

标签是标准的键值对，也就是可以根据一个键值获得对应的一个值的。SubShader 中的标签是用来决定渲染的次序和子着色器中的其他变量的。

### 决定渲染次序——队列标签（Queue tag）

- 后台（Background） - 这个渲染队列在所有队列之前被渲染，被用于渲染天空盒之类的对象。
- 几何体（Geometry，默认值）- 这个队列被用于大多数对象。 不透明的几何体使用这个队列。
- 透明（Transparent） - 这个渲染队列在几何体队列之后被渲染，采用由后到前的次序。任何采用alpha混合的对象（也就是不对深度缓冲产生写操作的着色器）应该在这里渲染（如玻璃，粒子效果等）
- 覆盖（Overlay） - 这个渲染队列被用于实现叠加效果。任何需要最后渲染的对象应该放置在此处。（如镜头光晕等）

Tags的示例

```
Shader "Transparent QueueExample"  
{  
  SubShader  
  {  
  //写上Tags标签  
     Tags {"Queue" = "Transparent" }  
                //开始一个通道  
    Pass  
    {  
             // 写Shader实体内容  
    }  
  }  
}  
```

### 自定义中间队列

在Unity实现中每一个队列都被一个整数的索引值所代表。后台为1000，几何体为2000，透明为3000，叠加层为4000. 着色器可以自定义一个队列，如：

**Tags { "Queue" ="Geometry+1" }**

## 通道（Pass）

### 通道中的名称与标签（Name and tags ）

一个通道能定义它的Name 和任意数量的Tags。通过使用tags来告诉渲染引擎在什么时候该如何渲染他们所期望的效果。语法如下：

`Tags { "TagName1" ="Value1" "TagName2" = "Value2" }`

#### 光照模式标签（LightMode tag）

LightMode 标签定义了Shader的光照模式，具体含义以后会在讲渲染管线时讲到。下面我们先简单了解一下有哪些光照模式可选，以及他们的具体作用：

- Always: 总是渲染。没有运用光照。
- ForwardBase:用于正向渲染,环境光、方向光和顶点光等
- ForwardAdd:用于正向渲染，用于设定附加的像素光，每个光照对应一个pass
- PrepassBase:用于延迟光照，渲染法线/镜面光。
- PrepassFinal:用于延迟光照，通过结合纹理，光照和自发光渲染最终颜色
- Vertex: 用于顶点光照渲染，当物体没有光照映射时，应用所有的顶点光照
- VertexLMRGBM:用于顶点光照渲染，当物体有光照映射的时候使用顶点光照渲染。在平台上光照映射是RGBM 编码
- VertexLM:用于顶点光照渲染，当物体有光照映射的时候使用顶点光照渲染。在平台上光照映射是double-LDR 编码（移动平台，及老式台式CPU）
- ShadowCaster: 使物体投射阴影。
- ShadowCollector: 为正向渲染对象的路径，将对象的阴影收集到屏幕空间缓冲区中。

#### 条件选项标签 （RequireOptions tag ）

### 关于渲染设置 （Render Setup ）

通道设定显示硬件的各种状态，例如能打开alpha混合，能使用雾，等等。这些命令如下：

**Material { Material Block }**

定义一个使用顶点光照管线的材质,详情参考上次我们讲的Material

**Lighting On | Off**

开启或关闭顶点光照。开启灯光之后，顶点光照才会有作用

**Cull Back | Front | Off**

设置多边形剔除模式，详细内容后面的文章会讲解到。

**ZTest (Less | Greater | LEqual | GEqual |Equal | NotEqual | Always)**

设置深度测试模式，详细内容后面的文章会讲解到。

**ZWrite On | Off**

设置深度写模式，详细内容后面的文章会讲解到。

**Fog { Fog Block }**

设置雾参数，详细内容后面的文章会讲解到。

**AlphaTest (Less | Greater | LEqual | GEqual| Equal | NotEqual | Always) CutoffValue**

开启alpha测试

**Blend SourceBlendMode |DestBlendMode**

设置alpha混合模式

**Color Color value**

设置当顶点光照关闭时所使用的颜色

**ColorMask RGB | A | 0 | any combination of R, G, B, A**

设置颜色写遮罩。设置为0将关闭所有颜色通道的渲染

**Offset OffsetFactor , OffsetUnits**

设置深度偏移

**SeparateSpecular On | Off**

开启或关闭顶点光照相关的平行高光颜色。

**ColorMaterial AmbientAndDiffuse | Emission**

当计算顶点光照时使用每顶点的颜色

### 关于纹理设置（Texture Setup ）

在完成渲染设定后，我们可以指定一定数量的纹理和当使用 SetTexture 命令时所采用的混合模式：

**SetTexture [texture property]{ [Combineoptions] }**

纹理设置，用于配置固定函数多纹理管线，当自定义fragment shaders 被使用时，这个设置也就被忽略掉了。

### 高端特效的通道命令

#### UsePass——包含已经写好的通道

UsePass 可以包含来自其他着色器的通道，来减少重复的代码。

例如，在许多像素光照着色器中，阴影色或顶点光照通道在在相应的顶点光照着色器中是相同的。UsePass命令只是包含了另一个着色器的给定通道。例如如下的命令可以使用内置的高光着色器中的名叫"Base"的通道：

**UsePass "Specular/BASE"**

而为了让UsePass能够认识到指定的是谁，必须给希望使用的通道命名，弄个身份证。通道中的Name命令就是这个功能：

**Name "MyPassName"**

#### GrabPass——捕获屏幕内容到纹理中

GrabPass 可以捕获物体所在位置的屏幕的内容并写入到一个纹理中，通常在靠后的通道中使用，这个纹理能被用于后续的通道中完成一些高级图像特效。

一个示例如下：

```
Shader"Shader编程/1.基础单色"
{
	SubShader{

	//在所有不透明几何体之后绘制  
       Tags { "Queue" = "Transparent" }  
   
	//捕获对象后的屏幕到_GrabTexture中  
       GrabPass { }  
   
       //用前面捕获的纹理渲染对象，并反相它的颜色  
       Pass{  
           SetTexture [_GrabTexture] { combine one-texture }  
       }  
	}
}
```

## 纹理（Texturing）

### 纹理合并命令


combine src1 * src2
将源1和源2的元素相乘。结果会比单独输出任何一个都要暗

combine src1 + src2
将将源1和源2的元素相加。结果会比单独输出任何一个都要亮

combine src1 - src2
源1 减去 源2

combine src1 +- src2
先相加，然后减去0.5（也就是添加了一个符号）

combine src1 lerp (src2) src3
使用源2的透明度通道值在源3和源1中进行差值，注意差值是反向的：当透明度值是1是使用源1，透明度为0时使用源3

combine src1 * src2 + src3
源1和源2的透明度相乘，然后加上源3

combine src1 * src2 +- src3
源1和源2的透明度相乘，然后和源3做符号加

combine src1 * src2 - src3
源1和源2的透明度相乘，然后和源3相减


其中，所有src属性都可以是previous,constant, primary or texture其中的一个。

Previous 是上一次SetTexture的结果
Primary 是来自光照计算的颜色或是当它绑定时的顶点颜色
Texture是在SetTexture中被定义的纹理的颜色
Constant是被ConstantColor定义的颜色

一些小技巧：

1.上述的公式都均能通过关键字 Double 或是 Quad 将最终颜色调高亮度2倍或4倍。
2.所有的src属性，除了差值参数都能被标记一个“-”负号来使最终颜色反相。
3.所有src属性能通过跟随 alpha 标签来表示只取用alpha通道。

## 剔除与深度测试（Culling & DepthTesting）语法

语句之一：Cull Back | Front| Off
此语句用于控制几何体的哪一面会被剔除（也就是不被绘制） 。其中：

Cull Back—— 不绘制背离观察者的几何面
Cull Front—— 不绘制面向观察者的几何面，用于由内自外的旋转对象
Cull Off —— 显示所有面。用于特殊效果。

语句之二：ZWrite On | Off
此语句用于控制是否将来之对象的像素写入深度缓冲（默认开启），如果需要绘制纯色物体，便将此项打开，也就是写上ZWrite On。如果要绘制半透明效果，关闭深度缓冲，则用ZWrite Off。

语句之三：ZTest Less |Greater | LEqual | GEqual | Equal | NotEqual | Always
此语句用于控制深度测试如何执行。
缺省值是LEqual （绘制和存在的对象一致或是在其中的对象；隐藏其背后的对象），含义列举对应如下：

```
Greater
只渲染大于AlphaValue值的像素
GEqual
只渲染大于等于AlphaValue值的像素
Less
只渲染小于AlphaValue值的像素
LEqual
只渲染小于等于AlphaValue值的像素
Equal
只渲染等于AlphaValue值的像素
NotEqual
只渲染不等于AlphaValue值的像素
Always
渲染所有像素，等于关闭透明度测试。等于用AlphaTest Off
Never
不渲染任何像素
```

语句之四：Offset Factor ,Units
此语句用两个参数（Facto和Units）来定义深度偏移。
Factor参数表示 Z缩放的最大斜率的值。
Units参数表示可分辨的最小深度缓冲区的值。
于是，我们就可以强制使位于同一位置上的两个集合体中的一个几何体绘制在另一个的上层。例如偏移量Offset 设为0, -1（即Factor=0, Units=-1）的值使得靠近摄像机的几何体忽略几何体的斜率，而偏移量为-1,-1（即Factor =-1, Units=-1）时，则会让几何体偏移一个微小的角度，让观察使看起来更近些。

## Alpha测试

语句之一：AlphaTest Off                 
此语句用于渲染所有像素（缺省）

语句之二：AlphaTest comparison AlphaValue

此语句用于设定透明度测试只渲染在某一确定范围内的透明度值的像素。其中的comparison取值为下表之一：

```
Greater
Only render pixels whose alpha is greater than AlphaValue. 大于
GEqual
Only render pixels whose alpha is greater than or equal to AlphaValue. 大于等于
Less
Only render pixels whose alpha value is less than AlphaValue. 小于
LEqual
Only render pixels whose alpha value is less than or equal to from AlphaValue. 小于等于
Equal
Only render pixels whose alpha value equals AlphaValue. 等于
NotEqual
Only render pixels whose alpha value differs from AlphaValue. 不等于
Always
Render all pixels. This is functionally equivalent to AlphaTest Off. 
渲染所有像素，等于关闭透明度测试AlphaTest Off
Never
Don't render any pixels. 不渲染任何像素
而AlphaValue为一个范围在0到1之间的浮点值。也可以是一个指向浮点属性或是一个范围属性，在后一种情况下需要使用标准的方括号写法标注索引名字，如([变量名]).
```

