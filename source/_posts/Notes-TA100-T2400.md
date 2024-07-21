---
title: 技术美术百人计划学习笔记（图形2.1-2.4）
date: 2023-06-14 08:48:56
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 颜色空间、模型与材质、HLSL常用函数、传统经验光照模型
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

> 因为前两章基础概念较多，所以放在一起整理了

![【图形渲染】 2.1 - 2.4 颜色空间、模型与材质、HLSL常用函数、](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2400.png)

## 颜色空间

### 色彩发送器

色彩认知：光源是出生点，光源发射出光线，光线通过直射反射折射等路径最终进入人眼。但人眼接收到光线后，人眼的细胞产生了一系列化学反应。由此把产生的信号传入大脑，最终大脑对颜色产生了认知感知。

#### 光的要素

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-28.png" alt="TA100-28" style="zoom: 80%;" />

#### 光源

光源就是产生的物体

#### 波长

光理论上讲是无限大的，只是我们人眼可见光是有局限的。

如果没有光，我们就无法在黑暗中看到色彩，光的本质就是一种**物理现象**，光在没有进入我们的眼睛前，我们对它的认知是一种**波长与能量分布**。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-29.png" alt="TA100-29"  />



#### 能量分布

我们讲光线是一种波，那么既然是真实存在的就会有能量，能量单位就是功率，我们认知的光就会有不同的功率。比如一个光是由多个波长组合起来的波形

那么也就是说我们阐述色彩就用这个波长就可以了，但是这么做实在是太反人类了，我们无法保证能简单描述色彩。于是人们发明了一个叫做**分光光度计**的东西。

![TA100-30](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-30.png)

#### 分光光度计

分光光度计就是用于描述光线的具体能量强度，记得小时候用棱镜分光吗。我们通过分光之后对区间波长进行了感应与测量，最后得知了光谱的分布最终得知光线额能量集中在了550nm附近（图中绿色地方）

![TA100-31](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-31.png)

#### 光的传播

- 直射光：光源直射眼睛

- 折射光： 光源穿过物体进入眼睛

- 反射光：光源经过物体表面反射进入眼睛

- 光线追踪：光线弹来弹去，然后我们根据权重确定光线最后进入眼睛中的颜色



下图就是光通过反射之后，在能量上发生的变化，可以明显看到，少了一部分的能量，这是因为一部分的能量被物体吸收了，也就是说每次光经过反射或投射都会或多或少对光的能量分布产生一些影响

![TA100-32](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-32.png)



### 光源的接收者

#### 相对亮度感知

在某些阴暗的环境下，点亮一盏灯，这时人眼就会觉得非常亮。如果同时点亮1000盏灯，反而觉得只是10倍的亮度，对亮度的认知相当于从0\~1再从1\~10.

![TA100-33](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-33.png)



#### 人眼HDR

人眼既可以分辨出高亮度的云彩的不同层次区别，还可以分辨出阴影中物不同物体的异同。

但是人眼的能力并**不能保证这两个功能同时生效**。

这样一说，反而就能发现人眼真是个变化莫测的存在，它可能会随着不同的环境，感知到不同的色彩，体验到不同的明暗效果，甚至可能会随着盯着某一个点时间流逝而变化颜色。



#### 人眼感光细胞分布

- 杆状细胞：感知亮度
- 锥状细胞：感知色彩

感光细胞（杆状细胞）对亮度特别的敏感，只要有5~14个光子打到杆状细胞就会产生神经信号，这也可以解释为什么闪光弹能让人致盲，一部分原因就是因为光实在太亮，直接干涉了人眼最敏感的感光细胞



#### 锥状细胞

这种细胞专门用于感知颜色，但是他们还被区分为了L细胞，M细胞，S细胞。

这三种细胞负责感知的波长不一，如下图所示，L感知红色区间，M感知绿色区间，S感知蓝色区间

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-34.png" alt="TA100-34" style="zoom:80%;" />



#### 人眼的本质

人眼的本质是光源的接收者。他的作用就是接收外部光线输入，输出神经电信号进入大脑。



#### 完整积分公式

![TA100-35](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-35.png)



### 色彩空间历史

#### 19世纪的猜想

