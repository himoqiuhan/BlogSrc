---
title: 技术美术百人计划学习笔记（图形3.2 混合模式及剔除）
date: 2023-08-10 12:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 介绍Unity及PS中的混合模式，以及剔除相关的知识
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3200.png)

## 什么是混合模式

### 混合的定义

混合就是把两种颜色混在一起，具体就是某一像素原位置的颜色与将要画上去的颜色，通过某种方式或算法混在一起，从而实现新的效果。例如PS中的叠加、正片叠底、滤色

### 混合模式

**最终颜色 = Shader计算后的颜色值 \* ScrFactor + 累计颜色 \* DstFactor**

累计颜色可以理解为G-buffer中的像素，混合模式控制的就是ScrFactor和DstFactor

脚本中会看到的是`Blend SrcFactor DstFactor`



**混合模式是Material层面的控制**



## 混合模式的类型

### PS中的混合模式

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3200-1.png)

### ShaderLab的混合

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3200-2.png)

1. 如果颜色某一分量超过1，则会被自动截取为1，不需要考虑越界的问题
2. 在所有着色器执行完毕，所有纹理都被应用，所有像素准备被呈现到屏幕之后，使用Blend命令来操作这些像素进行混合
3. 语法：

- Blend Off：关闭Blend（默认）

- Blend SrcFactor DstFactor

  - 配置并启用混合

  - shader计算的颜色 * SrcFactor + 已经在target buffer的颜色 * DstFactor

- Blend SrcFactor DstFactor, SrcFactorA DstFactorA
  - 与上同理，但是使用不同的factor来混合alpha

- BlendOp Value
  - 设置Blend的计算公式，默认为Add

- BlendOp OpColor， OpAlpha
  - 同上，但是对Alpha进行不同处理

- AlphaToMaskOn
  - 常用在开启多重渲染（MSAA）的地表植被的渲染

官方文档：https://docs.unity3d.com/cn/2018.4/Manual/SL-Blend.html



### 总结

1. Blend命令：启用会禁用GPU上的一些优化（主要是隐藏表面去除Early-Z），会导致GPU开销增加
2. 混合操作默认为Add，如果使用BlendOp命令，则混合操作将设置为该方式
3. 混合方程：FinalValue = SrcFactor * SrcValue [operation] DstFactor * DstValue
4. 单独的RGB和Alpha混合与高级OpenGL混合操作不兼容

【常用非高级混合命令】：

- Alpha混合：Blend SrcAlpha OneMinusSrcAlpha

- Addtive相加混合：Blend One One

- 柔和相加混合：Blend One OneMinusDstFactor

- 相乘混合：Blend DstColor Zero

- 2倍相乘混合：Blend DstColor SrcColor

- 预乘透明度混合：Blend One OneMinusSrcAlpha



## 混合模式的实现方式

### Unity中使用自带枚举控制

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3200-3.png)

### PS混合模式实现方式

 【以下是最常用的】

- alpha：使用当前颜色的alpha值进行混合
- darken：对两个颜色乘1取两者的最小值
  - 语法：BlendOp Min，Blend One One

- Multipy正片叠底：当前颜色 * 缓存颜色 + 缓存颜色 * 0
  - 语法：Blend Add， Blend DstColor Zero

- Screen滤色：当前颜色 * （1 -  缓存颜色） + 缓存颜色 * 1
  - 语法：Blend Add，Blend OneMinusDstColor One

- Lighten变亮：对两个颜色乘1取最大值
  - 语法：BlendOp Max，Blend One One

- LinearDodge线性减淡：缓存颜色 * 1 + 当前颜色 * 1
  - 语法：Blend One One

- ColorBurn颜色加深：（高级OpenGL混合，只支持OpenGL平台）
  - 目前只在具有GL_KHR_blend_equaion_advanced或GL_NV_blend_equation_advanced扩展支持的OpenGL硬件上可用



## 剔除

### 法线剔除

也被称为**背面消隐**，根据法线朝向判断哪个面被剔除掉，可以控制是否双面渲染。

**也是Material层面的控制**

【语法】

`Cull + <Off/Front/Back>`（默认为Back）

【Unity内置的剔除模式的枚举】

`[Enum(UnityEngine.Rendering.CullMode)]_CullMode("Cull Mode", float) = 2`

### 面裁切

Clip函数会将参数小于值的片元直接在片元阶段丢弃掉，常用于制作溶解、裁剪等效果

【语法】

`Clip();`（默认会切掉0.5的部分）

【Clip背后的命令实际上是if】

```glsl
if (input.posInObjectCoords.y > 0.5)
{
  discard;
}
```

### 总结

1. 开启双面渲染（Cull Off）相当于绘制了两次--Overdraw
2. Clip函数在某些PowerVR的机型上效率很低
3. 面裁切Clip最好是使用AlphaTest的Queue（2450）
