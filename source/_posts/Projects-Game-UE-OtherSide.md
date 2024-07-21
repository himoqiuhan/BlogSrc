---
title: 【UE4】《OtherSide》游戏介绍
date: 2022-01-23 10:34:42
tags:
 - 游戏开发
 - UE4
 - 蓝图
categories: Portfolio
description: GlobalGameJam 2022参赛作品，镜像翻转解密游戏
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GGJ2022.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GGJ2022.jpg
---

## 开发者信息及作品展示

- 策划：曾經、
- 程序：楸涵
- 美术：Dr.Schmidt
- 动画：啦啦哈哈哈啊 
- 项目管理：wyf112

> 展示视频：【GGJ2022参赛作品Other Side】 https://www.bilibili.com/video/BV13q4y1w7Dd



## 游戏截图

![开始界面](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/otherside%E6%B8%B8%E6%88%8F%E6%88%AA%E5%9B%BE1.jpg)

![游戏内截图](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/otherside%E6%B8%B8%E6%88%8F%E6%88%AA%E5%9B%BE2.jpg)

![美宣](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/othersideEnd.png)



## 游戏介绍

- 主题：duality
- 游戏名称解释：OtherSide，现实的另一端，也是镜中的世界，但是究竟镜中的时间和当前的世界哪个是真实世界呢？我们会不会其实是在镜中世界，只是一个镜像呢？并且，你所看到的，只是镜中世界的一角，没有被镜子展现出来的世界一定和你所想象的一致吗？游戏名称OtherSide昭示着主要玩法，玩家需要结合两个世界的信息，去到这个世界的OtherSide（另一端），来寻求到问题的解决办法

### 游戏类型

解密游戏、推箱子

### 游戏目标

玩家需要将能源块拖放到能源接收区，打开下一扇大门，逐步发掘镜中世界的真相

### 游戏玩法

镜子内外的世界构造看似相同，实则暗藏玄，唯有被清洁能源包裹的能源方块在世界中的位置是一定真实可信的。玩家需要观察记录不同世界下的地理构造，在不同的世界推动能源块，最终将能源块移动到能源接收处，打开大门，最终将清洁能源块带到世界的能源核心之中，发掘镜中世界的真相，净化并拯救两个世界



## 开发过程

> 因为是比较久之前做的游戏了，记录下的更多是基于开发时留下的资料对开发的回顾

我们游戏的核心元素是**推箱子**和**镜面**，玩家可以穿梭于镜内镜外两个空间，有镜子的地方二者同步，无镜子的地方二者不同步。

关于美术风格，我们想要的是像饥荒那样2.5D视角的效果，但是因为小组内成员都是第一次正式做游戏，受于技术力的限制，我们只在角色上做出了2.5D的效果，以一系列的帧动画和一张面片作为我们的主角，场景则是用3D的方块进行堆砌

![美术风格参考](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902153051362.png)

> 策划方面的构思，目前我只找到了初步的关卡构思，Level2和Level3的资料暂时还没有找到

Level1首要的是引入镜面机制，让玩家初步了解到游戏玩法，并通过两次简单的位置转移就能通过。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902153458088.png" alt="Level1的关卡平面图" style="zoom:80%;" />

在Level2中，我们引入了“被遮挡”这个信息，也就是被挡住的地方是可能会有能源块的，将能源块推进镜子中可以实现能源块在不同世界中的生成

![没有进入镜面区域的能源块会被隐藏](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902154128844.png)

在Level3中我们则是增加了充能机制，玩家需要先将空的能源方块推至充能区进行充能，然后才能进行能源输送

![需要对能源块进行充能](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230902154440040.png)
