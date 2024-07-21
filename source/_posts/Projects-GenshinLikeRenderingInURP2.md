---
title: 【Unity】URP中的仿原神渲染2.0
date: 2023-11-22 20:06:16
tags:
 - Unity
 - URP管线
 - 卡通渲染
 - 技术美术
categories: Portfolio
description: 在URP管线下探究原神的卡通渲染实现，对1.0从效果、代码架构上进行迭代
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/73410871d211425732b5205f15856be.png
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/c29cb1345f98a9f92545947d81d692c.png

---



## 效果展示

> 展示视频：【【卡通渲染/MMD】五等分の気持ち】 https://www.bilibili.com/video/BV1ew41187H5/



![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/73410871d211425732b5205f15856be.png)

![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/c29cb1345f98a9f92545947d81d692c.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/cac5b96ce08adf526486128c49df132.png)

![夜间、白天及加入Bloom后的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204162907101.png)

![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2_Character_GeneralShow.gif)



## 介绍

对之前制作的仿原神渲染的进一步优化，相比于仿原神渲染1.0，这次着重升级了：

> 省流一下：优化角色渲染、加入自定义的场景渲染管线、更好看的Bloom、更清晰的代码架构

- 角色部分
  - 【漫反射】面部刘海投影
  - 【漫反射】适配多样化Diffuse贴图使用方式
  - 【漫反射】更灵活更美观的Ramp处理方式
  - 【高光】基于对Mat Cap新理解的Blinn Phong + Mat Cap组合高光
  - 【边缘光】更灵活可控的Rim Light
  - 【描边】区分区域可控的描边颜色
  - 【描边】更平滑的描边表现（优化平滑法线计算方式）
  - 【流程优化】更灵活的顶点色使用方式
  - 【代码架构优化】更加清晰的代码架构

- 场景部分
  - 自定义的一套简易、灵活的渲染管线，可以实现：多光源的光照处理、单个方向光的Shadow Mapping、Baked Lightmap、PBR材质、PBR光照模型（Phong + Cook-Torrance）
  - 十分清晰的代码架构（详见思维导图和文档）

- 后处理部分
  - 效果更好的Bloom
  - 更加清晰的代码架构



## 思维导图（图片较大，建议下载查看）

![仿原神渲染2.0渲染部分的思维导图](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2-ShadingPart.png)



## 文档指路

- 代码仓库：https://github.com/himoqiuhan/URPToonRenderReconstruction

- 渲染部分文档
  - 角色渲染：https://zhuanlan.zhihu.com/p/670601962
  - 后处理：https://zhuanlan.zhihu.com/p/670604475
- MMD-Blender-Unity MMD制作流程经验分享：（Updating...）

