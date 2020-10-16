---
layout: post
title: "[Unity Shader] ShaderLab Syntax"
data: 2020-10-16 12:00:00 +0800
category: "Unity Shader"
---
Unity中所有的着色器都是用ShaderLab语法编写的。实际着色器的代码位于`CGPROGRAM`代码片段中。

用ShaderLab语法编写的Shader结构
```shaderlab
Shader "name" { [Properties] Subshaders [Fallback] [CustomEditor] }
```
- Shader 着色器文件的根命令。每个文件必须定义一个（且仅一个）Shader。
- Properties 显示在材质中的属性列表
- Subshaders 子着色器列表
- Fallback 着色器回退
- CustomEditor 自定义编辑器

## 属性
语法
```shaderlab
Properties { Property [Property ...]}
```
单个Property的语法
```shaderlab
[attributes] _name ("display name", type) = default value
```
- 着色器中的每个属性均通过name引用，在Unity中通常以下划线开头。
    - Unity将_MainTex的纹理视为主纹理
    - Unity将_Color的颜色视为主颜色
- display name用于显示在材质面板中。
- type为属性的类型，支持以下类型
    - Range(min, max) 材质面板中显示为滑动条
    - Float 浮点类型
    - Int 整型
    - Color 颜色 4D矢量
    - Vector 4D矢量
    - 2D 2D纹理
    - Cube Cube纹理
    - 3D 3D纹理
- default value默认值
    - Range, Float, Int都是单个数字
    - Color和Vector 例如(1, 0.5, 0.2, 1)
    - 2D纹理 例如："white"
    - Cube，3D 默认为空字符串，如果未指定使用灰色(0.5, 0.5, 0.5, 0.5)
- attributes Unity支持的特性
    - [HideInInspector] 材质中不显示改属性
    - [NoScaleOffset] 贴图属性中不显示缩放和偏移
    - [Normal] 声明是法线贴图
    - [HDR] 表示纹理属性需要高动态范围 (HDR) 纹理。
    - [Gamma] 表示在 UI 中将浮点/矢量属性指定为 sRGB 值
    - [PerRendererData] 表示纹理属性将以 MaterialPropertyBlock 的形式来自每渲染器数据。
    - [MainTexture] - 表示是材质的主纹理。
    - [MainColor] - 表示是材质的主色。

可以通过实现自定义的MaterialPropertyDrawer类，控制属性在材质中的呈现方式。

## 子着色器
每个着色器必须至少包含一个子着色器，加载着色器时，Unity会使用设备支持的第一个子着色器。如果不支持任何子着色器，Unity将尝试使用回退着色器。

语法：
```shaderlab
Subshader { [Tags] [CommonState] Passdef [Passdef ...]}
```
### Tags 子着色器标签(SubShader Tags)
标签是一种键值对，可以使用Unity的内置标签，也可以自定义标签，在代码中通过Material.GetTag函数来获取标签。
语法：
```shaderlab
Tags { "TagName1" = "Value1" "TagName2" = "Value2" }
```
- `Queue` 标签。用于确定对象的绘制顺序，预定义队列包括
    - `Background` 1000 此渲染队列在任何其他渲染队列之前渲染。通常会对需要处于背景中的对象使用此渲染队列。
    - `Geometry` 2000 （默认值）此队列用于大部分对象。不透明几何体使用此队列。
    - `AlphaTest` 2450 进行 Alpha 测试的几何体将使用此队列。这是不同于 Geometry 队列的单独队列，因为在绘制完所有实体对象之后再渲染经过 Alpha 测试的对象会更有效。
    - `Transparent` 3000 此渲染队列在 Geometry 和 AlphaTest 之后渲染，按照从后到前的顺序。任何经过 Alpha 混合者（即不写入深度缓冲区的着色器）都应该放在这里（玻璃、粒子效果）。
    - `Overlay` 4000 此渲染队列旨在获得覆盖效果。最后渲染的任何内容都应该放在此处（例如，镜头光晕）。
