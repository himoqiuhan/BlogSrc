---
title: 【UE5】UE5 Niagara特效
date: 2023-05-25 21:24:13
tags:
 - UE5
 - Niagara
categories: Portfolio
description: UE5的Niagara特效系统学习展示
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/HighresScreenshot00000.png
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20230903092046.png
---

> 这里是制作的效果总览，学习笔记和详细的制作思路请看这里：

## 火焰喷射器

![火焰喷射器](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Firefire1.gif)

- 底部蓝色到紫色再到橙色的过渡火光
- 橙色火光主体部分
- 尾部烟雾
- 热扰动效果

![火焰喷射器的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903083019374.png)



## 传送门

![Portal1](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Portal1.gif)

- 边缘转动飘舞的粒子群
- 内部向外扩散的火焰
- 中心的黑洞
- 漩涡图案
- 飞入传送门的岩石块

![传送门的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903083742940.png)



## 陨石攻击(特效ACD三阶段)

![陨石攻击](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/MeteorAttack1.gif)

- Anticipation准备阶段
  - 下坠的陨石
  - 陨石外的火焰
- Climax高潮阶段
  - 烟雾
  - 火光
  - 炸开的碎石
- Dissipation消散阶段
  - 地表痕迹

![陨石攻击的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903084124752.png)



## 魔法阵攻击

![魔法阵攻击](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/EnergyStrike1.gif)

- Anticipation准备阶段
  - 闪电
  - 闪电击中特定位置留下的痕迹
  - 魔法阵
- Climax高潮阶段
  - 能量火焰
  - 中心光
- Dissipation消散阶段
  - 魔法阵停止旋转并逐渐消失

![能量攻击的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903084920453.png)



## 剑气攻击(蓝图控制Niagara声明周期)

![剑气攻击](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BladeSlash1.gif)

重点在于蓝图对Nigara的生命周期控制

- BP_Projectile_VFXSpawner：控制VFX的生成
- BP_Projectile_Master：控制VFX本身
  - Main Projectile VFX（Niagara Component）
  - Projectile Movement Component 编辑器自带的专门用于处理projectile的组件（Component）
  - Muzzle VFX（Niagara System Obj Ref）
  - Hit Impact VFX（Niagara System Obj Ref）
  - Spawn Logic 控制什么时候生成Muzzle VFX、Hit Impact VFX，Projectile该动多快等信息（Begin Play/Begin Overlap）

![BP_Projectile_Master](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BladeSlashMaster.png)

![BP_Projectile_VFXSpawner](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/BladeSlashSpawner.png)



## 下雨(自定义Niagara Module Script)

![下雨](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/NiagaraRaining1.gif)

- 区域内雨点效果
- 雨点落到地面的水花和涟漪
- 自适应坡面水花溅射方向及坡面不生成涟漪

![下雨特效的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903090159809.png)

![控制斜坡粒子方向和生命的Niagara Module Script](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903090253825.png)



## 链状闪电(自定义Niagara Module Script)

![链状闪电](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/NiagaraFollowing.gif)

- 漂浮球
- 闪击球体的粒子与Ribbon

![链状闪电的Niagara Graph](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903090617102.png)

![更新速度的Niagara Module Script](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903090957147.png)

![更新追踪目标的Niagara Module Script](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230903091110825.png)

