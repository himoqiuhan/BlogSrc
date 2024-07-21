---
title: 技术美术百人计划学习笔记（图形3.3 曲面细分与几何着色器）
date: 2023-08-10 13:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 介绍曲面细分着色器与几何着色器，并介绍对两个着色器可配置阶段的控制
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300.png)

## 应用

### 曲面细分着色器

- 海浪、雪地等

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-1.png)

- 与DP贴图结合

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-2.png)

相较于直接增加模型的面数，曲面细分着色器可以在游戏进行时，根据自定义的规则（距离等），动态调整模型的复杂度，可以带来更好的性能。



### 几何着色器

- 几何动画

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-3.png)

- 草地（与曲面细分着色器结合，动态调整草地的疏密）

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-4.png)



## 着色器执行顺序

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-5.png)

Hull Shader：可编程，定义细分的参数

Tessellation Primitive Generator：不可编程不可控制

Domain Shader：曲面细分得到的点是在重心空间的，需要在Domain Shader中将点转换到我们要用的空间中



## TESS

### 输入和输出

- 输入：Patch，可以看作是多个顶点的集合，包含每个顶点的属性，可以指定一个Patch包含的顶点数以及自己的属性
- 功能：将图元细分（可以是三角形、矩形等）
- 输出：细分后的顶点

### TESS流程

- Hull Shader

  - 决定细分的数量（设定Tessellation Factor和Inside Tessellation Factor）

  - 对输入的Patch参数进行改变（按需）

- Tessellation Primitive Generation
  - 进行细分操作

- Domain Shader
  - 对细分后的点进行处理，从重心空间（Barycentric Coordinate System）转换到屏幕空间

### Hull Shader参数解析

- Tessellation Factor：决定将一条边分成几个部分，切分方法有：

  - equal_spacing：进行等分

  - fractional_even_spacing，fractional_odd_spacing：从两端开始细分，到中间部分逐渐平均。目的是为了让细分更平滑。两者区别在于，even一定在中点部位有一个点（点的数量是奇数），odd则没有中点（点的数量是偶数）

    ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-6.png)

- Inner Tessellation Factor：控制内部图形的切割方式

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3300-7.png)



## GS

### 输入与输出

- 输入：图元（三角形、矩形、线等），据图元的不同，shader中会出现对应不同数量的顶点
- 输出：输出同样为图元，一个或多个，需要自己从定点构建，顺序很重要；同时需要定义最大输出的顶点数