- `RenderType` 标签将着色器分为几个预定义的组。
    - `Opaque` 大部分着色器（法线、自发光、反射和地形着色器）。
    - `Transparent` 大部分半透明着色器（透明、粒子、字体和地形附加通道着色器）。
    - `TransparentCutout` 遮罩透明度着色器（透明镂空、两个通道植被着色器）。
    - `Background` 天空盒着色器。
    - `Overlay` 光环、光晕着色器。
    - `TreeOpaque` 地形引擎树皮。
    - `TreeTransparentCutout` 地形引擎树叶。
    - `TreeBillboard` 地形引擎公告牌树。
    - `Grass` 地形引擎草。
    - `GrassBillboard` 地形引擎公告牌草。
- `DisableBatching` 标签
    - true 始终对此着色器禁用批处理
    - false 不禁用批处理；这是默认值
    - LODFading（当 LOD 淡化处于激活状态时禁用批处理；主要用于树）
- `ForceNoShadowCasting` 标签 值为true时，渲染的对象强制禁用投射阴影。
- `IgnoreProjector` 标签 值为“True”时，使用此着色器的对象不会受到投影器的影响。
- `CanUseSpriteAtlas` 标签 用于Sprite对象。
- `PreviewType` 标签 材质中预览的类型。

### Pass
当 Unity 选择要用于渲染的子着色器时，它会为每个定义的通道 (Pass) 渲染一次对象（并且数量可能由于光交互而增加）。由于对象的每次渲染成本都很高，因此应以尽可能少的通道数量定义着色器。当然，有时在某些图形硬件上，所需的效果不能在单个通道中完成；那么您别无选择，只能使用多个通道。

语法：
```shaderlab
Pass { [Name and Tags] [RenderSetup] }
```
- `Name and Tags` 一个通道 (Pass) 可以定义一个名称 (Name) 和任意数量的标签 (Tags)。这些名称/值字符串用于将通道的意图传达给渲染引擎。
    - `LightMode` 标签 定义通道在光照管线中的角色。
        - Always：始终渲染；不应用光照。
        - ForwardBase：在前向渲染中使用；应用环境光、主方向光、顶点/SH 光源和光照贴图。
        - ForwardAdd：在前向渲染中使用；应用附加的每像素光源（每个光源有一个通道）。
        - Deferred：在延迟渲染中使用；渲染 G 缓冲区。
        - ShadowCaster：将对象深度渲染到阴影贴图或深度纹理中。
        - MotionVectors：用于计算每对象运动矢量。
        - PrepassBase：在旧版延迟光照中使用；渲染法线和镜面反射指数。
        - PrepassFinal：在旧版延迟光照中使用；通过组合纹理、光照和反光来渲染最终颜色。
        - Vertex：当对象不进行光照贴图时在旧版顶点光照渲染中使用；应用所有顶点光源。
        - VertexLMRGBM：当对象不进行光照贴图时在旧版顶点光照渲染中使用；在光照贴图为 RGBM 编码的平台上（PC 和游戏主机）。
        - VertexLM：当对象不进行光照贴图时在旧版顶点光照渲染中使用；在光照贴图为双 LDR 编码的平台上（移动平台）。
    - `PassFlags`标签 一个通道可指示一些标志来更改渲染管线向通道传递数据的方式。这可通过使用 PassFlags 标签来实现，该标签的值为空格分隔的标志名称。
    - `RequireOptions` 标签 一个通道可指示仅当满足某些外部条件时才渲染该通道。这可通过使用 RequireOptions 标签来实现，该标签的值为空格分隔的选项字符串。
