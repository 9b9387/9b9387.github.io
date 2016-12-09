---
layout: post
title: "Unity摄像机控制"
data: 2016-12-07 16:26:00 +0800
categories: diary
---

摄像机控制是一个非常常见的需求，这次花了点时间整理一下，单独把这部分的功能提炼出来，封装成一个通用类。

### 先列一下需求：

- 摄像机视野平移
- 摄像机视野缩放
- 摄像机视野旋转，摄像机的forward方向与世界坐标系z轴同向。实现视野绕x轴，y轴旋转。z轴不旋转。

### 设计方案：
三个需求，比较难的是视野旋转。这里的实现方式是，假设摄像机视野的中心点是球体的中心点，摄像机的位置到球心的距离是球的半径，那摄像机出现的位置就在球体的表面上的某个位置。

这里定义球心为摄像机的**锚点**，锚点计算方式为，摄像机的forward方向与y=0平面的交点。

```
void CalcAnchorPoint() {

	Vector3 camForward = Camera.main.transform.forward;
	Vector3 camPos = Camera.main.transform.position;
	float k = Mathf.Abs(camPos.y / camForward.y);

	anchorPoint.x = camForward.x * k + camPos.x;
	anchorPoint.y = 0;
	anchorPoint.z = camForward.z * k + camPos.z;
}
```
**平移的实现方式：**摄像机的坐标 - 偏移向量。要注意的是，输入的向量需要做角度的变换。否则在旋转后平移会出Bug。我把摄像机当前y轴的角度保存到一个变量angleY里。平移结束后更新锚点位置。

```
private void Move(float delatX, float deltaY, float deltaZ) {

	Vector3 orginVec = new Vector3(delatX, deltaY, deltaZ);
	Vector3 rotatedVec = Quaternion.AngleAxis(Mathf.Rad2Deg * angleY, Vector3.down) * orginVec;

	Camera.main.transform.position -= new Vector3 (rotatedVec.x, rotatedVec.y, rotatedVec.z) / 100;
	CalcAnchorPoint();
}
```

**缩放的实现方式：**可以理解成为改变球体半径，摄像机会朝向锚点的（反）方向移动。计算出偏移量，然后加到摄像机的位置坐标。

```
private void Scale(float delta) {
	Vector3 camPos = Camera.main.transform.position;
	Vector3 offset = (camPos - anchorPoint) * delta;

	Camera.main.transform.position -= offset;
}
```

**旋转的实现方式：**计算摄像机到锚点的距离，然后根据偏移的角度计算球体表面的位置，更新位置后LookAt锚点。

```
private void Rotate(float delatAngleX, float delatAngleY) {
	Vector3 camPos = Camera.main.transform.position;
	float radius = Vector3.Distance (camPos, anchorPoint);

	angleY -= delatAngleY / 100;
	angleX += delatAngleX / 100;

	float y = Mathf.Sin (angleX) * radius;
	float r2d = Mathf.Cos (angleX) * radius;
	float x = Mathf.Sin (angleY) * r2d;
	float z = -Mathf.Cos (angleY) * r2d;

	Camera.main.transform.position = new Vector3(anchorPoint.x + x, anchorPoint.y + y, anchorPoint.z + z);
	Camera.main.transform.LookAt (anchorPoint);
}
```

最后根据实际需要加入限制条件就可以使用了，我用[TouchKit](https://github.com/prime31/TouchKit)的手势代码，注册手势事件来调用对应的视角控制方法。

- 单指拖动时，调用Move方法
- 双指拖动时，调用Rotate方法
- Pinch手势时，调用Scale方法

最后效果还不错，大部分都是数学知识。另一种实现方式是，锚点可以用一个GameObject代替，摄像机作为它的子节点会更容易计算，但是添加到实际项目中的时候配置会多一点点，而且逼格也低一些。