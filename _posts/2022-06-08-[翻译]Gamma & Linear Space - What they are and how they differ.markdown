---
layout: post
title: "[翻译]Gamma & Linear Space - What they are and how they differ"
data: 2022-06-08 12:00:00 +0800
category: "翻译 TA"
---

Linear space lighting is a term that game developers are becoming ever more used to hearing as games reach for the next level of realism with physically based rendering models (PBR). Though linear space and its counterpart, gamma space, are fairly simple and important concepts to understand, many developers don't learn what these terms really mean. This document will define gamma and linear space, how they differ, and how it applies to the Unity engine.

随着基于物理的渲染模型（PBR）游戏的普及，`线性空间（Linear Space）`光照成为了游戏开发者经常听到的术语。虽然线性空间和它的对应物，`伽马空间（Gamma Space）`是相当简单和重要的概念，许多开发人员并不知道这些术语的真正含义。本文档将解释线性空间和伽马空间，以及说明它们之间有何不同并如何应用在Unity引擎中。

### LINEAR SPACE 线性空间

First we need to know what linear color space is. Simply, it means that numerical intensity values correspond proportionally to their perceived intensity. This means that the colors can be added and multiplied correctly. A color space without that property is called ”non-linear”. Below is an example where an intensity value is doubled in a linear and a non-linear color space. While the corresponding numerical values in linear space are correct, in the non-linear space (gamma = 0.45, more on this later) we can’t simply double the value to get the correct intensity.

首先我们需要知道什么是线性颜色空间。简单地说，就是数值的大小与感知强度为对应关系，这意味着颜色可以正确的相加和相乘。没有这种属性的颜色空间被称为“非线性”。下面是一个例子，强度值在线性和非线性颜色空间分别做加倍计算，在线性空间中对应的结果是正确的，但是在非线性空间中（gamma=0.45，稍后会详细介绍），我们不能通过简单的计算获取正确的数值强度。

![在线性和非线性空间中加倍深灰色正方形的强度]({{ "/img/gamma_linear/Linear.png" | prepend: site.baseurl }})

### GAMMA SPACE 伽马空间

The need for gamma arises for two main reasons: The first is that screens have a non-linear response to intensity. The other is that the human eye can tell the difference between darker shades better than lighter shades. This means that when images are compressed to save space, we want to have greater accuracy for dark intensities at the expense of lighter intensities. Both of these problems are resolved using gamma correction, which is to say the intensity of every pixel in an image is put through a power function. Specifically, gamma is the name given to the power applied to the image.

之所以需要伽玛射线，主要有两个原因：第一，屏幕对强度的响应是非线性的，另一个原因是，相比于浅色调人类的眼睛能更好的分辨暗色调的不同。这意味着我们通过压缩图片来节省空间时，我们可以牺牲浅色的精度为代价来提供给暗色更多的精度。这两个问题都可以用伽马校正来解决，也就是说图片上每一个像素点的强度可以用一个幂函数来表示，伽马就是这个幂函数的名字。

![伽马函数的图像 pow(x, gamma)]({{ "/img/gamma_linear/gamma_graph.png" | prepend: site.baseurl }})

Looking at the above graph, it is apparent that any gamma aside from 1 is a non-linear space.

上面的图表很明显，除1以外的任何gamma都是非线性空间。

![两个常见的gamma值应用在中间的图像上]({{ "/img/gamma_linear/various_gamma.png" | prepend: site.baseurl }})

Most images are stored with a gamma of 0.45 applied to them, which will have the effect demonstrated in the above left image. The darker regions of the image are now recorded using a greater range of values while bright ranges are compressed. Images stored like this are in “gamma space”. For example, a neutral grey in an image doesn’t have a numerical intensity of 0.5, rather it is around 0.73, while pure whites and blacks remain the same. This is the default behavior of nearly every digital camera, as well as image editing applications and so on. In fact, nearly every image you see on your computer has that gamma applied to it.

大多数的图像都用伽马值0.45来储存，如上面左图所示的效果，图像中较暗的区域使用更多的值域来记录，而较亮的区域则被压缩。这样储存的图像是在伽马空间中。例如，图像中的中间的灰色的数值强度并不是0.5，而是约为0.73，而纯白色和纯黑色保持不变。这几乎是所有的数码相机以及图像编辑应用程序的默认行为，实际上，你在电脑上看到的几乎每一张图像都应用了伽马。

