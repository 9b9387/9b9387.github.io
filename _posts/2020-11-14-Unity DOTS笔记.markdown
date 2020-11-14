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
```C#
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
|   }
}
```

#### Archetype 原型

Archetype是不同类型Component的特定组合，相同Component组合的Enity会放在同一个Archetype里

#### Chunk 块

相同Archetype的数据内存中连续存放于固定大小（16k）的chunk块中。
- 相关component数据快速进入缓存
- Chunk中同类型component数据连续存放（SOA）可以避免缓存行浪费，提升缓存命中。
- 内存管理不使用托管堆，避免GC

### Job System

CPU多线程实现方式，把主线程上的数据处理，分割成多个子任务，放入一个Job Queue中，在通过这个Job队列中，分发到不同的Worker线程上，实现多线程的并行处理。

#### C# Job System

- 通过C#调用，隐藏线程调度，资源竞争等问题的处理。
- 数据类型限制，不可以使用引用类型。
    - struct
    - blittable
    - 非托管内存容器Native container

### Burst

基于LLVM的编译器，LLVM是一个开源的多平台的编译架构
编译过程分为三个步骤：
- 高级语言翻译为LLVM IR的中间语言
- 对LLVM IR中间语言进行优化
- 根据目标平台，把中间语言转化为机器码

Unity实现了一个把C#转化为.NET assembly转化为LLVM IR的编译前端，然后接入到了LLVM框架中，这就形成了Burst编译器

代码运行高效比肩C/C++的原因
- 利用了LLVM的代码优化
- SIMD指令（数据并行）

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
