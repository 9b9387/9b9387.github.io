---
layout: post
title: "Blender UV笔记"
data: 2021-02-16 12:00:00 +0800
category: "Blender"
---

 笔记参考 [Blender 2.8 UV贴图基本原理](https://www.bilibili.com/video/BV1RT4y1L7Pf)

### 4.Marking Seams and Unwrapping

- 删除UV Maps
选中物体，在Object Data Properties中，点开UV Maps，点击 `"-"` 按钮。

- Mark Seam
选中需要标记的边，按 `U` 或者 `CMD+E` 键，在菜单中选择 `Mark Seam`。

- Clear Seam
同上，在菜单中选择 `Clear Seam`。

- 展开UV
选中要展开UV的物体，按 `U` ，在菜单中选择 `Unwarp`。

- 操作UV
UV可以在UV Editing进行旋转，移动，缩放。

### 5.Testing for Stretching

- 创建UV Grid贴图
在UV Editor下，点击New Image，Generated Type选择`UV Grid`。这时的贴图的Source为`Generated`，保存后变为`Single Image`。

- 创建 Material
在材质面板中，创建一个新的材质槽，再在Shader Editor下，创建纹理节点 Texture->Image Texture，选择UV Grid贴图。

### 6.Seams and Applying Scale
- 应用变换
在Object Mode下，按`CMD+A`

### 7. Arranging UV islands
- 利用Seam合理拆解UV，合理摆放UV位置，并检查上下方向按自然方式排列。

### 8.Using the UV map for Texturing
- 导出UV Layout
UV Editor的`UV`菜单下，选择`Export UV Layout`.

### 10.Adding the soda can texture
- 创建Color Grid贴图
- UV顶点对齐
对于有一些弯曲的连续UV，可以拉直UV。选中需要对齐的顶点，鼠标右键的菜单中选择`Align X/Y/Z`。

### 11.Using smart UV project
- Lightmap Pack
UV上的各个面拆分成独立的，跟其他面没有任何关联。用于保存烘焙灯光和阴影信息。
光照贴图用于静态网格，如果使用动态光源，那么则不需要光照贴图。

- 烘焙光照贴图
    - 在Shader Editor中新建一个贴图节点，并选中。
    - 在Render Properties面板下，Render Engine为Cycles时，点击Bake下的Bake按钮，可以烘焙光照贴图。

- 智能UV
全选物体，按`U`调出UV菜单，选择Smart UV Project。在Smart UV Project的菜单中，可以调整UV之间的间隔。

### 12.Creating a wood texture in Krita

- 纹理资源库
[https://cloud.blender.org/libraries](https://cloud.blender.org/libraries)

### 13.Creating and applying a normal texture

- Krita制作法线贴图
Krita点击`Filter菜单`，选择`Eage Detection`，点击`Height to Normal Map`

- Shader Editor
    - 选中节点，按`M`键，可以Mute当前节点
    - 设置Normal Map节点
    - 设置RGB曲线节点，调整明暗
    
### 17.Organizing the UV map
- Average Islands Scale
平均所有选中的uv的缩放大小。

- Pack Islands
将所有选中的UV，自动放入0~1的范围内。

### 18.Finish the dumpster UV map
- 重复网格的UV复用
复制，或者镜像对象时，UV重叠。

### 21.Applying the Texture Map to Different Materials
- 同一个物体包含多个材质

### 25.Using the Stitch tool
- UV Stitch
UV缝合工具，将不同的UV缝合在一起，产生无缝的效果。在`UV菜单`下，点击`Stitch`

### 32.Baking a normal map
- 把雕刻`Sculpting`信息，烘焙到法线贴图。和光照贴图类似，只是把`Bake Type`改为 `Normal`

### 34.UV mapping the head
- 固定uv点
选中需要pin的点，uv菜单点击`Pin`。可以固定uv坐标。

- 最小化拉伸
选中需要pin的点，uv菜单点击`Minimize stitch`。