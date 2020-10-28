---
layout: post
title: "[Unity Shader] GPU Instancing"
data: 2020-10-28 12:00:00 +0800
category: "Unity Shader"
---
GPU实例化可使用少量绘制调用一次绘制（或渲染）同一网格的多个副本，显著提高项目的渲染性能。对于绘制在场景中重复出现的对象非常有用。GPU 实例化在每次绘制调用时仅渲染相同的网格，但每个实例可以具有不同的参数以增加变化并减少外观上的重复。

在材质的Inspector中勾选`Enable Instancing`复选框即可开启GPU实例化。所有的表面着色器都支持GPU实例化。

### 向着色器中添加GPU实例化参数
示例代码：[UnityShaderPlayground](https://github.com/9b9387/UnityShaderPlayground) 02_GPU Instancing

#### 着色器宏定义：
- `UNITY_INSTANCING_BUFFER_START(name) / UNITY_INSTANCING_BUFFER_END(name)` 着色器中在这对宏内声明每个实例的属性。
- `UNITY_DEFINE_INSTANCED_PROP(float4, _Color)` 声明实例的属性
- `UNITY_ACCESS_INSTANCED_PROP(name, _Color)` 访问实例的属性

#### 在C#代码中访问设置实例的属性
```csharp
    MaterialPropertyBlock props = new MaterialPropertyBlock();
    var renderer = gameObject.GetComponent<MeshRenderer>();
    props.SetColor("_Color", new Color(1.0f, 0.0f, 0.0f));
    renderer.SetPropertyBlock(props);
```



### 其他
#### 注意事项
- 批处理优先级：静态批处理 -> GPU实例化 -> 动态批处理
- #pragma noinstancing 可以在表面着色器中关闭GPU实例化。
- 实例化的绘制调用在帧调试器 (Frame Debugger) 中显示为 Draw Mesh (instanced)。
- 如果多 pass 着色器有两个以上的 pass，则只有第一个 pass 可以实例化。
- 以上示例中使用的所有着色器宏都在 UnityInstancing.cginc 中定义。

#### 限制
- 根据MeshRenderer组件自动处理，不支持SkinnedMeshRenderer。
- 只有共享相同网格和相同材质的对象才会GPU实例化。

#### 支持的平台
- Windows 上的 DirectX 11 和 DirectX 12
- Windows、macOS、Linux、iOS 和 Android 上的 OpenGL Core 4.1+/ES3.0+
- macOS 和 iOS 上的 Metal
- Windows、Linux 和 Android 上的 Vulkan
- PlayStation 4 和 Xbox One
- WebGL（需要 WebGL 2.0 API）

参考文档：[GPU 实例化](https://docs.unity3d.com/cn/current/Manual/GPUInstancing.html)