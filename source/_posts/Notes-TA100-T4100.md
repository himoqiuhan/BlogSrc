---
title: 技术美术百人计划学习笔记（图形4.1 Bloom算法）
date: 2023-08-11 11:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 初级Bloom的基础实现原理
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100.png)

## Bloom算法

### Bloom算法介绍

#### Bloom的效果

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-1.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910142602489.png)

Bloom，也称辉光效果。用于模拟摄像机的一种图像效果（光向周围扩散的效果，原理是高亮的值使得数码相机的传感器饱和，并且**泄漏到临近的传感器单元**），让物体具有真实的明亮效果



### 实现思路

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-2.png)



### HDR与LDR

LDR：Low Dynamic Range，低动态范围

- JPG、PNG格式图片
- RGB范围在[0,1]之间，造成精度丢失

HDR：High Dynamic Range，高动态范围

- HDR、EXR格式图片
- RGB范围可超过1，提取亮度大于1的区域作为辉光区域（在LDR中，一些不应该有Bloom效果的区域也可能因为亮度过高而带有不合适的Bloom效果，比如如下的地面反射的区域不应该有Bloom）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-3.png)



### 高斯模糊

一种图像模糊处理方法，目的是减少图像噪声、降低细节层次。通过高斯函数得到的高斯核，对图像信息进行卷积运算。

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910142750916.png)

【计算高斯核】

核中心(0,0)，核大小3x3，标准方差σ为1.5

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-5.png)

【二维高斯核的优化】

二维高斯核计算量大，需要N\*N\*W\*H次纹理采样

但是二维高斯核具有可分离性，可拆成两个一维高斯核，横竖方向分别进行两次高斯模糊，只需要2\*N\*W\*H

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-6.png)



## Bloom算法实现

思路：

- C#：URP需要用Render Feature增加Pass，Built-in管线调用OnRenderImage函数。核心就是处理图像存为RT，再将RT传给shader作为处理的源图像进行下一步处理。
- shader：使用4个Pass完成Bloom效果（Pass1用于提取亮度，Pass2和Pass3进行不同方向的高斯模糊，Pass4用于混合），**实际上在脚本流中将uv偏移值传入shader可以用一个Pass完成处理**
  - 提取亮度时，用相减做clamp比用step更省性能，因为clamp只需要进行一次比较，而step需要进行两次比较



## Bloom的应用

### 配合自发光

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-7.png)

### 配合特效

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-8.png)

### 实现GodRay效果

实现思路与Bloom效果类似，通过一个设定好的亮度阈值去提取原图像中比较亮的区域，区别在于不用高斯模糊进行模糊处理，而是使用径向模糊Radial Blur，对提取后的图像进行多次模糊处理来模拟光线扩散的效果，最后进行图像混合

![=](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-9.png)

### 配合Tonemapping

以下两图的右侧都是使用ACES模式的色调映射效果，配合了ACES模式的Bloom效果饱和度偏低，视觉上更加柔和，较好的保留暗部和亮部的细节

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-10.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4100-11.png)