You may be wondering why images display correctly, and don’t look too bright. This is where the non-linear response of displays comes in. CRT screens, simply by how they work, apply a gamma of around 2.2, and modern LCD screens are designed to mimic that behavior. A gamma of 2.2, the reciprocal of 0.45, when applied to the brightened images will darken them, leaving the original image.

您可能想知道为什么图像显示正确，而不看起来太亮。这就是显示的非线性响应发挥作用的地方。简单地说，CRT屏幕的工作原理是，伽马值在2.2左右，而现代LCD屏幕就是为了模仿这种行为而设计的。gamma值为2.2，是0.45的倒数，当应用于明亮的图像时，将使其变暗，留下原始图像。

For example, the above left image represents what is stored on your computer, and when that is displayed, the result looks like the center image. The left image has been “gamma corrected”, which means it has a gamma value such that it will be displayed correctly after the screen’s gamma is applied to the image. If instead the computer is storing the center image, which has not been gamma corrected, the displayed result looks like the right image, clearly unwanted behavior.

例如，左上方的图像表示存储在计算机上的内容，当它显示时，结果看起来就像中间的图像。左边的图像已经被“伽玛校正”，这意味着它有一个伽玛值，这样它将被正确显示后，屏幕的伽玛应用到图像。如果计算机存储的是中间的图像，而中间的图像没有经过伽马校正，那么显示的结果看起来就像右边的图像，这显然是不想要的行为。

### COLOR SPACES AND THE RENDERING PIPELINE 颜色空间和渲染管线

When using the gamma pipeline in rendering, textures are passed into shaders, gamma corrected. Next the lighting is calculated. After, the final image is output to the display and adjusted by the display’s gamma value. This behavior, while simple, is not physically correct. In real life, light behaves linearly, which means that each contribution from multiple light sources added together give the correct intensity. Because of this, shading is done in linear space, but in the gamma pipeline the input colors and textures remain in gamma space. This means the result of the shading is not truly accurate, but after the display’s correction it is often good enough. However, with increasing demand for immersive, photo-real rendering, this is no longer a suitable method.

当在渲染中使用伽马管道时，纹理被传递到着色器，伽马校正。接下来计算光照之后，最终图像被输出到显示器，并由显示器的伽玛值进行调整。这种行为虽然简单，但在物理上是不正确的。在现实生活中，光的表现是线性的，这意味着来自多个光源的每个贡献加在一起会给出正确的强度。正因为如此，阴影是在线性空间中完成的，但在gamma管道中，输入颜色和纹理仍然在gamma空间中。这意味着阴影的结果不是真正准确的，但在显示器的校正后，它通常是足够好的。然而，随着对沉浸式、照片真实渲染的需求不断增加，这已经不再是一种合适的方法。

Therefore, typical practice in PBR is to use a linear pipeline. Here, the input colors and textures have their gamma correction removed before shading, putting them into linear space. When shaded, the result is physically correct because the shading process and inputs are all in the same space. After, any post effects should be computed while the frame is still in linear space, as post effects are typically linear, much like shading. Finally the image is then gamma corrected so it will have the proper intensity after the display’s gamma adjustments.

因此，PBR的典型做法是使用线性管道。在这里，输入的颜色和纹理在着色前的伽马校正被删除，将它们放入线性空间。当着色时，结果是物理上正确的，因为着色过程和输入都在同一个空间。之后，任何后处理也应该在线性空间下进行计算，因为后期效果通常是线性的，比如阴影。最后，对结果进行伽马校正，使其在显示器的伽马调整后正确显示。

![对比伽马和线性管线]({{ "/img/gamma_linear/comparing.png" | prepend: site.baseurl }})

Above you can see the difference between the end result of the gamma and linear pipelines on a simple sphere. Notice the brighter specular highlight and stronger falloff of the light in gamma space. These are examples of unrealistic behavior, and will make achieving photo-realism difficult.

上图中，在一个简单的球体上可以看出伽马和线性最终结果的区别。注意，在伽马空间中，有更明亮的镜面高光和更强的衰减。这些都是不真实的表现，很难实现更有真实感的效果。

### COLOR SPACES IN UNITY Unity下的颜色空间

Fortunately, Unity is able to switch between color spaces very easily, and for many projects will seamlessly work with your rendering pipeline. By default, Unity uses gamma space as only PC, Xbox, and PlayStation platforms support linear rendering. For these platforms, to change between linear and gamma space, go to:

