---
title: 技术美术百人计划学习笔记（图形5.1.1 基于物理的光照模型）
date: 2023-08-12 12:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 简单介绍了PBR的思想，以及Cook-Torrance反射率方程
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5100.png)

## PBR框架概述

PBR框架包括：

- 基于物理的材质（光照模型）
- 基于物理的灯光
- 基于物理的相机
- 以及美术的PBR全流程。

PBR背后更重要的是PBR带来的**框架理念：轻松地利用一个框架去保证项目的整体画面统一**



“基于物理”是对现实世界的近似，需要满足三个条件：

- 基于微平面（Microfacet）的表面模型
- 能量守恒
- 应用基于物理的BRDF



## 微平面理论

概念：将物体表面建模成做无数微观尺度上有**随机朝向**的理想**镜面反射**的小平面（microfacet）的理论

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-1.png)

![TA100T5110-23](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-23.png)



## 能量守恒

概念：出射光线的能量永远不能大于入射光线的能量

表现：随着粗糙度的上升，镜面反射区域的面积会增加，作为平衡，镜面反射区域的平均亮度则会下降

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-2.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-3.png)

### 如何能量守恒

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910151314019.png)

![TA100T5110-4](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-4.png)

简单理解就是：

反射光的强度 = 入射光的强度 * 反射比例 + 入射光的衰减

最终出射光的强度 = 反射光的强度 + 自发光的强度

入射光的衰减 = 半角向量和法线的点积

【唯一的难点就是知道**反射比例**是多少】

### BRDF -- 计算反射比例

Bidirectional Reflectance Distribution Function，双向反射分布函数。“双向”意为相机方向和光源方向调换之后，他们所计算出来的能量等级是一致的

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-5.png)

BTDF：双向透射分布函数，用于描述光线透过物体的表现，其也是包含高光反射和漫反射

**BSDF = BRDF + BTDF（BSDF双向散射分布函数）**

BRDF![TA100T5110-24](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-24.png)

BRDF函数作用是：通过入射光的方向和出射光的方向，得到反射比例

## 应用基于物理的BRDF

### BRDF的计算

一般将反射拆分为漫反射和高光反射：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-6.png)

### 漫反射

算法基本采用Lambert光照模型（经验模型）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-7.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910151549710.png)

- Kd：漫反射系数，与高光反射系数Ks加和为1
- （I / r ^2）：描述光线能量的衰减

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-8.png)

### 高光反射

- 经验模型

  - Phong模型

  - Blinn-Phong模型

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-9.png)

- 基于物理的高光反射：Cook-Torrance反射率方程

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910151657659.png)

#### D：NDF法线分布函数

“Normal Distrubution Function”

GGX算法：更接近于物理

此处使用的是Trowbridge-Reitz GGX法线分布函数

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910151730565.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-12.png)

![绿色为GGX](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-13.png)

#### G：几何遮蔽函数

我们使用的微平面理论每个面的计算是互不干扰的，但实际中物体表面凹凸存在相互遮蔽的情况。

因此，几何函数从**统计学上近似的求得了微平面间相互遮蔽的比率**。这种相互遮蔽会损耗光线的能量。（除了被吸收，还有被自身遮蔽带来的能量损耗）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910151853240.png)

传统的几何项模型中，需要考虑光线方向和视线方向两种情况的遮蔽效应，所以最终写为：

![img](https://cdn.nlark.com/yuque/__latex/8f568f8403f69833e0b8fde074d3c597.svg)

使用史密斯法与Schlick-GGX作为Gsub可以得到如下所示不同粗糙度的视觉效果

![0：没有微平面阴影 -- 1：微平面彻底被遮蔽](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-15.png)

#### F：菲涅尔方程

被反射的光线对比光线被折射的部分所占的比率（物体的边缘的更亮一些）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910152002086.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-17.png)

### 总结

**BRDF的核心算法是Cook-Torrance反射率方程**

![img](https://cdn.nlark.com/yuque/__latex/7bbad6903073c85946379ebb8a7aec5f.svg)

漫反射：Lambert

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-18.png)

D：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-19.png)

G：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-20.png)

F：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-21.png)



## URP中的PBR

Specular工作流的高光颜色由Specular Map控制；而Metallic工作流的高光颜色与Base Color相同

直接看源码就好，顺着URP默认材质的Pass，找到Lit.shader和Lighting.hlsl研究就行



## 迪士尼原则的BRDF

- 使用直观的参数，而不是物理类的晦涩参数
- 参数应尽可能少
- 参数在其合理范围内应该为0到1
- 允许参数在有意义时超出正常的合理范围
- 所有参数组合应尽可能健壮和合理

![用了11个参数即可非常真实地模拟出金属、非金属以及不同粗糙度的材质光照结果](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T5110-22.png)

