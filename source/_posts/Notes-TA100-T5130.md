---
title: 技术美术百人计划学习笔记（图形5.1.3 基于物理的灯光）
date: 2023-08-12 14:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 介绍了辐射度量学、光度量学的理论知识(烧脑)，以及游戏引擎中的灯光类型和灯光参数
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true


---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5100.png)

## 辐射度量学基本理论

定义：辐射度量学是一门以整个电磁波段的电磁辐射能测量为研究的科学。而计算机图形学中涉及的辐射度学，则集中对于整个电磁波普中光学谱段中的**可见光谱段的辐射能的计算**。

- 光学谱段范围：指从波长0.1cm的红外线到波长0.1mm的x射线这一距离
- 可见光谱段：波长0.38mm到0.76mm范围内，能对人眼产生目视刺激，从而形成亮光感和色感的谱段

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155110107.png)

### 立体角 Solid Angle

概念：单位球体上的一块区域对应的球面部分的面积

假设我们给定一个正球体，半径为R，再给定一个正圆锥体，正圆锥体的顶点与球心重合，到圆锥底面圆任意一点的连线，即正圆锥体的斜高（斜边），它的长度也为R。由正圆锥体的底面圆S所截取的那一部分球面的面积A和球体半径R的平方的比值称为立体角（球面度）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155323759.png)

单位立体角：一个立体弧度为半径1 米的球面上1 平方米面积所张的立体角。

立体角的微分形式定义如下(以Ω表示立体角)：

