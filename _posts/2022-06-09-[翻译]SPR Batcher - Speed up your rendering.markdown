---
layout: post
title: "[翻译]SPR Batcher - Speed up your rendering"
data: 2022-06-09 12:00:00 +0800
category: "翻译 TA"
---

The Unity editor has a really flexible rendering engine. You can modify any Material property at any time during a frame. Plus, Unity historically was made for non-constant buffers, supporting Graphics APIs such as DirectX9. However, such nice features have some drawbacks. For example, there is a lot of work to do when a DrawCall is using a new Material. So basically, the more Materials you have in a Scene, the more CPU will be required to setup GPU data.

Unity编辑器是非常灵活的渲染引擎，我们可以在运行期间随时修改材质属性。Unity历史上是基于–非常量缓冲区（non-constant buffers）的，支持如DirectX9这种图形API。所以， 当渲染使用了新材质时，这就需要大量的准备工作；即：在场景中拥有的材质越多，CPU提交给GPU的数据也越多。

![标准Unity渲染流程]({{ "/img/srp/SRP-Batcher-OFF.png" | prepend: site.baseurl }})

During the inner render loop, when a new Material is detected, the CPU collects all properties and sets up different constant buffers in the GPU memory. The number of GPU buffers depends on how the Shader declares its CBUFFERs.

在Unity内部渲染循环中，当检测到新材质时，CPU会收集所有属性并在GPU内存中设置不同的常量缓冲区。GPU缓冲区的数量取决于Shader如何声明其CBUFFER。

### How SRP Batcher works 工作原理

When we made the SRP technology, we had to rewrite some low-level engine parts. We saw a great opportunity to natively integrate some new paradigms, such as GPU data persistence. We aimed to speed up the general case where a Scene uses a lot of different Materials, but very few Shader variants.

如果我们使用SRP技术，我们需要重写一些引擎底层部分。我们有机会整合一些新范例，比如GPU数据持久化，我们的目标是加速使用了大量的材质，但是非常少的Shader变体的场景。

Now, low-level render loops can make material data persistent in the GPU memory. If the Material content does not change, there is no need to set up and upload the buffer to the GPU. Plus, we use a dedicated code path to quickly update Built-in engine properties in a large GPU buffer. Now the new flow chart looks like:

现在，底层渲染循环能使材质在GPU内存中数据持久化。如果材质没有发生变化，则无需设置上传数据到GPU。另外，我们使用一个专用代码快速更新内置的引擎属性至一个大的GPU缓存区。新的流程图是这样的：

![SRP Batcher渲染流程]({{ "/img/srp/image5-3.png" | prepend: site.baseurl }})

Here, the CPU is only handling the built-in engine properties, labeled object matrix transform. All Materials have persistent CBUFFERs located in the GPU memory, which are ready to use. To sum up, the speedup comes from two different things:

CPU只处理引擎内置属性，比如基本转换矩阵，所有的准备要使用材质被托管至GPU的CBUFFERs上，总之，速度提升来自于两点：

Each material content is now persistent in GPU memory
A dedicated code is managing a large “per object” GPU CBUFFER

- 所有材质数据现在都托管在GPU内存中持久化
- 一份专用代码来管理GPU CBUFFER中“每个对象”的基础数据

### SRP Batcher compatibility 兼容性

For an object to be rendered through the SRP Batcher code path, there are two requirements:

一个物体能通过SRP Batcher渲染，有两个条件：

- The object must be in a mesh. It cannot be a particle or a skinned mesh.
- 这个对象必须是使用Mesh组件，而不能是Skinned Mesh组件。
- You must use a Shader that is compatible with the SRP Batcher. All Lit and Unlit Shaders in HDRP and LWRP fit this requirement.
- Shader必须适配SRP Batcher。HDRP和URP下的Lit和Unlit Shader都满足需求。

For a Shader to be compatible with SRP:

Shader适配SRP需要满足：

- All built-in engine properties must be declared in a single CBUFFER named “UnityPerDraw”. For example, unity_ObjectToWorld, or unity_SHAr.
- 引擎内建的属性列表，含有名为UnityPerDraw的CBUFFER声明。例如： unity_ObjectToWorld, unity_SHAr …
- All Material properties must be declared in a single CBUFFER named “UnityPerMaterial”.
- Shader的属性声明，含有名为 UnityPerMaterial的CBUFFER声明

You can see the compatibility status of a Shader in the Inspector panel. This compatibility section is only displayed if your Project is SRP based.

注意1：选中Shader文件，在Inspector面板中，如果SRP Batcher为compatible，则为兼容。

In any given Scene, some objects are SRP Batcher compatible, some are not. But the Scene is still rendered properly. Compatible objects will use SRP Batcher code path, and others still use the standard SRP code path.

注意2：Unity可同时处理并渲染 SRP Batcher兼容与不兼容的对象。兼容的使用SRP Batcher渲染，不兼容的用标准SRP代码渲染

[原文链接](https://blog.unity.com/technology/srp-batcher-speed-up-your-rendering)