1. 人们有100多种感受颜色的细胞
2. 人们有三种，分别是RGB三种感色细胞
3. 人们有三种，分别是黑白，红绿，黄蓝感色细胞（是不是很眼熟）

现在这么多年过去了，其中的2和3这两种猜想都成为了我们当下的色彩视觉模型，也称之色彩模型



#### 1905Munsell色彩系统

美国艺术家 Albert Henry Munsell利用自己的艺术特长，最早提出了一个色彩系统，后来在1930年被优化改良。

Munsell通过很多色卡来描述色彩，下面旋转角度的是色相，Munsell垂直的是亮度，从圆心到外部是Munsell饱和度。 人们凭借自我主观意识认知与区分色彩就是HSL(色相、饱和亮度)，这套系统没有过多的物理科学在其中，更多的是一种艺术家的理解与归纳总结规范…

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-36.png" alt="TA100-36" style="zoom:80%;" />



#### 1931 CIE 1931 RGB Color Specification System

科学家们觉得上述的色彩系统还可以，但是不够科学，于是为了以一种科学的方式阐述色彩，于是一个叫CIE的机构在1931年建立了一套色彩系统， 希望完全客观完全物理的量化色彩。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-37.png" alt="TA100-37" style="zoom:80%;" />

CIE把所有可视波长的光线作为测试光挨个测试了一个遍，最终的到了三条曲线我们发现435.8~546.1 nm这段波长中的红色基色强度是负数。这虽然物理正确，但是一点也没有科学的美感，于是我们进行了归一化，保证色彩在-1~1之间。最终通过计算出$rgb$的基色的强度在当前混色强度的所占比例这样计算后，$ r'g'b'$都是在-1~1之间

那么如果我们发现$r'+g'+b'=1$，就可以通过其中两个已知数计算出另一个的强度

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-38.png" alt="TA100-38" style="zoom:80%;" />

![TA100-39](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-39.png)



#### XYZ Color Specification System

上文的CIE1931RGB色彩系统已经不错了，但是存在负数，这在计算上非常的麻烦，比如写个乘法，得先计算是正数还是负数。

于是人们就用数学的方式做了一个新的色彩空间。所以XYZ色彩空间就是一个**中转站**，主要目的就是**简化计算**。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-40.png" alt="TA100-40"  />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-41.png" alt="TA100-41" style="zoom:80%;" />

- 如何转换：

因为是空间转换所以我们用**矩阵(进行空间变换)**的方式进行

注：这里的RGB是CIE 1931RGB 不是sRGB中的RGB数值。

这个$xyz$矩阵也不太美，于是人们为了计算方便有把$xyz$矩阵进行了**归一化**

- 最终效果：色域马蹄图

也就是人眼可见范围表示， 但是我们发现图像上面好像没有亮度于是我们就在归一化的基础上，把XYZ中的Y单独拿出来与$xy$一起组成了$Y_{xy}$色彩空间 其中的Y表示亮度$xy$表示色度。

注：这里提一下 这里是$Y_{xy}$色彩空间 **$Y_{xy}$是由$XYZ$色彩空间衍生出来的**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-42.png" alt="TA100-42" style="zoom:80%;" />

- 不足与补充

  上述的XYZ色彩空间也不错，但是也有问题，就是**色彩的分布不均匀**，他们的分布色彩一些地方紧一些地方又很松，举个例子这个图的偏向绿色部分就非常平滑，然后左下角部分坐标变化小，但是色彩变化就很快。

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-44.png" alt="TA100-44" style="zoom:80%;" />



### 色彩空间的定义

至少需要满足三项重要指标：

1. 色域（三个基色的坐标，由此形成三角形）
2. Gamma（如何对三角形内进行切分）
3. 白点（色域三角形中心）

![TA100-45](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614092133606.png)

#### Gamma

Gamma并不是色彩空间，它只是如何对色彩进行**采样的一种方式**

每次对比顶点切割，就会发现切割的方式不同会导致每次对应的色彩不一样，大家通常理解的gamma=1的情况就是指代**均匀的切分**（下左图），这样的好处就是方便计算。而非均匀切割的方式就是gamma≠1

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614092220077.png" alt="TA100-46"  />

比如我们有个常用的空间 sRGB，那么**sRGB的构成**：

