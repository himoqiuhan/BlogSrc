---
title: 【Cocos】《北门公仆》游戏介绍
date: 2022-06-28 15:49:39
tags:
 - 游戏开发
 - Cocos2dx
 - TypeScript
categories: Portfolio
description: 大一游创结课作业、微信小游戏开发二等奖
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BMGP_Alley.JPG
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BMGP_Title.PNG
---

## 开发者信息及游戏链接

- 策划：楸涵、一帆
- 程序：楸涵、一帆、董佩航
- 美术：SOMA
- PM：楸涵

> 游戏体验链接（微信二维码）

![游戏体验微信二维码](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/b876d438effcf943fe3ba61ec1457b3c5d7d84d4.jpg@1036w_!web-dynamic.webp)

> 展示视频：【《北门公仆》——大一游创结课作业】 https://www.bilibili.com/video/BV1zW4y1z7Lq



## 游戏截图

![部分游戏内截图总述](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902160708008.png)



## 游戏介绍

- **主题：**复兴之路展品100个故事
- **理解：**通过游戏的形式，向大家讲述李公朴先生的故事

### 游戏类型

以情节为核心驱动的分关卡拼图解谜游戏

### 游戏目标

完成拼图复原并观看历史事件

### 游戏玩法

玩家通过在不同条件下完成拼图的构建来解锁剧情故事，并在特定时间段做出自己的选择，在不同的视角下观看李公朴先生在民国时期的故事



## 开发过程

> 因为是比较久之前做的游戏了，记录下的更多是基于开发时留下的资料对开发的回顾

在概念设计之初，我们为这个游戏确定的方向是：以情节为核心驱动的分关卡拼图解谜游戏，玩家需要在一定的限定条件下完成拼图内容，复原历史的场景和物件，解锁李公朴先生的故事经历

![真实的历史故事](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BMGP_History.png)

我们剧情设计的核心在于，构建了一个见证故事的普通人，以一个见证伟人历史的小人物的视角展开故事叙述，由一点开头发散到多段if线分支，最终再收束到一个结局进行整体故事构建。这样的处理方式让我们能够在故事上处理得更加灵活，并且也带来了一定的复玩性——从不同的视角看这一段历史

![《北门公仆》游戏剧情大纲](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BMGP_Story.png)

基于这个故事发展线，我们先是用ppt初步写了以下分镜，确认流程无误后才开始的引擎内部制作

![剧情内容分镜](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902163753681.png)

关于玩法，我们在简单的九宫格拼图的原型上，添加了时间限制、步数限制等条件，在不同的关卡内有不同的条件，不同的执行结果也会引导向不同的故事结局

![部分拼图的原型](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902164103721.png)

资源整合大纲：

![资源整合](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902164241375.png)
