---
title: 技术美术百人计划学习笔记（图形2.8 Flow Map流动效果实现）
date: 2023-06-14 11:52:27
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: Flow Map流动效果实现
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true 
---

![【图形渲染】 2.8 Flow Map](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800.png)

## Flowmap是什么

#### 基本概念

Flowmap实际上是一张记录了2D向量信息的纹理，flow map上的颜色（通常为RG通道）记录该处向量场的方向，让模型上某一点表现定量的特征。

通过在shader中偏移uv再对纹理进行采样，来模拟流动效果

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-1.png)

![TA100T2800-2](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-2.png)

使用flow map干扰uv坐标

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-3.png)



#### 为什么要使用flowmap

flowmap类似于UV动画，而非顶点动画。换而言之，无需对模型进行操作，易实现，运算开销小

不仅仅是水面，任何和流动相关的效果都可以采用flowmap

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-4.png)

所以flowmap是一种简单高效的用于实现流动效果的方法

flowmap还可以被用于天空球上

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-5.png)



## Flowmap shader

- 采样Flow map获得向量场信息

  - 得到的是将来用于乘time的方向信息
  - flowmap不能直接使用，要将色值从[0,1]的范围映射到方向向量的范围[-1,1]

  ```glsl
  //从flowmap中获取流向
  float3 flowDir = tex2D(_FlowMap, i.uv) * 2.0 - 1.0;
  ```

- 用向量场信息，使采样贴图的UV随时间变化

  - UV - time
  - 为什么是用减法（加上一个为负数的时间）：用减法从视觉上来看，采样得到的图象是在向右移动，与直观的运算法则相同

- 对同一张贴图以半个周期的相位差采集两次，并线性插值，使贴图流动连续

  - 随着时间的进行，变形越来越夸张，为了将偏移控制在一定范围内，使用frac函数进行牵制

    ```glsl
    float phase = frac(_Time);
    ```

    ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-6.png)

  - 这样得到的效果会“跳变”，所以就构造周期相同，相位相差半个周期的波形函数

    ```glsl
    float phase0 = frac(_Time * 0.1 * _TimeSpeed);
    float phase1 = frac(_Time * 0.1 * _TimeSpeed + 0.5);
    ```

    ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-7.png)

  - 用相位差半个周期的两层采样进行加权混合，使纹一个周期重新开始时的不自然情况被另一层采样重新覆盖

    ```glsl
    //平铺贴图用的uv
    float2 tiling_uv = i.uv * _MainTex_ST.xy + _MainTex_ST.zw;
    
    //用波形函数周期化向量场方向，用偏移后的uv对材质进行偏移采样
    half3 tex0 = tex2D(_MainTex, tiling_uv - flowDir.xy * phase0);
    half3 tex1 = tex2D(_MainTex, tiling_uv - flowDir.xy * phase1);
    
    //构造函数计算波形函数变化的权值，使得MainTex采样在接近最大偏移时有权值为0，并因此消隐，构造较平滑的循环
    float flowLerp = abs((0.5 - phase0) / 0.5);
    float3 finalColor = lerp(tex0, tex1, flowLerp); 
    
    ```

    ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-8.png)

    ![有效缓解突变](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-9.png)

    

    我的理解：因为tex1比tex0慢半个周期，所以当tex0的偏移采样接近1时，为了降低由0→1的突变，在[0.5,1]的区间上，逐渐降低tex0的权重；与此同时，tex1不会在1时发生突变，tex1的权重则在慢慢增加。这样就很好地规避了偏移达到最大的时候跳变的问题。

    同理，flowmap可以用于调整法线贴图的采样



## Flowmap的制作

#### flowmap的烘培和相关设置

- 注意gamma矫正选项、UV匹配
- flowmap贴图设置：
  - 无压缩或高质量
  - 确认色彩空间

#### Flowmap Painter

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-10.png)

- 使用Flowmap Painter绘制得到的Flowmap为线性空间下的颜色（gamma1.0），不需要gamma矫正，取消勾选sRGB
- 否则会出现“没画的地方在流动”，“画的地方流动方向错误”等问题

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2800-11.png)

#### Houdini Labs

Houdini Labs是内置在Houdini中的一组游戏开发相关的节点

//等到之后要用在项目里的时候再来学吧，不然现在直接学很快也就忘了

提前记录几个关键节点：

- flowmap：生成一张flowmap
- 修改flowmap的向量场：
  - flowmap_brush：手动绘制flowmap
  - flowmap_guide：使用绘制的曲线来控制flowmap（需要输入一条曲线）
  - flowmap_obstacle：让流向与物体进行碰撞检测（需要输入一个网格）
- flowmap_to_color：将flowmap向量场信息转化为颜色信息
- flowmap_visual：可视化flowmap
- maps_baker：烘焙（一个用于高模烘低模的节点）
