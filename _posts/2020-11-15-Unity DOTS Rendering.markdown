---
layout: post
title: "Unity DOTS Rendering"
data: 2020-11-15 12:00:00 +0800
---

ECS的渲染步骤：
- 数据
    - 位置
    - 模型信息
        - Mesh
        - Material
- 调用渲染API / Hybrid Render

Hybrid Render是Unity提供的DOTS渲染包，提供基于ECS的渲染System，封装了Culling和渲染API调用。

通过LocalToWorld和RenderMesh两个Components，Hybrid Render就可以把物体渲染到屏幕上。

示例代码
```C#
// Entity初始化，包含两个渲染Components LocalToWorld和RenderMesh
Entity cube = defaultWorld.EntityManager.CreateEntity(
    ComponentType.ReadOnly<LocalToWorld>(),
    ComponentTYpe.ReadOnly<RenderMesh>()
);
// 设置位置信息
defaultWorld.EntityManager.SetComponentData(cube, new LocalToWorld{
    Value = new float4x4(
        rotation: quaternion.identity, translation: new float3(0, 0, 0)
        )
});
// 设置模型信息
defaultWorld.EntityManager.SetSharedComponentData(cube, new RenderMesh{
    mesh = myMesh,
    material = myMaterial
});
```
### ComponentData vs SharedComponentData

**ComponentData**是每个对象自己的数据，使用非托管堆Chunk内存布局，实现IComponentData，数据类型为Struct和指类型，不能引用托管对象。但是也支持Managed IComponentData，是class类型，不能使用Burst和Job

**SharedComponentData**是相同Chunk的Entity共同关联的数据，由Chunk之外的SharedComponentManager进行管理，可以使用引用类型。用于数据共享，最主要的内容是对Entity进行Chunk分组。分组的好处，可以降低Set Pass call，优化GPU Instancing

### Draw API
- DrawMesh
- DrawMeshInstanced 移动平台大部分兼容
- DrawMeshInstancedIndirect 功能完善
- BatchRendererGroup 2019新增，通用渲染类，兼容ECS，Bultin和SRP，是渲染API的高层封装，Hybrid Render使用该API进行渲染。可使用自己的Culling方法，例如用Job加速culling。

### GameObject转为ECS
在GameObject上挂载ConvertToEntity脚本，可以自动将
- Transform -> LocalToWorld
- Mesh Renderer -> RenderMesh

自定义MonoBehavior组件转为ECS
- 转移逻辑到System
- 自定义数据转换
    - GameObjectConversionSystem
    - IConvertGameObjectToEntity
    - IDeclareReferencedPrefabs

可以在开发阶段保留Unity传统模式，并在Runtime阶段使用ECS高效处理数据。

GameObject全部转换为ECS是一个艰巨的任务，并且有可能不可能实现，转移MonoBehavior逻辑到System繁琐，第三方的插件也难以转换。用到了复杂的渲染管线更增加了难度。

#### DOTS-GameObject混合使用
- ECS / Job System负责部分逻辑运算，使用GameObject进行渲染
- 继续使用原来的渲染管线
- MonoBehavior组件继续有效

在ConverToEntity组件中，选择转换模式为`Convert And Inject GameObject`，可以使GameObject同时拥有ECS和GameObject两套数据。
使用方式：
```csharp
// 方式1：直接在System中获取Transform组件
this.Entities.WithAll<Player>().ForEach((Transform trans) => {
    trans.position += speed * timedelta * Vector3.up
}).WithoutBurst().Run();
// 方式2：Job中更新Entity位置数据后，同步至GameObject，在转换函数中为entity添加CopyTransformToGameObject
public void Convert(Entity entity, EntityManager dstManager, GameObjectConversionSystem conversionSystem)
{
    dstManager.AddComponentData(entity, new CopyTransformToGameObject());
}
```
方式2性能高于方式1，方式1高于GameObject模式

Unity提供一种更简单使用Job System多线程更新Transform的方式，不需要ECS的概念直接实现`IJobParallelForTransform`的接口`Execute`即可，可以快速改造Transform。

使用`IJobParallelForTransform`性能高于方式2，相对于传统方式提高大约50%的运算速度。