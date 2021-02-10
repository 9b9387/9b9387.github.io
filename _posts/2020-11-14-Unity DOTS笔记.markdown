---
layout: post
title: "Unity DOTS笔记"
data: 2020-11-14 12:00:00 +0800
category: "Unity"
---

DOTS是Data Oriented Technology Stack的缩写，其核心内容包括：
- Entity Component System
- C# Job System
- Burst Compiler

#### 优势
- 面向数据思想，提高硬件的并行能力。
- 解藕面向对象中的数据和逻辑。
- 适合大批量数据查询/计算。
- ECS面向数据，缓存友好
- Job/Burst充分发挥CPU并行性能
- ECS和Job System（+Burst）可以独立或者混合使用

#### Cache

- 缓存速度比内存快，但是容量小，时间开销少
- 数据计算不在缓存中

#### GameObject

- 数据散放在内存中（减慢缓存的写入）
- 多余数据一起加载（占用缓存空间）

#### 快速查询

查询entity和获取component数组数据效率远高于 扫描式查询`GameObject.Find`，`GameObject.GetComponents`

和数据库技术相似

| ECS | DataBase |
| --- | --- |
| Entity | Key |
| Archetype | Table Schema |
| Component | Column |
| Entity with Components | Record |

### Entity Component System

概念：
- Entity ID
- Component 数据
- System 逻辑
- World Manager 管理器

例子:
```csharp
// 面向对象
class Movement : MonoBehaviour
{
    public float x;
    public float y;
    
    void Update()
    {
        x++;
        y++;
    }
}

// 面向数据
struct Position : IComponentData
{
    public float x;
    public float y;
}

class MovementSystem : IComponentSystem
{
    void OnUpdate()
    {
        Entities.Foreach((ref Position pos)=>{
            pos.x++;
            pos.y++;
        });
    }
}
```

#### Component
Component就是struct结构体，因为是值类型，所以它是不会产生GC的，也不需要垃圾回收。
- IComponentData 通用组件
- ISharedComponentData 共享组件
需要实现`IEquatable<T>`接口，比较两个共享组件是否相等
- ISystemStateComponentData 系统状态组件
ECS不提供事件和回调，系统状态组件就是来解决这个问题的。当System对通用组件进行修改时，同步给系统状态组件。
- IBufferElementData 动态缓冲区
解决Job中使用堆内存数据的问题。

#### [WriteGroup]
覆盖组件，如果A，B同时添加在一个实体上，可以通过这个标签进行覆盖操作。

#### Archetype 原型

Archetype是不同类型Component的特定组合，相同Component组合的Enity会放在同一个Archetype里，Archetype其实就是一个固定长度的数组容器。

每个Archetype是固定16k字节，分成若干个块。好处就是当Chunk的容量不够的时候再开辟一个新的Chunk，Chunk和Chunk之间内存依然是连续的。

#### Chunk 块

相同Archetype的数据内存中连续存放于固定大小（16k）的chunk块中。
- 相关component数据快速进入缓存
- Chunk中同类型component数据连续存放（SOA）可以避免缓存行浪费，提升缓存命中。
- 内存管理不使用托管堆，避免GC

### Job System

大部分的Unity的API都必须运行在主线程上，所以大部分的Unity API不能在Job System调用。

CPU多线程实现方式，把主线程上的数据处理，分割成多个子任务，放入一个Job Queue中，在通过这个Job队列中，分发到不同的Worker线程上，实现多线程的并行处理。

多线程数据有三种状态：只读，只写，可读可写。如果某个数据只读，那这个数据在多线程访问永远都是安全的，如果数据可写就需要加锁。Unity并不希望加锁和解锁的操作出现在业务逻辑层。所以Unity提供了两个标签`[WriteOnly]`和`[ReadOnly]`

- ComponentSystem
在主线程中运行，无法利用多核并行处理。
- JobComponentSystem
多线程并行，如果都是只读数据，可以完成并行，但是如果其中一个job在改数据，第二个job必须要等待第一个job结束。这就是硬性同步点的概念。
- EntityCommandBuffers
System执行时标记会导致同步点的操作，最后同一一次删除。

#### [DisableAutoCreation]
Unity会默认激活所有System，加上`[DisableAutoCreation]`就不会被自动激活。
通过`system.Enable`控制开启/关闭

#### [UpdateBefore]和[UpdateAfter]
用于控制System的执行顺序