1.色域： sRGB首先设定了RGB三个基色的坐标

2.白点： sRGB也规定了**白点位置**

3.gamma： sRGB的gamma设定为≈2.2（上右图）也就是说从外而向内切，**先切的很细，然后逐渐变粗**



大家知道线性的好处(也就是gamma=1的时候)，方便计算，计算机效率高，方便理解，但是计算机储存（不需要花费很多空间去存储亮的区域）与显示器硬件因为早期的性能问题，采用的基本大部分都是gamma≈2.2的情况，但是我们目前大部分的机器都已经不是远古版本了，所以PC上的大部分游戏都会推荐使用**线性空间**

我们可以自定义色彩空间，换一个色域，换一个白点位置，换一个gamma值其实就是一个新的色彩空间了，所以也可以存在sRGB D65 linear这类空间，所以任何色彩空间都可以是linear线性的（改gamma值就可以去确定），但linear本身并不是一个色彩空间，它只是一个gamma值



### 常用色彩模型与色彩空间

色彩模型：使用一定规则描述（排列）颜色的方法

色彩空间：需要至少满足三个指标：色域、白点、gamma

![TA100-47](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-47.png)

### 补充

#### 色彩空间下的各种色域

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-48.png" alt="TA100-48" style="zoom:80%;" />



#### P3色域相较于sRGB的提升

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-49.png" alt="sRGB视频的色彩被映射到了更低的部分，而P3完整显示了色彩" style="zoom:80%;" />

系统能正确映射sRGB色彩的原因在于系统有正确的色彩配置文件，选用不同的色彩配置文件会造成显示的色彩差异$\longrightarrow$美术师需要用校色仪来获取正确的色彩配置文件



#### 为什么需要Gamma矫正

常见Gamma值为2.2$\longrightarrow$测试得到的**符合大部分人的灰阶感知的Gamma值**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614092920518.png" alt="TA100-50" style="zoom:80%;" />



#### HDR

高动态范围

HDR显示器的峰值亮度通常在1000nit以上，同时保持非常高的对比度，远高于SDR显示器的亮度和对比度

HDR内容：观看HDR视频内容需要HDR显示器



#### 色彩空间转换

一个广色域HDR的视频（Display P3）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614093015670.png" alt="TA100-51"  />



### 作业

1. 色彩空间的定义：

   满足色域、白点、Gamma三个指标的对色彩的组织方式

2. 人眼可见光范围是什么：

   波长在400纳米~700纳米之间的电磁波



## 模型与材质

### 渲染管线与模型基础

#### 图形渲染管线

![TA100-52](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-52.png)



#### 模型的实现原理

点连成线→线围成面→组成多边形→一个模型空间下的模型形成

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614093439928.png" alt="TA100-53" style="zoom:80%;" />



#### UV

UV平铺在一个二维坐标系中，模型的每个顶点在三维空间和二维空间中都能一一对应，在二维坐标系中的顶点对应的位置就是顶点的纹理坐标，因此每个顶点都能利用纹理坐标获取到贴图所存储的信息

在建模软件中进行UV展开，UV会放在一个横轴为U，纵轴为V，范围为（0~1）的二维坐标系中



###### 一个模型包含的信息（以obj文件为例）

- V：顶点坐标数据（模型空间中单个顶点的XYZ坐标）
- VT：贴图坐标（水平方向是U，垂直方向是V，范围在0~1之间）
- VN：顶点法线→会决定面的朝向
- 顶点色：单个顶点的RGBA通道颜色信息



**obj格式与fbx格式的区别**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-53.png" alt="TA100-53" style="zoom: 80%;" />



#### 材质

在现实世界里，每个物体会对光产生不同的反应：有些物体反射光的时候不会有太多的散射(Scatter)，因而产生一个较小的高光点而有些物体则会散射很多，产生一个有着更大半径的高光点

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-54.png" alt="TA100-54" style="zoom: 50%;" />



#### 漫反射

漫反射是最容易模拟的模型。

最简单的**Lambertian**很简单粗暴的认为光线**均匀地反射出去**
$$
Diffuse = BaseColor * LightColor * dot(LightDir, NormalDir)
$$
<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-55.png" alt="TA100-55" style="zoom: 50%;" />



