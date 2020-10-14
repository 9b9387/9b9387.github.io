---
layout: post
title: "[Unity Shader] Transfrom"
data: 2020-10-14 12:00:00 +0800
category: "OnePass"
---
之前研究OpenGL的时候接触的矩阵变换，在Unity中用Surface Shader实现一次，当作入门Unity ShaderLab的Hello World。

### Unity内置着色器
Unity内置的Shader库，封装很多常用的数据结构和方法。MacOS系统下，目录位置在`/Unity.app/Contents/CGIncludes`下。使用方法如下：
```shaderlab
CGPROGRAM
// ...
#include"UnityCG.cginc"
// ...
ENDCG
```

### ShaderLab语法
```shaderlab
Shader "name" { [Properties] Subshaders [Fallback] [CustomEditor] }
```
ShaderLab的语法，之后会参照Unity的文档，单独记录其中的各个模块。

### 着色器编译指令
#### 表面着色器的编译指令为：
```shaderlab
# pragma surface surfaceFunction lightModel [optionalparams]
```
- `#pragma surface ...` 指令来指示自己是表面着色器。
- `surfaceFunction` 具有表面着色器代码的Cg函数。
    - 格式应为: `void surfaceFunction (Input IN, inout SurfaceOutput o)`
    - 其中`Input`为自定义结构，包含所需的变量，如uv, 法线等。纹理坐标必须命名为“uv”后跟纹理名称的形式（如果要使用第二个纹理坐标集，则以“uv2”开头）。
    - `SurfaceOutput`是简单的非基于物理的 Lambert（漫射）和 BlinnPhong（镜面反射）光照模型的参数类型。定义在`Lighting.cginc`文件中。
    - 还有内置光照模型是基于物理的`Standard`和`StandardSpecular`光照模型，分别使用`SurfaceOutputStandard`和`SurfaceOutputStandardSpecular`输出结构。

- `optionalparams`可选参数中常用的是自定义修改器函数:
  - `vertex:VertexFunction` 自定义顶点修改函数。在生成的顶点着色器的开始处调用此函数，并且此函数可以修改或计算每顶点数据。
  - `finalcolor:ColorFunction` 自定义最终颜色修改函数。

详细参数介绍参考：[Unity文档：编写表面着色器](https://docs.unity3d.com/cn/2019.4/Manual/SL-SurfaceShaders.html)

#### 顶点/片元着色器编译指令
```shaderlab
#pragma vertex vert
#pragma fragment frag
```
1. `#pragma vertex vert` 指示vert函数为顶点着色器编译函数
2. `#pragam fragment frag`作为frag函数为片元着色器编译函数

之后的shader编写会以表面着色器为主，所以顶点/片元着色器只做简单记录。

顶点/片元着色器示例参考：[Unity文档：向顶点程序提供顶点数据](https://docs.unity3d.com/cn/2019.4/Manual/SL-VertexProgramInputs.html)

更多编译指令参考：[Unity文档：编写顶点和片元着色器](https://docs.unity3d.com/cn/2019.4/Manual/SL-ShaderPrograms.html)

### 矩阵变换
位移矩阵:
$$
M_{translate}(x, y, z) = 
\left\{ \begin{matrix} 
1 & 0 & 0 & x \\ 
0 & 1 & 0 & y \\ 
0 & 0 & 1 & z \\ 
0 & 0 & 0 & 1 
\end{matrix} \right\}
$$

缩放矩阵：
$$
M_{scale}(x, y, z) = 
\left\{ \begin{matrix} 
x & 0 & 0 & 0 \\ 
0 & y & 0 & 0 \\ 
0 & 0 & z & 0 \\ 
0 & 0 & 0 & 1 
\end{matrix} \right\}
$$

旋转矩阵：
三维旋转的顺序先绕z轴旋转，其次是x轴，y轴最后。
$$
M_{rotateZ} = 
\left\{ \begin{matrix} 
cos\theta & -sin\theta & 0 \\
sin\theta & cos\theta & 0 \\
0 & 0 & 1
\end{matrix} \right\}
$$

$$
M_{rotateX} = 
\left\{ \begin{matrix} 
1 & 0 & 0 \\
0 & cos\theta & -sin\theta \\
0 & sin\theta & cos\theta
\end{matrix} \right\}
$$

$$
M_{rotateY} = 
\left\{ \begin{matrix} 
cos\theta & 0 & sin\theta \\
0 & 1 & 0 \\
-sin\theta & 0 & cos\theta
\end{matrix} \right\}
$$

所以顶点旋转的公式为：
$$
P' = M_{rotateY} M_{rotateX} M_{rotateZ} P
$$

最终的三维旋转矩阵为：
$$
M_{rotate}(x, y, z) = 
\left\{ \begin{matrix} 
cosY * cosZ & -cosY * sinZ & sinY & 0.0 \\
cosX * sinZ + sinX * sinY * cosZ & cosX * cosZ - sinX * sinY * sinZ & -sinX * cosY & 0.0 \\
sinX * sinZ - cosX * sinY * cosZ & sinX * cosZ + cosX * sinY * sinZ & cosX * cosY & 0.0 \\
0.0 & 0.0 & 0.0 & 1.0
\end{matrix} \right\}
$$

在`vert`函数中对顶点坐标依次应用变换。对应`TransformShader.shader`

```shaderlab
// 顶点函数
void vert(inout appdata_full v)
{
    // 先缩放、后旋转、再平移
    // 应用缩放
    v.vertex = mul(Scale(_Scale), v.vertex);

    // 应用旋转
    v.vertex = mul(Rotation(_Rotation), v.vertex);

    // 应用位移
    v.vertex = mul(Translate(_Translate), v.vertex);
}
```

顶点缩放，旋转，移动的公式：
$$
\left\{ \begin{matrix}
x \\ y \\ z
\end{matrix} \right\}
= M_{translate} * M_{rotate} * M_{scale} *
\left\{ \begin{matrix}
x_0 \\ y_0 \\ z_0
\end{matrix} \right\}
$$

三个变换矩阵相乘后的复合矩阵。对应`TransformShaderTRS.shader`
$$
\left\{ \begin{matrix}
cosY * cosZ * s.x & -cosY * sinZ * s.y & sinY * s.z & t.x \\
(cosX * sinZ + sinX * sinY * cosZ) * s.x & (cosX * cosZ - sinX * sinY * sinZ) * s.y & -sinX * cosY * s.z & t.y \\
(sinX * sinZ - cosX * sinY * cosZ) * s.x & (sinX * cosZ + cosX * sinY * sinZ) * s.y & cosX * cosY * s.z & t.z \\
0.0 & 0.0 & 0.0 & 1.0
\end{matrix} \right\}
$$

### 源码
项目仓库地址：[UnityShaderPlayground](https://github.com/9b9387/UnityShaderPlayground)