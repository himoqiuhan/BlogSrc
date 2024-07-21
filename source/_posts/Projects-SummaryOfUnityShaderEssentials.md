---
title: 【Unity】《Unity Shader 入门精要》学习总结
date: 2023-04-01 00:00:04
tags:
 - Unity
 - Built-in管线
 - Shader
 - CG
 - 技术美术
categories: Portfolio
keywords: 'Unity Shader入门精要'
description: 入门精要里的效果全部都动手实现了一遍，用这一篇总的记录一下学习这本书写出来的效果
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/100789898_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/41TN3CwhyZL._SY346_.jpg
---

> 用这一篇总的记录一下学习这本书写出来的效果，具体的学习笔记见此：[《UnityShader入门精要》学习笔记](https://himoqiuhan.github.io/tags/《UnityShader入门精要》/)

## 总览

![效果总览](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902102643500.png)

## 基础光照模型

![Basic Lighting](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902093754351.png)

- 逐片元的光照计算
- 逐顶点的光照计算
- Blinn-Phong着色模型
- 逐顶点的高光反射计算

## 纹理的使用

![Basic Texture](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902094030537.png)

- 基础的纹理使用
- 法线贴图在切线空间和世界空间下计算光照
- Blinn-Phong光照模型结合贴图使用
- Ramp图的使用

## 前向渲染

![Forward Rendering](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902094235305.png)

- Alpha Blend实现半透明效果
- Alpha Test实现透明效果
- built-in中前向渲染的多光源

## 环境贴图

![Cubemap](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902095404409.png)

- 基于Cubemap的菲涅尔效果
- 基于Cubemap的环境反射
- 基于Cubemap的环境折射

## 基于RT的镜面反射

![Mirror](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902095748936.png)

- 基于摄像机动态更新RT实现镜子效果，更改C#代码使得镜子前后移动会更改镜面内容缩放（透视）

## 玻璃折射

![Glass](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902100142866.png)

- 使用GrabPass，通过法线扰乱贴图读取模拟玻璃折射效果；使用Cubemap模拟玻璃反射效果

## 顶点动画

![Vertex Animation](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902101650408.png)

- 顶点动画（需重新ShadowCaster）

## BillBoard

![BillBoard](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902101841164.png)

- 永远面向摄像机
  - 通过重构向前的方向来重构顶点位置，使得模型看上去始终面向摄像机

## 溶解效果

![Dissolve](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902102220207.png)

- 通过噪声来对片元渲染进行Clip，实现溶解