#### 镜面反射

镜面反射就是将入射光线根据表面法线进行反射，并且**只有在反射方向有能量，其他方向能量均为0**
$$
Specular = LightColor * pow(dot(ReflectDir, ViewDir), x)
$$
<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-56.png" alt="TA100-56" style="zoom:50%;" />



#### 折射

对于玻璃这种电介质，除了反射之外还有根据物体的折射率折射一部分光线进入物体之中反射和折射能量的多少是根据菲尼尔定律决定
$$
ReflDir = refract(ViewDir, NormalDir, ration)
$$

$$
ReflColor =  texCUBE(skybox, ReflDir)
$$

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-57.png" alt="TA100-57" style="zoom:50%;" />



#### 粗糙镜面反射

法线偏移较小。反射依然集中在一个区域。形成有磨砂质感的金属表面。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-58.png" alt="TA100-58" style="zoom:50%;" />



#### 粗糙镜面折射

毛玻璃效果

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-59.png" alt="TA100-59" style="zoom:50%;" />



#### 多层材质

涂了油漆的地板

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-60.png" alt="TA100-60" style="zoom: 45%;" />



#### SSS次表面散射

多发生在半透明的物体上，如玉石、牛奶、皮肤、蜡烛

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-61.png" alt="TA100-61 光进入皮肤后照亮了毛细血管，因此在明暗交界处更容易看到反射出的红光" style="zoom:45%;" />



#### 多层皮肤模型

我们把皮肤看成三层：油脂层（微量、很薄），表皮层，真皮层

- 正是因为有油脂层，油脂层直接把光反射出去，所以皮肤上才会有高光产生
- 没有被反射的光通过折射进入子表面层，光进入这些层之后被部分吸收（获得颜色）和散射
- 再从皮肤中入射点附近的出射点射出
- 这个过程就产生了次表面散射的效果

![TA100-62](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-62.png)



#### 改变材质表面

现实世界中不存在完美平滑的表面，因此需要对模型表面的法线进行扰动。

其中一个方法是使用法线贴图。

漫反射，高光，折射，都与法线有关，因此改变法线，就能改变其光照计算结果



### 模型数据解析

#### 模型数据在渲染中的作用

1. 顶点动画：在顶点着色器中，修改模型的顶点位置，进而达到模型运动的效果

   - 顶点动画就是在顶点着色器中对纹理的顶点进行进行操作进而产生动画效果

   - 顶点动画改变的是一个顶点的位置，需要一定数量的顶点才能得到比较好的效果

   - 一个顶点传入一个顶点着色器，顶点着色器控制顶点时，每个顶点都会执行同样的算法

2. 纹理动画：在片段着色器中，修改模型的UV信息，使得采样贴图时，发生位移而产生运动效果

3. 顶点色

   - 通过重心坐标插值计算三角形内部的颜色



#### 顶点法线与面法线

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230614095731628.png" alt="TA100-63" style="zoom: 80%;" />

差异的原理：

- （面法线）未使用平滑时，三角形三个顶点**共用**一个法线。 那么插值时，因为三个顶点的法线相同，所以插值的结果**相同**。

- （顶点法线）使用平滑后，**一个顶点一个法线**。三角形三个顶点的法线也就**不相同**。插值结果，也就会不同。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-64.png" alt="TA100-64" style="zoom: 67%;" />

在NPR渲染中。通常在顶点着色器中，将顶点往法线方向偏移。然后再片段着色器中直接输出一个颜色，达到描边的效果。**BackFacing描边**时，线条之间断开就是因为没有使用顶点法线（没有进行平滑着色）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-65.png" alt="TA100-65" style="zoom:50%;" />



## HLSL常用函数

官方文档：

https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions

一个实用的函数可视化网站：

https://graphtoy.com/



### 基本数学运算

- max(x,y)

  - 返回x和y两者中较大的那个数

- min(x,y)

  - 返回x和y两者中较小的那个数

- mul(x,y)

  - 两变量相乘，常用于矩阵运算
    - 如果x是向量，则被视为行向量；如果y是向量，则被视为列向量

- abs(x)

  - 取绝对值