![img](https://cdn.nlark.com/yuque/__latex/b996f967bd70eb23923994776341c205.svg)

如果在球面坐标系下对立体角进行定义，面积微元dA的公式为：

![img](https://cdn.nlark.com/yuque/__latex/510ea9781c331fee9ff5195801fba7a1.svg)

由上式可得，整个球面的立体角可以写为关于θ和φ的二重积分公式，其中φ表示维度，θ表示经度，所以整个球面的立体角为4Π；对于一个正方体的面，从该正方体的中心点测量的立体角为(2/3)Π；半球的立体角为2Π

![img](https://cdn.nlark.com/yuque/__latex/f75629cbe76c3a177242462213c2af25.svg)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155346801.png)

### 辐射通量 Radiation Flux

辐射通量定义为以辐射的形式发射，传输或者接受的功率，即单位时间内的辐射能，单位为W，记为ɸ。是为了研究**单位时间**内，**通过某表面的各个光子**所携带的**能量之和是多少**，物理学中引入了辐射通量（Radiation Flux）的概念

**不同的波长**的光引入人对**不同颜色的感知**，**不同的辐射通量**则引起人对光的**不同亮度的感知**

光子虽然在物体表面不停流过，但总体上讲，光子的分布保持一个常数——恒定光源，且开灯瞬间光速很快

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-1.png)

如果知道了任意一点p在任意一个方向w上的辐射通量的值Φ(p,w)，就能得到计算机图形学中的光照问题的完整解决方案。因为知道辐射通量，就能计算出单位时间内的能量，从而计算出光波，根据光波重构颜色。不过因为以上公式计算量极大，所以图形学中更多的是在用近似公式

### 辐射强度 Radiation Intensity

定义：在给定的传输方向上，**单位立体角内**光源发出的**辐射通量**

记辐射通量为Φ，立体角为I，辐射亮度为L，则以微分形式定义光的辐射强度公式为：

![img](https://cdn.nlark.com/yuque/__latex/001b208f605ab71edc8e50f3a4f59048.svg)

对一个面A的区域求积分得到：

![img](https://cdn.nlark.com/yuque/__latex/d67eb8510f7796bfd50ad890a5100564.svg)

I称为表面面积微元dA在方向(α, β)上的辐射强度，与距离无关，但是与发射面dA的面积有关

### 辐射亮度 Radiance

定义：辐射表面在其单位投影面积的单位立体角内发出的辐射通量

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155522508.png)

辐射亮度L的微分定义：

![img](https://cdn.nlark.com/yuque/__latex/4b875145b544c208d8a5c3f1c71d24cf.svg)

辐射亮度描述的是**光源的表面面积微元在垂直传输方向上的辐射强度特性**。好比单去描述一个白炽灯的某一块区域的发射特性是没有实际意义的，应该将它视作一个点光源整体，来描述在某个给定的观察方向上的辐射强度

### 辐射照度 Irradiance

辐射照度被用来测量光进入一个单位面积的强度，也可以表示光离开一个表面的强度，我们称为入射度与出射度

- 入射度定义：单位面积被照射的辐射通量
- 出射度定义：离开光源表面的单位面积的辐射通量

对于点光源，给定辐射通量为Φ，则在一个半径为R的球面某一面积微元上的入射度遵循平方反比定律（距离越近，亮度越大，为距离平方的倒数）



## 光度学基本理论

人眼的灵敏度用CIE光度拉姆达曲线表示，代表我们眼睛对某些光波长的接收效率。人眼的灵敏度在555nm初达到峰值，在我们看来是绿色。这个波长上，灵敏度函数值为1个单位，意味着100%的效率

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155605529.png)

和辐射度量学相比，光度量学只限于可见光的范围内，并且要以人眼的视觉特性为基础。光通量和辐射通量的转换公式为：

![img](https://cdn.nlark.com/yuque/__latex/040e5c1727677353c44785496786d30c.svg)

![链接辐射度量学和光度学的桥梁，是视见函数](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-2.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155705346.png)

![辐射度量和光度量的名称、符号和定义](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155722003.png)



## 灯光类型

### 精确光

Frostbite和Unity3D 只支持两种类型的精确光：**点光源和聚光灯**。为了使精确光在物理上正确，它必须遵循反平方定律，如下图。从恒定亮度的光源观察到的光强与物体距离的平方成正比下降。

平方反比定律只对点光源有效，不适用于：

1. 窄分布的泛光灯和探照灯，因为光束是高度聚光的
2. 区域灯，或像菲涅尔透镜这样的特殊灯

将反平方定律转化为方程，可得：

光照度E = I / 距离

![img](https://cdn.nlark.com/yuque/__latex/92d0b3dcf9aaecac5b50a4a40e06916c.svg)

该方程要求在整个照明计算中，距离单位是均匀的（米，厘米，毫米）

在Frostbite和Unity3D中，1个单位=1米，定义精确光的尺寸为1厘米。光通量（也叫光功率）总是换算成光强度（ luminous intensity ）进行照明计算。通过对光强度在光照立体角上的积分而得到光照度

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155812993.png)

光通量到光强度的转换：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155839824.png)

【Attenuation衰减】

平方反比定律的一个问题是它永远不会到达零，但是由于性能原因，渲染器必须实现有限的光照范围，使得光强度为0，以剔除灯光。在某个极限下，光照度应该平滑地达到零，解决这个问题的一种方法就是以这样的方式对falloff进行窗口化的处理，使大部分功能不受到影响，所以使用基于距离的lerp插值到零：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-3.png)

这个简单的方法可行，但会造成硬性的切断，看起来不自然。第二种方法是用一个threshold对函数进行重置到1，并将其重新映射到初始范围0到1：(参考：https://imdoingitwrong.wordpress.com/2011/02/10/improved-light-attenuation/)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-4.png)

结果较好，但这种方法的缺点是在0处有一个非零的梯度，这会导致一个可见的不连续。更好的方法是对函数进行窗口化处理，并确保lightRadius处的梯度为零。这可以通过提高窗口函数的power来实现

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910155959371.png)

第一张图显示了不同的窗口函数，窗口2是一个不连续的问题，窗口3是一个平滑步长后的 。第二和第三张图显示了分别应用于光半径（Distance）为10和40的范围的窗口函数。这表现了随着光半径的增加，曲线的拟合程度

寒霜和Unity中都是适用的Window1的衰减函数

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-5.png)

### IES

使用光度轮廓来描述其强度分布，这些分布文件被存储在一个光度测量文件中。存在两种常见的格式：IES(.ies)和eulumdat(.ldt) 。在计算机图形学中，大多数都只支持IES格式，Unity和寒霜也不例外。

从IES文件创建的光度轮廓可以直接应用在一个点或聚光灯上。IES轮廓可用于描述光强，并可用一个乘法进行调整。这是控制具有发光强度的灯的唯一方法。第二种选择是使用IES配置文件作为Mask，通过配置文件的最大强度进行归一化

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910160051231.png)

为了用光强度点光方程处理这两种情况，我们根据其最大强度对轮廓进行归一化，光亮度计算如下

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-6.png)

当创建一个新的光轮廓时，球形光度函数被重建并采样，以填充一个球形参数化(θ，cos（φ）)。我们存储按最大强度的倒数缩放的归一化值，来处理MASK和未Mask的使用。在着色器中，2D纹理被计算并作为衰减使用

【IES配置文件的使用】

IES配置文件在室内设计中比在游戏中更有用。能够使用光配置文件作为Mask带来了有趣的效果，IES配置文件可以使用对应的工具创建，并可用于模拟复杂的光影效果，类似于cookie

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910160137397.png)

### 平行光Sun

太阳是非常重要的光源，特别是对于户外环境。太阳对方向和照度变化非常敏感。将这样的光源作为材质漫射部分的精确光是可接受的近似，但对镜面材质这样做会有问题。在寒霜引擎中，为了部分缓解这些问题，将太阳视为一个总是垂直于外半球的圆盘区域光。艺术家为垂直于太阳方向的表面指定太阳照度（以勒克斯为单位）。

下图为他们直接使用的光表

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910160233811.png)



## 灯光参数

常用灯光参考值：[Physical light units - High Definition RP - 13.1.9](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@13.1/manual/Physical-Light-Units.html)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5130-7.png)