- `RenderSetup` 渲染状态设置:
    - `Cull Back | Front | Off` 设置多边形剔除模式。
        - `Back` 不渲染背离观察者的多边形（默认值），即剔除背面多边形。
        - `Front` 不渲染面向观察者的多边形。用于从里到外翻转对象。
        - `Off` 禁用剔除 绘制所有面。用于特殊效果。
    - `ZTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always)` 设置深度缓冲区测试模式。默认值为 LEqual（隐藏其后面的对象绘制）。
    - `ZWrite On | Off` 控制是否将此对象的像素写入深度缓冲区（默认值为On）。
        - 如果要绘制实体对象，请将其保留为 On。
        - 如果要绘制半透明效果，请切换到 Off。
    - Offset OffsetFactor, OffsetUnits 设置 Z 缓冲区深度偏移。
    - `Blend` 设置 Alpha 混合、Alpha 操作和 alpha-to-coverage 模式。
        - Blend Off：关闭混合（这是默认值）
        - Blend SrcFactor DstFactor：配置并启用混合。生成的颜色将乘以 SrcFactor。屏幕上的已有颜色乘以 DstFactor，然后将这两个值相加。
        - Blend SrcFactor DstFactor, SrcFactorA DstFactorA：同上，但使用不同系数来混合 Alpha 通道。
        - BlendOp Op：不将混合颜色相加，而是对它们执行不同的操作。
        - BlendOp OpColor, OpAlpha：同上，但是对颜色 (RGB) 通道和 Alpha (A) 通道使用不同的混合操作。
    - ColorMask RGB | A | 0 | R、G、B、A 的任意组合 设置颜色通道写入遮罩。
- 旧版固定函数着色器命令 一些命令用于编写旧版“固定函数样式”着色器。这是视为已弃用的功能，因为编写表面着色器或着色器程序 可带来更大的灵活性。但是，对于非常简单的着色器，以固定函数样式编写着色器有时会更容易，因此这里提供了命令。请注意，如果不使用固定函数着色器，则会忽略以下所有命令。
    - Lighting On | Off 顶点光照 
    - Material { Material Block } 材质颜色
    - SeparateSpecular On | Off 镜面高光
    - Color Color-value 默认颜色（如果顶点光照关闭）
    - ColorMaterial AmbientAndDiffuse | Emission
    - Fog { Fog Block } 设置固定函数 Fog 的参数。
    - AlphaTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always) CutoffValue 开启固定函数 Alpha 测试。
    - SetTexture textureProperty 固定函数纹理组合器
    - Stencil 模板缓冲区可用作一般目的的每像素遮罩，以便保存或丢弃像素。
## UsePass
UsePass 命令使用来自另一个着色器的指定通道。插入来自给定着色器的具有给定名称的所有通道。Shader/Name 包含着色器名称和通道名称，以斜杠字符分隔。请注意，系统只会考虑第一个受支持的子着色器。
语法
```shaderlab
UsePass "Shader/Name"
```
## GrabPass
GrabPass 是一种特殊通道类型，它把即将绘制对象时的屏幕内容抓取到纹理中。在后续通道中即可使用此纹理，从而执行基于图像的高级效果。
简单的 GrabPass { } 可将当前屏幕内容抓取到某个纹理中。在随后的通道中可通过 `_GrabTexture` 名称访问该纹理。注意：这种抓取通道的形式将为使用它的每个对象执行耗时的屏幕抓取操作。
GrabPass { "TextureName" } 可将当前屏幕内容抓取到纹理中，但仅为使用给定纹理名称的第一个对象在每一帧执行一次该操作。在后续通道中可通过给定纹理名称访问该纹理。场景中有多个对象在使用 GrabPass 时，这种方法更高效。

## 回退 Fallback
在所有子着色器之后定义Fallback，表明如果没有任何子着色器能够在此硬件上运行，则尝试使用另一个着色器中的子着色器。
Fallback 语句的效果等同于插入其他着色器中的所有子着色器。

语法：
回退到名为name的着色器
```shaderlab
Fallback "name"
```
或
显式说明即使没有子着色器可以在此硬件上运行，也不会进行回退并且不会显示警告。
```shaderlab
Fallback Off
```

## 自定义编辑器 CustomEditor
可为着色器定义一个 CustomEditor。 Unity会查找该名字并继承自ShaderGUI的类。所有使用此着色器的材质都会使用此ShaderGUI。

语法：
```shaderlab
CustomEditor "name"
```
参考文档：[自定义着色器GUI](https://docs.unity3d.com/cn/2019.4/Manual/SL-CustomShaderGUI.html)