#### 销毁一个System
官方并不建议动态创建或销毁。
```csharp
World world = World.DefaultGameObjectInjectionWorld;
MySystem system = world.GetOrCreateSystem<MySystem>();
var simulationSystemGroup = world.GetOrCreateSystem<SimulationSystemGroup>();
simulationSystemGroup.RemoveSystemFromUpdateList(system);
World.DefaultGameObjectInjectionWorld.DestroySystem(system);
ScriptBehaviourUpdateOrder.UpdatePlayerLoop(world);
```

#### System的声明周期

```csharp
public class MySystem : ComponentSystem
{
    protected override void OnCreate()
    {
        //系统初始化调用，并且只会被调用一次
        base.OnCreate();
    }
   
    protected override void OnDestroy()
    {
        //系统被删除时调用
        base.OnDestroy();
    }
     
    protected override void OnStartRunning()
    {
        //系统每次进入激活状态时调用
        //每次 Enabled = true;时就会调用
        base.OnStartRunning();
    }
     
    protected override void OnStopRunning()
    {
        //系统每次取消激活状态时调用
        //每次 Enabled = false;时就会调用
        //系统被销毁时会优先调用，然后才会调用OnDestroy()
        base.OnStopRunning();
    }
     
    protected override void OnUpdate()
    {
        //正常更新
    }
}
```
#### C# Job System

- 通过C#调用，隐藏线程调度，资源竞争等问题的处理。
- 数据类型限制，不可以使用引用类型。
    - struct
    - blittable
    - 非托管内存容器Native container(HPC# 数据结构)

### Burst

基于LLVM的编译器，LLVM是一个开源的多平台的编译架构
编译过程分为三个步骤：
- 高级语言翻译为LLVM IR的中间语言
- 对LLVM IR中间语言进行优化
- 根据目标平台，把中间语言转化为机器码

Unity实现了一个把C#转化为.NET assembly转化为LLVM IR的编译前端，然后接入到了LLVM框架中，这就形成了Burst编译器

代码运行高效比肩C/C++的原因
- 利用了LLVM的代码优化
- SIMD指令（single instruction, multiple data 数据并行）

普通指令 Scalar
```
A + B = A+B
```

SIMD指令
```
A1, A2, A3 ... + B1, B2, B3 ... = A1+B1, A2+B2, A3+B3...
```
Burst的限制
- 只能用于Job System和Function Pointer（FunctionPoiner<T>）
- 不能使用引用类型
- 无法try/catch
- 无法访问托管对象
- 无法写入静态变量

#### [BurstCompile]
目前Burst编译器`[BurstCompile]`必须放在继承IJob、IJobParallelFor等接口上面才能生效，如果给一个普通的结构体上写入`[BurstCompile]`Burst编译器是无法生效的。

对应的`[BurstDiscard]`表示这个方法不进行`[BurstCompile]`编译。
### HPC#
C#效率低的大部分原因是GC，GC需要查找需要释放的对象，大部分耗时就是用来寻找。此外GC操作还会分配额外的内存。

Job System里传递数据必须是HPC#的数据结构，同样的原理，即便我们不使用DOTS依然可以单独使用HPC#来优化内存。

例如：
```
A对象只占4字节，标记指针4字节，同步块索引4字节，最终A对象在内存中会占用12字节，不过标记指针和同步块索引是由CLR进行管理，所以我们在GC中只会看到4字节。
```

C#中，值类型会被分配到栈上，而栈上的数据是不需要GC的。但是数组型结构是个例外，比如List\<int\>, int[]，虽然数组中的元素是值类型，但是数组本身是个引用类型，所以它还是会被分配到堆内存上面。

为此，Unity提供了HPC#高性能C#方法，不产生GC，手动释放(调用Dispose()方法)。

#### NativeList

Native List是一个经过封装的数组，为了高性能，并不提供`Remove`和`Insert`操作，但是提供了一种移除操作的实现方式`RemoveAtSwapBack`，先把需要移除的元素和最后一个元素交换位置，然后再删除最后一个元素。这样会破坏排序，所以推荐用这种数据结构保存排序无关的数据。

#### NativeQueue
Native Queue提供入队出队操作，不提供索引取值操作。

#### NativeHashMap
Native HashMap保存键值对，不能用foreach遍历，遍历需要获取keys，用for遍历。

#### NativeMultiHash
相同的key可以对应多个不同的Value

#### NativeSlice
切片，用于数据结构切块和拷贝。

#### NativeStream
用于原生读写二进制数据。

### Unity.Mathematics
`Mathematics`是配和Burst编译器使用，支持SIMD运算，对比`Mathf`可以大幅提高性能。