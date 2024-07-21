---
title: 技术美术百人计划学习笔记（图形4.5 DOF景深算法）
date: 2023-08-11 14:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: DOF景深效果的基础实现
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500.png)

## 什么是景深

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-1.png)

景深的原理：离散圈

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-2.png)



## 景深的作用

选择性的**突出或者强调**画面中的一部分，例如某个物体或者某个人物，吸引观察者的注意力到画面中清晰对焦的部分，而忽略其他的模糊部分的细节。

强调所拍摄**场景的深度**，增加画面的**层次立体感**。

**艺术意境**的表达。摄影师可以利用景深效果，营造出虚幻、梦境、或者神奇等意境。表示主观的视线。

在电影学中，通过调节浅景深的镜头，使之对焦在不同位置上，来表示某个人的主观**视线的转移**，**交代人物之间的关系**。在电影学中，通过景深聚焦位置的变化，来表达前景和背景人物之间的关系。



## 景深的制作

### 制作思路

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-3.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-4.png)

### 截取景深区域

【深度图获取】

先获取深度图，如果需要让远裁剪面不影响实际的景深效果，则需要对读取深度图后的深度值进行处理：乘上ProjectionParams.z

```glsl
depth = Linear01Depth(tex2D(_CameraDepthTexture, i.uv)) * _ProjectionParams.z;
```

【通过深度值提取景深范围】

定义一个_FocusDistance和一个_DepthOfDield，景深的范围就是[_FocusDistance - _DepthOfField, _FocusDistance + _DepthOfField]

通过景深值与深度值进行比较可获得景深的模糊区域遮罩

```glsl
if(depth < focusNear)
{
  final_depth = saturate(abs(focusNear - depth) * _SmoothRange);
}
if(depth > focusNear)
{
  final_depth = saturate(abs(depth - focusFar) * _SmoothRange);
}
```

### 模糊处理与贴图合并

与高斯模糊、bloom效果制作相似



## 高级景深效果思路拓展

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-5.png)

- (a) p在背景区域
- (b) p在前景区域
- (c) p在聚焦区域

不同区域可以使用不同滤波，甚至使用不同滤波方法

### 颜色泄露缺陷

在对后处理的对焦区域之外进行模糊处理的过程中，将模糊的背景色叠加在聚焦区域之上，或者前景聚焦区域的颜色混合到了模糊背景之中（类似于Bloom的“扩散”效果）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-6.png)

为了解决这个问题，可以使用**扩散滤波**

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-7.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-13.png)

原理是将每个像素点的颜色扩散到这个像素点的**模糊圈范围中**，这样由于聚焦区域以外的像素有大量的模糊圈，所以被模糊了；而聚焦区域的模糊圈直径小于一个像素，所以颜色就不会扩散，保持清晰

![左为原图，右为通过扩散滤波得到的结果](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-8.png)

### 模糊的不连续缺陷

产生原因是焦点的像素为零，而前景区域大于0

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-9.png)

解决方法：单独取出前景和后景进行区分，把前景进行单独的模糊处理，然后再和背景进行融合



## 散景的模拟（Bokeh）

**焦外成像**

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-10.png)

在背景滤波的基础上可以通过**点函数来模拟散景效果**

![一个单色光点光源在不同的参数下成像](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-11.png)

![(a): 原始图像 (b): 真实相机的景深模糊效果 (c): 高斯分布的点扩散函数的景深模糊效果(d): 均匀分布的圆形点扩散函数的景深模糊效果 (e): 均匀分布的六边形点扩散函数的景深模糊效果](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T4500-12.png)



## UE在18年的景深效果

https://epicgames.ent.box.com/s/s86j70iamxvsuu6j35pilypficznec04

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230910144904055.png)