- round(x)

  - 返回与x最近的整数

    ![TA100-66](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-66.png)

- sqrt(x)

  - 返回x的平方根

- rsqrt(x)

  - 返回x的平方根的倒数

- degrees(x)

  - 将弧度转换成角度

- radians(x)

  - 将角度转换成弧度

- noise(x)

  - 根据传入的浮点型向量，返回基于一个Perlin-noise算法的范围在-1~1之间的噪波值



### 幂指对函数

- pow(x,y)
  - 返回x的y次幂（x和y均可为常量或变量）
- exp(x)
  - 返回以e为底，x为指数的幂，即$e^x$
- exp2(value x)
  - 返回以2为底，x为指数的幂，即$2^x$
- ldexp(x,exp)
  - 返回x与2的exp次方的乘积，即$x*2^{exp}$
- log(x)
  - 返回指定值的以e为底的对数，即$lnx$lo
- log10(x)
  - 求以10为底的对数，即$log_{10}x$
- log2(x)
  - 求以2为底的对数，即$log_{2}x$
- frexp(x,out exp)
  - 把浮点数x分解成尾数和指数，$x=ret*2^{exp}$
  - 函数返回值为尾数，exp为指数（原始浮点数的二进制指数）
  - 如果x参数为0，则此函数的尾数和指数均返回0



### 三角函数与双曲函数

- asin(x)
  - 返回输入值的反正弦值
- acos(x)
  - 返回输入值的反余弦值
- atan(x)
  - 返回输入值的反正切值
- atan2(y,x)
  - 返回y/x的反正切值
- **sin(x)、cos(x)、tan(x)、tan(y/x)**
- sincos(x,out s,out c)
  - 返回x的正弦值和余弦值
- sinh(x)
  - 返回x的双曲正弦值，即$(e^x-e^{-x})/2$
- cosh(x)
  - 返回x的双曲余弦值，即$(e^x+e^{-x})/2$
- tanh(x)
  - 返回x的双曲正切值，即$(e^x-e^{-x}/e^x+e^{-x})$



### 数据范围类

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-67.png" alt="TA100-67" style="zoom: 80%;" />



### 类型判断类

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-68.png" alt="TA100-68"  />



### 向量与矩阵类

![TA100-69](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-69.png)



### 光线运算类

- reflect(i ,n)
  - 以i为入射向量，n为法线方向的反射光
- refract(i, n, ri)
  - 以i为入射向量，n为法线方向，ri为折射率的折射光
- lit(*n_dot_l*, *n_dot_h*, *m*)
  - 返回光照系数向量**(ambient, diffuse, specular, 1)，其中：**
    - ambient = 1
    - diffuse = n · l < 0 ? 0 : n · l
    - specular = n · h < 0 ? 0 : pow( n · l, m)
- faceforward(*n*, *i*, *ng*)
  - 在特定条件下翻转表面法线，表面面向i所指方向，返回面向i所指方向的面的法线方向（得到面向视图的曲面法向量）
  - return -n*sign(dot(i, ng))



### 纹理查找

#### 1D纹理查找（几乎不用）

- GPU在PS阶段是在屏幕空间XY坐标系中对每一个像素去对应的纹理中查找对应的纹素来确定像素的颜色
- tex1D(s, t) 普通一维纹理查找 返回纹理采样器s在标量t位置的color4
- tex1D(s,t,ddx,ddy) 使用微分查询一维纹理 t和ddxy均为vectortex1Dlod(s, t) 使用LOD查找纹理s在t.w位置的color4
- tex1Dbias(s, t) 将t.w决定的某个MIP层偏置后的一维纹理查找
- tex1Dgrad(s,t,ddx,ddy) 使用微分并指定MIP层的一维纹理查找
- tex1Dproj(s, t) 把纹理当做一张幻灯片投影到场景中，先使用投影纹理技术需要计算出投影纹理坐标t(坐标t.w除以透视值)，然后使用投影纹理坐标进行查询

#### 2D纹理查找

