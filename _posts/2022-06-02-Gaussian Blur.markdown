---
layout: post
title: "Gaussian Blur"
data: 2022-06-02 12:00:00 +0800
category: "Blur TA"
---

高斯模糊（Gaussian Blur），也叫高斯平滑（Gaussian smoothing），作为最经典的模糊算法，一度成为模糊算法的代名词。
高斯模糊在图像处理领域，通常用于减少图像噪声以及降低细节层次，以及对图像进行模糊，其视觉效果就像是经过一个半透明屏幕在观察图像。

从数字信号处理的角度看，图像模糊的本质一个过滤高频信号，保留低频信号的过程。过滤高频的信号的一个常见可选方法是卷积滤波。从这个角度来说，图像的高斯模糊过程即图像与正态分布做卷积。由于正态分布又叫作“高斯分布”，所以这项技术就叫作高斯模糊。而由于高斯函数的傅立叶变换是另外一个高斯函数，所以高斯模糊对于图像来说就是一个低通滤波器。

用于高斯模糊的高斯核（Gaussian Kernel）是一个正方形的像素阵列，其中像素值对应于2D高斯曲线的值。
下面是一个典型的高斯核：

$$\frac{1}{256} \cdot \begin{bmatrix}1 & 4 & 6 & 4 & 1\\ 4 & 16 & 24 & 16 & 4\\6&24&36&24&6\\4&16&24&16&4\\1&4&6&4&1 \end{bmatrix}$$

图像中的每个像素被乘以高斯核，然后将所有这些值相加，得到输出图像中此处的值。

N维空间高斯模糊方程表示为：

$$G(r)=\frac{1}{\sqrt{2\pi\sigma^2}^N}e^{\frac{-r^2}{2\sigma^2}}$$

在二维空间定义为：

$$G(u, v)=\frac{1}{2\pi\sigma^2}e^\frac{-(u^2+v^2)}{2\sigma^2}$$

其中，r是模糊半径

$$r^2= u^2 + v^2$$

高斯模糊也可以在二维图像上对两个独立的一维空间分别进行计算，即满足线性可分（Linearly separable）。这也就是说，使用二维矩阵变换得到的效果也可以通过在水平方向进行一维高斯矩阵变换加上竖直方向的一维高斯矩阵变换得到。从计算的角度来看，这是一项有用的特性，因为这样计算复杂度只需要：

$$O(n \times M \times N) + O(m \times M \times N)$$

而原先计算的复杂度为：

$$O(m \times n \times M \times N)$$

其中，M，N是需要进行滤波的图像的维数，m，n是滤波器的维数。

以下为一个Gaussian Kernel的线性分解过程：

$$\frac{1}{256} \cdot \begin{bmatrix}1 & 4 & 6 & 4 & 1\\ 4 & 16 & 24 & 16 & 4\\6&24&36&24&6\\4&16&24&16&4\\1&4&6&4&1 \end{bmatrix} = \frac{1}{256} \cdot \begin{bmatrix}1\\4\\6\\4\\1 \end{bmatrix} \cdot \begin{bmatrix}1&4&6&4&1 \end{bmatrix}$$

实现方面，可以采用经过线性分解的高斯核的方案，且用乒乓RT交互blit的方法。高斯模糊对应的Fragment Shader的实现为：

```c++
float4 FragGaussianBlur(v2f i): SV_Target
{
	half4 color = float4(0, 0, 0, 0);
	
	color += 0.40 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
	color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.xy);
	color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.zw);
	color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.xy);
	color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.zw);
	color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.xy);
	color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.zw);
	
	return color;
}
```