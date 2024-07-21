---
title: 【UE5】回合制卡牌游戏《失恒Diseternity》
date: 2023-07-09 15:37:03
tags:
 - 游戏开发
 - UE5
 - 蓝图
categories: Portfolio
description: 大二游戏创作专业课结课作业，一个回合制的卡牌游戏
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Diseternity.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Diseternity.jpg
---

## 开发者信息及游戏链接

- 策划：曾經、一帆
- 程序：楸涵、6Д9
- 美术：SOMA、子珏、二呈

> 游戏链接：https://pan.baidu.com/s/1TbsCvCp66PSYo-kFlkktuQ?pwd=3jj4 
> 提取码：3jj4 

> 展示视频：【【前瞻预告】Rogue卡牌游戏：《失恒》（Diseternity）】 https://www.bilibili.com/video/BV17c411A7wP



## 游戏截图

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709154847462.png" alt="主界面" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709160036364.png" alt="关卡选择" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709160227027.png" alt="卡牌战斗" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709155135109.png" alt="角色对话" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709155207786.png" alt="角色获取" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709160330398.png" alt="角色删除" style="zoom: 40%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709155410847.png" alt="卡牌获取" style="zoom: 40%;" />



## 游戏介绍

- 主题：永生的代价

- 游戏名称解释：Diseternity失·恒，意为失去永恒，dis前缀和eternity永恒，连接而成

### 游戏类型

Roguelike类，卡牌战斗，DBG（deck building game）卡组构筑类游戏

### 游戏目标

不断前进构筑自己的队伍（增删减查），击败敌人，了解真相，做出抉择

### 游戏玩法

- 以卡牌随从战斗为核心，通过控制自己的资源，放置随从，打出卡牌，成长随从，辅助战斗来达到消灭敌人的目标。

  - 战场之上共有9个格子，玩家与病毒共享这些格子，病毒源会周期性的召唤病毒来充斥这个战场，若是病毒占满所有格子，那么游戏结束。玩家需要打出随从站定一个位置，并且用卡牌强化随从的属性和辅助战斗运转让他来消灭病毒和病毒源并且确保他不会被敌人所击败失去所占位置。
  - 在玩家游玩的过程中会遇见各式各样前世有血有肉的人以计算机数据类型为载体的角色（如主角int，猫猫float等）他们身怀绝技，是冒险路上的强大助力，但是随从的数量与整个世界的能量挂钩（存在上限），玩家需要在不断前进的道路中不断选择放弃牺牲之前获得的随从来确保整个世界不会崩塌，所以牺牲队友换来永生世界的维持，换来继续前进的可能。

### 背景设定

- 世界观

  - 电子意识上传，在一个机房里消耗着**号称无尽的资源**，人们实现了所谓的永生。但是这背后有这一个**骗局**，无尽资源只够少数人消耗，激活使用的电脑越多，资源消耗的越快，若是人数过多，那么资源和意识将面临崩溃。（对标随从数量上限的设定）

    终局涉及到无我概念，多世界的跳脱与冲突矛盾，对于永生的代价的不同答案。

- 剧情与反派

  - 知情的上位者为了达成自己的永生（其实是为了拖缓全局崩坏进程），在全体人类带入这个意识机房之后开始了他的计划，他打算利用病毒一步一步关掉下层人民的机子，减少资源的消耗。

- 主角/玩家任务

  - 随着病毒的侵蚀，愣头青主角（初始不知道这个能源问题）身边的电脑被一个一个关停，主角意识到事情的严重性，他瞧准时机切入到了一个即将被病毒拿下的电脑中，联合这个电脑里的“人”将病毒击退，赢得他的信任，打算去联合大家一起拯救那些被病毒残害而关机的人并且去“守护”这个我们来之不易的永生家园。

- 矛盾点

  - 矛盾点在于发现了资源不是无尽的情况下，主角如何**抉择**
    - **直接告诉**手下的人们真实情况 
    - 编一个**借口**遣散一些人继续前进试图寻找到解决问题的方法
    - **关闭自己**让他人生存
    - 成为那个上位者



## 游戏开发

### 分工

组内共7名同学，楸涵（我）、SOMA、子珏、二呈、曾經、一帆、6Д9

- 我：程序（主要逻辑框架，除战斗系统外各个子系统的全部开发，战斗系统卡牌、角色相关功能开发）、技术美术（交互反馈效果）
- 6Д9：程序（战斗系统中角色及病毒的生成、攻击、技能开发）
- 曾經：策划（战斗系统、卡牌系统、角色系统、关卡系统）
- 一帆：策划（数值调试、剧情系统、音频、背景设定）
- SOMA：美术（原画设计、卡牌牌面绘制、背景绘制、UI绘制、宣传图设计）
- 子珏：美术（原画设计、角色设计、角色帧动画制作）
- 二呈：美术（宣传图设计、部分角色制作）

### 开发过程

我们使用了Notion这个软件作为多人协同的开发辅助工具，极大地方便了我们进行开发管理以及一些资料的整理与共享

![部分开发截图-看板](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709162915870.png)

![中期邀请玩家试玩后的反馈总结](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709163033084.png)

由于此次我们是两名程序进行开发，所以notion也很大程度上帮助我们处理多人协同的问题，大大降低了我们merge时产生冲突的概率

![总计250+小时的开发时间最后看来也很有成就感！](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709163326261.png)

虽然但是还是因为一些失误有过一两次冲突

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/b70e89011192f319992b5a75ac547f6.jpg" alt="QAQ" style="zoom: 50%;" />

## 收获

这次开发是我第一个长线开发的项目（前前后后一共做了接近四个月），自己也摸爬滚打出了一些对于长期开发项目的经验：

1. 一个系统的架构思路一定要写下来，因为你永远都不知道什么时候需要回去看自己以前写的屎山

   <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230709165053013.png" alt="拆解系统的部分笔记" style="zoom: 33%;" />

2. 一些系统偏向底层的功能一定要在一开始就和策划同学沟通清楚，不然你永远不知道一个系统要重写几遍

3. Git是一个很好的版本管理工具，为了便于组员快速上手还写了一点小入门教学文档：https://zhuanlan.zhihu.com/p/615928723

4. 开发过程最好写一些关于变量和函数用途的文档，否则变量多了起来很容易出现同一个信息有很多个变量在不同的地方存储的问题



总之，做游戏很好玩！

然后，能遇到一群想做一个好游戏的队友一起做游戏更好玩！