- tex2D(s, t) 普通二维纹理查找 返回纹理采样器s在vector t位置的颜色
- tex2D(s,t,ddx,ddy) 使用微分查询二维纹理 t和ddxy均为vector（只有小于ddxy的值才会采样）
- tex2Dlod(s, t) 使用LOD查找纹理s在t.w位置的color4tex2Dbias(s, t) 将t.w决定的某个MIP层偏置后的二维纹理查找
- tex2Dgrad(s,t,ddx,ddy) 使用微分并指定MIP层的二维纹理查找
- tex2Dproj(s, t) 把纹理当做一张幻灯片投影到场景中，先使用投影纹理技术需要计算出投影纹理坐标t(坐标t.w除以透视值)，然后使用投影纹理坐标进行查询

#### 3D纹理查找

- tex3D(s, t) 普通三维纹理查找 返回纹理采样器s在vector t位置的颜色
- tex3D(s,t,ddx,ddy) 使用微分查询三维纹理 t和ddxy均为vector
- tex3Dlod(s, t) 使用LOD查找纹理s在t.w位置的color4
- tex3Dbias(s, t) 将t.w决定的某个MIP层偏置后的三维纹理查找tex3Dgrad(s,t,ddx,ddy) 使用微分并指定MIP层的三维纹理查找
- tex3Dproj(s, t) 把纹理当做一张幻灯片投影到场景中，先使用投影纹理技术需要计算出投影纹理坐标t(坐标t.w除以透视值)，然后使用投影纹理坐标进行查询

#### 立体纹理查找

- texCUBE(s,t) 返回纹理采样器s在vector t位置的颜色
- texCUBE(s,t,ddx,ddy)使用微分查询立方体维纹理 t和ddxy均为vector
- texCUBEDload(s,t) 使用LOD查找纹理s在t.w位置的color4
- texCUBEbias(s,t) 将t.w决定的某个MIP层偏置后的立方体纹理查找
- texCUBEgrad(s,t,ddx,ddy) 使用微分并指定MIP层的立方体纹理查找
- texCUBEproj(s,t) 使用投影方式的立方体纹理查找



## 传统经验光照模型

### 光照模型

#### 是什么

illumination model，是一种模拟自然界光照的物理过程的一种计算机模型，即光线与空间中物体表面的交互模型，大致分为两类：

1. 基于物理的模型
2. 经验模型

#### 为什么

现实世界光照复杂，使用简化的光照模型对现实世界的光照情况进行模拟

###### 光照模型发展历程

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-70.png" alt="TA100-70" style="zoom:80%;" />



### 局部光照模型

#### 定义

只考虑直接光照，不考虑物体间反射的间接光

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-71.png" alt="TA100-71" style="zoom:80%;" />



#### 漫反射

光线均匀被反射到各个方向

漫反射过程中，光线发生了吸收和散射，因此改变颜色和方向

使用Lambert余弦定理计算：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-72.png" alt="TA100-72" style="zoom: 67%;" />

- **漫反射只与光源和表面法线有关**



#### 高光反射

描述了光线与物体表面发生的反射（光强不变，方向改变）

反射率根据菲涅尔效应决定

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-73.png" alt="TA100-73" style="zoom: 67%;" />

- **Gloss影响了高光反射的范围，但高光的亮度是不变的**

- **高光反射与观察方向有关**



#### 环境光

引入环境光弥补间接光照

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-74.png" alt="TA100-74" style="zoom:67%;" />



#### 自发光

物体自身发出的光



### 经典光照模型

#### Lambert光照模型

只计算漫反射，没有高光效果

#### Phong模型

Phong = Ambient + Diffuse + Specular

使用环境光来模拟间接光照

加上了高光反射（使用反射光线方向和视线方向进行计算）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-75.png" alt="TA100-75" style="zoom: 50%;" />



#### Blinn-Phong模型

在计算高光反射时，用的是半角向量和法线向量

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-76.png" alt="TA100-76" style="zoom: 80%;" />



#### Phong和Blinn-Phone模型的区别

Blinn-Phong模型使用半角向量：

1. 计算更简洁（计算半角向量比计算反射向量更简洁）

2. 半角向量与法线的角度永远不会大于90度（但是反射光线与视线方向的角度会大于90度，导致点乘计算的值钳制在0，出现明显的边缘效果）

   <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-77.png" alt="TA100-77" style="zoom:80%;" />



#### Flat模型

使用面法线，每个面都是同一个颜色