幸运的是，Unity可以很容易的在不同的颜色空间之间切换，对于许多项目来说，可以无缝地与你的渲染管线一起工作。Unity默认使用伽马空间，因为只有PC，Xbox和PlayStation平台支持线性渲染（注：文章写于2016年，现在移动设备也支持线性渲染），对于这些平台，要在线性空间和伽马空间之间进行转换，需要执行：

**Edit -> Project Settings -> Player -> Other Settings**

Here, there will be an option called Color Space, where you can choose either Linear or Gamma. It’s as easy as that! Shaders should now receive non-gamma corrected textures. Do note that if your project is already in development, you’ll likely need to rework the lighting and various textures to produce a good result, since the rendered scene will not look the same as before. If you had any lightmaps baked, they will need to be re-baked to be correct.

在这里，将有一个选项称为颜色空间，你可以选择线性或伽马。就这么简单！着色器现在应该接收非伽马校正纹理。请注意，如果你的项目已经在开发中，你可能需要重做灯光和各种纹理，以产生一个良好的结果，因为渲染的场景将不像以前一样。如果你有任何光照贴图，他们将需要重新烘焙，以确保正确。

When linear color space and HDR are used, all post effects will be done by Unity in a full linear space. However, when only linear color space is enabled, Unity will use a gamma framebuffer, but fortunately when reading and writing to this buffer, Unity will automatically convert the values’ color space properly so that the image effects are still done in linear space.

当使用线性颜色空间和HDR时，所有的后期效果将由Unity在一个完整的线性空间中完成。然而，当只启用线性颜色空间时，Unity将使用伽马帧缓冲区，但幸运的是，当读写该缓冲区时，Unity将自动正确转换值的颜色空间，以便图像效果仍然在线性空间中完成。

Although Unity does not support the default linear pipeline on some platforms such as mobile. It is possible to do so yourself within shaders. This is done by applying the pow() function to gamma corrected input textures to transform the inputs to linear space, and applying pow() again before returning the result to put it back in gamma space. Note that this method will be computationally expensive, so be aware of the capabilities of your target devices and use it only where needed.

虽然Unity在一些平台上不支持默认的线性管道，比如移动平台。你自己也可以在着色器中这样做。这是通过将pow()函数应用到伽马校正的输入纹理，将输入转换为线性空间，并在返回结果之前再次应用pow()将其放回伽马空间来完成的。注意，这种方法的计算成本很高，所以要了解目标设备的功能，并只在需要的地方使用它。

### CONCLUSION 结论

Hopefully, you have gained a further understanding of gamma and linear space and now know where to begin applying these principles to your projects. If you are making a lot of custom changes to Unity’s rendering pipeline, shaders, and post effects, you’ll need to strongly grasp the fundamentals to avoid common mistakes in color management that can drag your image quality down. When implemented correctly however, a linear rendering pipeline is a key part on the road to delivering immersive, realistic worlds.

希望您已经对gamma和线性空间有了进一步的理解，现在知道从哪里开始将这些原则应用到您的项目中。如果你正在对Unity的渲染管道、着色器和后期效果进行大量自定义更改，你就需要牢牢掌握基本原理，以避免颜色管理中常见的错误，从而降低图像质量。然而，当实现正确时，线性渲染管道是交付沉浸式、逼真世界道路上的关键部分。

To learn more about our team at KinematicSoup and how we are working to help Unity developers build better games faster, check out our real-time multi-user Unity scene collaboration tool, Scene Fusion.

想要更多地了解我们的KinematicSoup团队，以及我们如何帮助Unity开发者更快地构建更好的游戏，请查看我们的实时多用户Unity场景协作工具，场景融合。

### FURTHER READING 更多

[http://docs.unity3d.com/Manual/LinearLighting.html](http://docs.unity3d.com/Manual/LinearLighting.html)

[http://http.developer.nvidia.com/GPUGems3/gpugems3_ch24.html](http://http.developer.nvidia.com/GPUGems3/gpugems3_ch24.html)

[http://www.slideshare.net/ozlael/hable-john-uncharted2-hdr-lighting](http://www.slideshare.net/ozlael/hable-john-uncharted2-hdr-lighting)

这篇文章作者为：Scott Sewell, KinematicSoup的一个开发者，翻译：9b9387。

[原文链接](https://www.kinematicsoup.com/news/2016/6/15/gamma-and-linear-space-what-they-are-how-they-differ)