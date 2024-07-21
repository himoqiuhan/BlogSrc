---
title: 【Unity】自定义渲染管线学习笔记（Updating）
date: 2023-10-28 19:30:11
tags: 
 - TA
 - Unity
 - Render Pipeline
categories: Notes
keywords: 'Unity Custom Render Pipeline'
description: Unity自定义渲染管线学习笔记
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20231028192918.png
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20231028195012.png
katex: true
---

> 封面和头图均来自这个太太：https://www.pixiv.net/users/38297201

> Catlike Coding中Custom Render Pipeline的学习笔记，原链接在这里：https://catlikecoding.com/unity/tutorials/custom-srp/
>
> 代码链接（带注释）：https://github.com/himoqiuhan/CatlikeRenderPartition
>
> 尝试写了一点文档式的笔记，但是写下来后感觉，只是基于我不扎实的英语做的一个拙劣的汉化版，自己的理解不够深入。加之学习着学习着，愈发发现写管线就是个工程学的问题，更重要的是如何统筹业界已有的技术。所以在这一篇文章中持续更新我学习时写下的的思维导图，导图的框架基本是基于我写代码的框架，其中也有一些对Catlike上内容的补充，其中有不详细的内容可以去到对应的文章中查看。
>
> 注：原图过大可能无法正常显示，推荐右键下载查看



## Basic Custom RP

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/custom-render-pipeline/

简述：创建自定义渲染管线，并实现一些基础内容绘制，入SkyBox、Unlit物体等

![Custom Render Pipeline](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CustomRenderPipeline.png)



## Shaders And Batches

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/draw-calls/

简述：SRP环境下的Shader写法，以及Unity中的Batching合批技术，包含SRP Batcher、GPU Instancing、Static/Dynamic Batching

![Shaders And Batches](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/ShadersAndBatches.png)



## Directional Lights

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/directional-lights/

简述：自定义管线下的基础光照实现、与光照信息解耦的BRDF处理，以及自定义的材质GUI制作

![Directional Light](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/DirectionalLight.png)



## Directional Shadows

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/directional-shadows/

简述：基于Shadow Mapping的阴影实现，以及Cascade Shadow级联阴影、PCF软阴影的实现

![Directional Shadow](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/DirectionalShadow.png)



## Baked Light

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/baked-light/

简述：通过自定义的Meta Pass烘焙并采样含有Diffuse Reflectivity和Emission的Lightmap，采样Light Probe以及LPPV的原理概述

![Baked Light](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderBakedLight.png)



## Shadow Masks

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/shadow-masks/

简述：烘焙ShadowMask、采样ShadowMask并使烘焙阴影能够和实时阴影融洽融合

![Shadow Mask](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderShadowMasks.png)



## LOD and Reflections

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/lod-and-reflections/

简述：LOD部分利用Unity的LOD Group Component实现LOD，同时使用dither来确保LOD之间的切换平滑；Reflection部分增加IndirectBRDF的处理，环境反射暂时只支持对Skybox的反射

![LOD and Reflections](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderLODAndReflections.png)



## Complex Maps

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/complex-maps/

简述：在渲染管线中加入对HDRP中MODS(Metallic, Occlusion, Detail, Smoothness) Map、Detail Map、Normal Map的支持

![Complex Maps](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderComplexMaps.png)



## Point and Spot Lights

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/point-and-spot-lights/

简述：加入对Point Light和Spot Light的光照的计算，以及其Baked Shadow的计算

![Point and Spot Lights](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderPointAndSpotLights.png)



## Point and Spot Shadows

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/point-and-spot-shadows/

简述：同Directional Light一样使用Shadow Atlas来实现Point Light和Spot Light的实时阴影

![Point and Spot Shadows](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/CatlikeRenderPointAndSpotShadows.png)



## Post Processing (Updating)

原文链接：https://catlikecoding.com/unity/tutorials/custom-srp/post-processing/

简述：为自定义的渲染管线中加入后处理，并添加Bloom效果

......





> What Next：虽然现在说有点早，但是还是立一个志向：自己从零搓一个卡通渲染管线！在那个时候或许会同步写一些更多基于自己理解的文档出来















































































