---
title: Games101课程笔记（一）
date: 2022-10-09 00:00:00
tags:
 - 图形学
 - GAMES
categories: Notes
keywords: 'Games101'
description: Games101课程的学习笔记
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/499b82598b3b769de3a25ab517d720e.jpg
cover: 
katex: true
---

## 线性代数 Linear Algebra

### 点乘 Dot

- 计算：

$$
\vec{x} \cdot \vec{y} = |x||y|\cos\theta
(x_1,y_1) \cdot (x_2,y_2) = x_1x_2 + y_1y_2
$$

​		向量点乘向量得到常量

- 用途：

​		判断两条向量**是否同向**

### 叉乘 Cross

- 计算：
  - 方向：右手螺旋定则
  - 大小：


$$
\vec{x} \times \vec{y} = |\vec{x}||\vec{y}|\sin\theta
$$

​		向量叉乘向量得到向量，即 $\vec{a}\times\vec{a} = \vec0$

​		写成矩阵形式：

​		
$$
\begin{aligned}
\vec{a}\times\vec{b} &= 
\left( 
\begin{matrix}
x_a\\y_a\\z_a
\end{matrix}
\right)
\times
\left( 
\begin{matrix}
x_b\\y_b\\z_b
\end{matrix}
\right)
= \left(
\begin{matrix}
y_az_b - y_bz_a\cr
z_ax_b - z_bx_a\cr
x_ay_b - x_by_a\cr
\end{matrix}
\right)
\end{aligned}
$$


- 用途：


  - 判定左右：判断一条向量在另一条向量的顺时针或逆时针方向

    ![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20230603162954.png)

    判断 $\vec{b}$ 在 $\vec{a}$ 的顺时针方向还是逆时针方向：

    - 若 $\vec{z} = \vec{a} \times \vec{b}> \vec{0}$，则$\vec{b} \text{在} \vec{a}$ 的逆时针方向
    - 若 $\vec{z} = \vec{a} \times \vec{b}< \vec{0}$，则$\vec{b} \text{在} \vec{a}$ 的顺时针方向

  - 判定内外：二维上判断点在多边形内或外

    ![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20230603163029.png)
    $$
    \because
    \vec{AB}\times\vec{AP}>\vec{0},\ 
    \vec{BC}\times\vec{BP}>\vec{0},\
    \vec{CA}\times\vec{CP}>\vec{0},\
    $$
    

所以
$\vec{AP} \text{在}\vec{AB}$的逆时针方向
$\vec{BP} \text{在} \vec{BC}$的逆时针方向
$\vec{CP} \text{在} \vec{VA}$的逆时针方向
所以
P在$\triangle{ABC}$内部

### 矩阵相乘

(M $\times$ N) (N $\times$ P) = (M $\times$ P)，即 M行N列矩阵$\times$N行P列矩阵 = M行P列矩阵



## 变换 Transformation

### 二维变换

- **缩放矩阵**

  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  S_x & 0\cr
  0 & S_y\cr
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  $$

- **对称矩阵**

  沿Y轴
  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  -1 & 0\cr
  0 & 1\cr
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  $$
  沿X轴
  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  1 & 0\cr
  0 & -1\cr 
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  $$

- **切变矩阵**

  ![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20230603164027.png)
  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  1 & a\cr
  0 & 1\cr
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  $$
  


- **旋转变换**

  （默认旋转中心为原点、逆时针方向旋转）

  ![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/20230603164129.png)
  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  \cos{\theta} & -\sin{\theta}\cr
  \sin{\theta} & \cos{\theta}
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  $$
  ==**旋转变换是正交矩阵**==，证明如下：
  $$
  \because
  R_\theta = 
  \left(
  \begin{matrix}
  cos{\theta} & -sin{\theta}\cr
  sin{\theta} & cos{\theta}
  \end{matrix}
  \right)
  $$
  
  $$
  R_{-\theta} = 
  \left(
  \begin{matrix}
  cos{\theta} & sin{\theta}\cr
  -sin{\theta} & cos{\theta}
  \end{matrix}
  \right)
  \therefore
  R_\theta R_{-\theta} = E
  $$
  
  所以 $ R_\theta$ 和 $R_{-\theta}$ 互逆
  
  $$
  \therefore R_{-\theta} = R_\theta^{-1} = R_\theta^T
  $$
  
  所以$R_\theta$是正交矩阵
  
- **平移变换**（非线性变换）
  $$
  \left(
  \begin{matrix}
  x'\\y'
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  1 & 0\cr
  0 & 1\cr 
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y 
  \end{matrix}
  \right)
  +
  \left(
  \begin{matrix}
  t_x\\t_y
  \end{matrix}
  \right)
  $$



### 齐次坐标

用途：将线性变换与非线性变换统一为一个公式（矩阵x向量）

#### 齐次坐标的引入

1. 引入齐次坐标——通过增加一个维度的数来表示原本维度的数据

   一般像这样设：

   - 2D的点
     $$
     \left(
     \begin{matrix}
     x\\y\\1
     \end{matrix}
     \right)
     $$

   - 2D的向量
     $$
     \left(
     \begin{matrix}
     x\\y\\0
     \end{matrix}
     \right)
     $$

2. 引入齐次坐标之后，也满足以下计算的意义

- vector + vector = vector
- point - point = vector
- point + vector = point
- point + point = 两个点的中点point

3. 齐次坐标中
   $$
   \left(
   \begin{matrix}
   x\\y\\w
   \end{matrix}
   \right)
   ==
   \left(
   \begin{matrix}
   x/w\\y/w\\1
   \end{matrix}
   \right)
   $$
   

   所以在齐次坐标中，point + point表示的是两个点的中点



#### 齐次坐标的运用

**仿射变换 = 线性变换 + 平移变换**，即
$$
\left(
\begin{matrix}
x'\\y'
\end{matrix}
\right)
=
\left(
\begin{matrix}
a & b\cr
c & d\cr 
\end{matrix}
\right)
\left(
\begin{matrix}
x\\y 
\end{matrix}
\right)
+
\left(
\begin{matrix}
t_x\\t_y
\end{matrix}
\right)
$$
(由此可知：**仿射变换先进行线性变换再进行平移**)

**如果使用齐次坐标：**
$$
\left(
\begin{matrix}
x'\\y'\\1
\end{matrix}
\right)
=
\left(
\begin{matrix}
a & b & t_x\cr
c & d & t_y\cr 
0 & 0 & 1
\end{matrix}
\right)
\left(
\begin{matrix}
x\\y \\1
\end{matrix}
\right)
$$
二维仿射变换矩阵的最后一行永远是(0,0,1)，但在其他情况下可能有所不同

用齐次坐标的形式表示常用仿射变换：

- Scale
  $$
  S(s_x,s_y) = 
  \left(
  \begin{matrix}
  s_x & 0 & 0\cr
  0 & s_y & 0\cr
  0 & 0 & 1
  \end{matrix}
  \right)
  $$
  
- Rotation
  $$
  R(\alpha) = 
  \left(
  \begin{matrix}
  cos{\alpha} & -sin{\alpha} & 0\cr
  sin{\alpha} & cos{\alpha} & 0\cr
  0 & 0 & 1
  \end{matrix}
  \right)
  $$
  
- Transition
  $$
  \left(
  \begin{matrix}
  1 & 0 & t_x\cr
  0 & 1 & t_y\cr
  0 & 0 & 1
  \end{matrix}
  \right)
  $$



### 组合变换

逆变换

乘以逆矩阵$M^-$



组合变换

**矩阵的应用是从右到左的：**

变换的顺序不同则得到的结果不同，进行一次变换就在当前矩阵**前方**乘上一个矩阵，且注意矩阵乘法不能交换顺序

即 $R_\theta T(1,0)\ != \ T(1,0)R_\theta$

**(性能)**但是矩阵乘法有结合律，通过预先相乘获得一个表示一系列变换的$3 \times 3\text{的矩阵}\rightarrow$可以借此来优化性能
$$
A_n(...A_2(A_1(x))) = A_n...A_2A_1
\left(
\begin{matrix}
x\\y\\1
\end{matrix}
\right)
$$

分解组合变换

问题的提 “如何沿着一个给定的点旋转？”

将中心点移动到远点$\rightarrow$进行旋转$\rightarrow$移动回原位置
$$
T(c)T(\alpha)T(-c)
$$
(变换是从右向左进行)



### 三维变换

用二维进行类比，可以得出：

- 3D point
  $$
  \left(
  \begin{matrix}
  x\\y\\z\\1
  \end{matrix}
  \right)
  $$

- 3D vector
  $$
  \left(
  \begin{matrix}
  x\\y\\z\\0
  \end{matrix}
  \right)
  $$

- 齐次坐标中
  $$
  \left(
  \begin{matrix}
  x\\y\\z\\w
  \end{matrix}
  \right)
  ==
  \left(
  \begin{matrix}
  x/w\\y/w\\z/w\\1
  \end{matrix}
  \right)
  $$

- 仿射变换：
  $$
  \left(
  \begin{matrix}
  x'\\y'\\z'\\1
  \end{matrix}
  \right)
  =
  \left(
  \begin{matrix}
  a & b & c & t_x\cr
  d & e & f & t_y\cr
  g & h & i & t_z\cr
  0 & 0 & 0 & 1
  \end{matrix}
  \right)
  \left(
  \begin{matrix}
  x\\y\\z\\1
  \end{matrix}
  \right)
  $$

  - Scale
    $$
    S(s_x,s_y,s_z) = 
    \left(
    \begin{matrix}
    s_x & 0 & 0 & 0\cr
    0 & s_y & 0 & 0\cr
    0 & 0 & s_z & 0\cr
    0 & 0 & 0 & 1
    \end{matrix}
    \right)
    $$

  - Translation
    $$
    T(t_x,t_y) = 
    \left(
    \begin{matrix}
    1 & 0 & t_x\cr
    0 & 1 & t_y\cr
    0 & 0 & 1\cr
    \end{matrix}
    \right)
    $$

  - Rotation

    - 基于轴旋转

    $$
    R_x(\alpha) = 
    \left(
    \begin{matrix}
    1 & 0 & 0 & 0\cr
    0 & cos{\alpha} & -sin{\alpha} & 0 \cr
    0 & sin{\alpha} & cos{\alpha} & 0\cr
    0 & 0 & 0 & 1
    \end{matrix}
    \right)
    $$

    $$
    R_y(\alpha) = 
    \left(
    \begin{matrix}
    cos{\alpha} & 0 & sin{\alpha} & 0\cr
    0 & 1 & 0 & 0 \cr
    -sin{\alpha} & 0 & cos{\alpha} & 0\cr
    0 & 0 & 0 & 1
    \end{matrix}
    \right)
    $$

    $$
    R_z(\alpha) = 
    \left(
    \begin{matrix}
    
    cos{\alpha} & -sin{\alpha} & 0 & 0\cr
    sin{\alpha} & cos{\alpha} & 0 & 0\cr
    0 & 0 & 1 & 0\cr
    0 & 0 & 0 & 1
    \end{matrix}
    \right)
    $$

    

    - 任意旋转
      $$
      R_{xyz}(\alpha,\beta,\gamma) = R_x(\alpha)R_y(\beta)R_z(\gamma)
      $$
      **欧拉角：**

      ​	yaw：左右摇头

      ​	pitch：上下点头

      ​	roll：左右偏头

      沿着轴$\vec{n}$旋转$\alpha$角度——**Rodrigoues Rotation Formula**：
      $$
      R(\vec{n},\alpha) = cos{\alpha}I + (1-cos{\alpha})\vec{n}\vec{n}^T+sin({\alpha})
      \left(
      \begin{matrix}
      0 & -n_x & n_y\cr
      n_z & 0 & -n_x\cr
      -n_y & n_x & 0
      \end{matrix}
      \right)
      $$
      ($\vec{n}$为默认轴，即$\vec{n}$过圆心)



## MVP

### Model Transformation 模型变换

通过World Matrix将顶点数据从Object Space转换到World Space



### View / Camera Transformation 视图/相机变换

#### 首先定义一个相机

1. Position:  $\vec{e}$
2. Look at/gaze direction: $\vec{g}$
3. Up direction : $\vec{t}$（定义Roll —— 即是横屏还是竖屏）

#### 如何进行视图变换

1. 相机标准位置

   - 原点
   - 向上方向为y轴正方向
   - 看向-z方向

2. 通过$M_{view}$将Camera以及其他物体转换到标准位置

   - Translation Matrix：$\vec{e}$ to orgin
   - Rotate：$\vec{g}$ to $-z$
   - Rotate：$\vec{t}$ to $y$
   - Rotate： $\vec{g}\times\vec{t}$ to $x$

3. $M_{view}$ in Math

   $M_{view} = R_{view}T_{view}$（先平移再旋转）
   
   注：旋转矩阵是正交矩阵，用逆矩阵更好得出矩阵
   逆变换inverse rotation是: X to (g x t), Y to t, Z to -g
   $$
   T_{view} = 
   \left(
   \begin{matrix}
   1 & 0 & 0 & -x_{\vec{e}}\cr
   0 & 1 & 0 & -y_{\vec{e}}\cr
   0 & 0 & 1 & -z_{\vec{e}}\cr
   0 & 0 & 0 & 1
   \end{matrix}
   \right)
   $$
	
	$$
	R_{view} = 
	\left(
	\begin{matrix}
	x_{\vec{g}\times\vec{t}} & y_{\vec{g}\times\vec{t}} & 			z_{\vec{g}\times\vec{t}} & 0\cr
	x_{t} & y_{t} & z_{t} & 0\cr
	x_{-g} & y_{-g} & z_{-g} & 0\cr
	0 & 0 & 0 & 1
	\end{matrix}
   \right)
   $$
   
   $$
   R_{view} = {(R_{view}^{-1})}^T =   
   \left(
   \begin{matrix}
   x_{\vec{g}\times\vec{t}} & x_{\vec{t}} & x_{\vec{g}} & 0\cr
   y_{\vec{g}\times\vec{t}} & y_{\vec{t}} & y_{\vec{g}} & 0\cr
   z_{\vec{g}\times\vec{t}} & z_{\vec{t}} & z_{\vec{g}} & 0\cr
   0 & 0 & 0 & 1
   \end{matrix}
   \right)
   $$
   
   
   

### Projection Transformation 投影变换

—— 这一步实现3D$\rightarrow$2D的效果

投影变换的两种类型

- Orthographic Projection（假设相机无限远）
- Perspective Projection（相机作为一个点）



#### Orthographic Projection 正交投影

1. 平移到中心

2. 缩放为canonical cube ($[-1,1]^3$)

   Transform Matrix：
   $$
   M_{ortho} = 
   \left(
   \begin{matrix}
   \frac{2}{r-l} & 0 & 0 & 0\cr
   0 & \frac{2}{t-b} & 0 & 0\cr
   0 & 0 & \frac{2}{n-f} & 0\cr
   0 & 0 & 0 & 1
   \end{matrix}
   \right)
   \left(
   \begin{matrix}
   1 & 0 & 0 & -\frac{r+l}{2}\cr
   0 & 1 & 0 & -\frac{t+b}{2}\cr
   0 & 0 & 1 & -\frac{n+f}{2}\cr
   0 & 0 & 0 & 1
   \end{matrix}
   \right)
   $$
   ![image-20221016122040590](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221016122040590.png)

（因为我们使用的是**右手系**，所以Z值越小深度值越大，照相机看向$-z$）



#### Perspective Projection 透视投影

######## 如何定义一个Frustum

定义一个垂直可视角度field-of-view(fovY)、和一个宽高比aspect radio即可

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017144903795.png" alt="image-20221017144903795" style="zoom: 50%;" />

######## **Perspective Projection**

近大远小，平行线不再平行

![image-20221016155734310](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221016155734310.png)

如何进行投影变换

1. “Squish” the frustum into a cuboid（将视锥体的内容压缩到Cuboid中）

2. do orthographic projection（进行正交投影）

推导过程：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221016202533188.png" alt="image-20221016202533188" style="zoom: 25%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230605111452269.png" alt="image-20230605111452269" style="zoom: 25%;" />



## 光栅化 Rasterization

### Canonical Cube to Screen

—— **Rasterize == drawing onto the screen**

#### 屏幕和像素

- Screen is an arrays of **pixels**. 

- **Resolution** is the size of array.

  

**Pixel**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017204039400.png" alt="image-20221017204039400" style="zoom:50%;" />



**Screen Space**

(不同地方对Screen Space的定义会有不同)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017145901163.png" alt="image-20221017145901163" style="zoom:50%;" />

- Pixel以$(x,y)$的形式索引，$x$和$y$都是整数
- $(x,y)$范围从$(0,0)$到$(width-1,height-1 )$
- Pixel中心是$(x+0.5,y+0.5)$
- 屏幕大小为$(0,0)$到$(width,height)$

#### 视口变换

- 忽略z轴

- 变换$xy$平面：$[-1,1]^2\rightarrow [0,width]\times[0,height]$

- Viewport transform matrix:
  $$
  M_{viewport}=
  \left(
  \begin{matrix}
  \frac{width}{2} & 0 & 0 & \frac{width}{2}\cr
  0 & \frac{height}{2} & 0 & \frac{height}{2}\cr
  0 & 0 & 1 & 0\cr
  0 & 0 & 0 & 1
  \end{matrix}
  \right)
  $$



###  Drawing to Raster Display

#### Frame Buffer 帧缓冲区

 为成像设备进行信息显示而开辟的，用于存储显示信息的内存

成像设备：

- 示波器

  原理：阴极射线管

- **LCD**(Liquid Crystal Display) Pixel（液晶显示）

  原理：控制光栅

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017155732323.png" alt="image-20221017155732323" style="zoom:50%;" />

- **LED**(Light emitting diode array) Array Display

  发光二极管

- Electrophoretic (Electronic Ink) Display

  水墨屏

#### Sample 采样

判断像素中心点是否在三角形内部

- 定义一个inside函数

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017202645582.png" alt="image-20221017202645582" style="zoom: 67%;" />

- inside函数的实现：三个叉积运算

- 边缘问题：三角形边上的点的归属关系（不同地方定义不同，图形API内会有严格定义）

- Axis Alined Bounding Box (AABB)

  判断Bounding Box区域内的像素点，不需要整个屏幕的所有像素点都做判断

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017203512580.png" alt="image-20221017203512580" style="zoom: 67%;" />

- 进一步缩小bounding box的方法，但是这样处理不一定能让程序运行更快

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221017203936076.png" alt="image-20221017203936076" style="zoom: 50%;" />

#### Anti-Aliasing 反走样

**Sampling Theorem 采样理论**

- Rasterization = Sample 2D Position（在屏幕上采样像素位置）
- Photograph = Sample Image Sensor Plane（在光感受器上采样自然光线信息）
- Vedio = Sample Time (在时间上采样，每一帧都是一个特定的时间点，而非连续的)



 **Sampling Artifacts 采样走样**

- Jaggies (Aliasing) — sampling in space
- Moire — undersampling images
- Wagon Wheel effect (False Motion) — sampling in time

Sampling Artifacts的原因：信号变化太快导致采样跟不上他的速度（速度不匹配）



**Anti-Aliasing 反走样**

先对原始信号做一个滤波，然后再进行采样

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018090056916.png" alt="image-20221018090056916" style="zoom:50%;" />



**Signal Processing Theorem 信号处理理论**

**傅里叶级数展开**：把一个函数描述成很多正弦余弦函数的和（一个函数可以分解为不同的频率）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018092939328.png" alt="image-20221018092939328" style="zoom:80%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018093733480.png" alt="image-20221018093733480" style="zoom:50%;" />

**傅里叶变换**：将一个信号分解为不同的频率

给定一个函数，我们都可以对其进行一定操作让他变为另一个函数，并且操作可逆

（**时域与频域**之间的转换）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018093802743.png" alt="image-20221018093802743" style="zoom:50%;" />

**通过频率的概念理解Aliasing**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018094327200.png" alt="image-20221018094327200" style="zoom:50%;" />

- 高频信号没有被充分采样，导致采样后错误地表现出一个低频信号
- 在给定采样速率下，两个不同频率的信号经过采样得出的结果难以区分，叫做**aliasing**

**因而，高频率需要更快的采样速度**，以满足对原始信号的更充分表达

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018093908480.png" alt="image-20221018093908480" style="zoom:50%;" />



**Filtering 滤波**

用于去掉一些特定频率的内容

**频谱：**中心低频，越往外频率越高（对应到图像上，颜色信息差距越大的地方频率越高）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230605113700273.png" alt="image-20230605113700273" style="zoom: 67%;" />

- 过滤低频信号：Edge获取内容的边界（高通滤波器）

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221018100448310.png" alt="image-20221018100448310" style="zoom: 45%;" />

- 过滤高频信号：Blur模糊（低通滤波器）

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221018100716159.png" alt="image-20221018100716159" style="zoom: 45%;" />

- 过滤低频和高频（获取中间频率内容）

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221018100911332.png" alt="image-20221018100911332" style="zoom: 45%;" />

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221018100922466.png" alt="image-20221018100922466" style="zoom: 45%;" />



**Filetering = Convolution(Average)**

**Convolution(卷积)：**

图形学简化后的概念：原始信号与周围信号进行加权平均得到结果

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018101236140.png" alt="image-20221018101236140" style="zoom: 33%;" />

**Convolution Theorem(卷积定理)：**

- 时域卷积 = 频域乘积

- 时域乘积 = 频域卷积 

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018103251589.png" alt="image-20221018103251589" style="zoom:50%;" />

**Box Filter(卷积核)：**

- 乘$\frac{1}{9}$是为了做归一化，防止亮度过高

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018103529470.png" alt="image-20221018103529470" style="zoom:50%;" />

- 卷积核在时域上越大，则表现在频域上就越小

  即：卷积的范围越大，信号处理后高频信息更少，所以乘上的卷积核在频域上越小

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230605114110085.png" alt="image-20230605114110085" style="zoom:50%;" />



**通过频率的概念来理解Sampling**

采样 = 模拟原始信号的频谱

给你原始信号A，乘上冲击函数，得到采样结果

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221018104656047.png" alt="image-20221018104656047" style="zoom: 50%;" />

走样 = 混叠频率内容

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019104206921.png" alt="image-20221019104206921" style="zoom:50%;" />



**如何减少走样错误**

- Option 1：Increase sampling rate

  - 本质是增加傅里叶变换级数
  - 高分辨率的呈现、显示设备和framebuffers
  - 但是消耗高，并且可能受限于物理限制（低分辨率屏幕）

- Option 2：Antialiasing

  - 在repeating前先让傅里叶展开的内容更接近

  - 即在采样前通过低通滤波先过滤掉高频信息，然后再采样

    Antialiasing = **Limiting, then repeating**

    <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221019110128918.png" alt="image-20221019110128918" style="zoom: 50%;" />

    <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221019110152125.png" alt="image-20221019110152125" style="zoom: 50%;" />

  - 实际运用的Pre-Filter

    一个1像素大小的box filter（低通滤波，模糊效果）

    <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221019110554940.png" alt="image-20221019110554940" style="zoom: 50%;" />

    **Solution**：

    - 通过一个一像素大小的box-blur去卷积$f(x,y)$

      (convolving = filtering = averaging)

    - 然后再对每个像素的中心进行采样

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019111052408.png" alt="image-20221019111052408" style="zoom:50%;" />



#### 常见抗锯齿方法

**MSAA**

Multi-Sample Anti-alias, 对Anti-alias的近似，而不能严格意义上完全解决Anti-alias的问题

- Step 1

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019114336950.png" alt="image-20221019114336950" style="zoom:80%;" />

  在一个像素内部增加采样点，得到更加准确的三角形在像素内的近似覆盖率

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019114541770.png" alt="image-20221019114541770" style="zoom: 50%;" />

- Step 2

  根据三角形的覆盖率求每个像素的平均值

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019114838200.png" alt="image-20221019114838200" style="zoom: 50%;" />

  

MSAA是通过**优化Blur(处理模糊信号)**（对像素采样操刀）那个过程来实现反走样的，而不是通过提高采样率，不是增加分辨率



**FXAA**

Fast Approximate AA，快速近似抗锯齿

是一种**图像的后期处理**，原理是得到一副有锯齿的图，通过图像匹配的方法找到边界，并且把这些边界换成没有锯齿的边界，而且非常非常快



**TAA**

Temporal AA，**基于后处理实现**

简单高效，复用上一帧的信息来做AA，对于静态场景来说，相当于把MASS的样本分布在了时间上，并且对于当前帧没有做任何操作



**Super Resolution / Super Sampling 超采样**

- 把低分辨率图拉大成高分辨率的图
- 本质上也是在解决采样不足的问题
- **DLSS(Deep Learning Super Sampling)**



#### Visibility / Occlusion 可见性与剔除

**Painter’s Algorithm 画家算法**

从最远的内容开始画，画到近处的内容，近处的内容覆盖远处的内容（画油画）

时间复杂度：$O_{(nlogn)}$ for $n$ triangles（在深度上进行排序）

会有无法解决的深度排序问题：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019152725046.png" alt="image-20221019152725046" style="zoom: 67%;" />



**ZBuffer 深度缓冲区**

Idea：

- 储存当前每个像素的最小深度值，**每个像素内记录最浅深度**
- 需要一个额外的buffer来存储深度信息
  - frame buffer储存颜色信息
  - depth buffer(z—buffer)存储深度信息

我们假定z是一个正值，代表物体到摄像机的深度距离，z越小越近，z越大越远

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019153402311.png" alt="image-20221019153402311" style="zoom:50%;" />



算法实现：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019182525558.png" alt="image-20221019182525558" style="zoom: 50%;" />

图解：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221019182547450.png" alt="image-20221019182547450" style="zoom:50%;" />

深度算法的时间复杂度：

$O_{(n)}$ for $n$ triangles（不进行排序，直接替换最小值）



## 着色 Shading

### 着色的定义

将**材质**应用到一个物体上的过程



### 着色模型

#### **Perceptual Observation**

着色是一种感性的认知

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230605115049589.png" alt="image-20230605115049589" style="zoom:50%;" />

**着色是局部的**

shading有局部性，不考虑其他物体的存在，只考虑这个点的着色，不会考虑光线是否被遮挡

shading $!=$ shadow，shading得到的是**明暗变化**，但是不生成阴影



#### Diffuse Reflection(Lambertian Term)

- 接收角度

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608211854012.png" alt="image-20230608211854012" style="zoom:50%;" />

- 衰减角度

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608211918465.png" alt="image-20230608211918465" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608211956088.png" alt="image-20230608211956088" style="zoom: 67%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608212505869.png" alt="image-20230608212505869" style="zoom:50%;" />



#### Specular Term(Blinn-Phong)

Intensity depends on view direction

- Bright near mirror reflection direction

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608212706491.png" alt="image-20230608212706491" style="zoom:50%;" />

  V close to mirror reflection direction <=> **half vector** near normal

- 通过单位向量的点乘来判断法线的“near”程度

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608212751472.png" alt="image-20230608212751472" style="zoom:50%;" />

  1. 因为Blin-Phong模型是经验模型，所以没有考虑能量的问题（漫反射、点乘）

  2. 使用反射向量与观测向量点乘的方法叫做Phong模型，但是反射向量较难计算，**Blinn-Phong模型更占优**

  3. 指数p的由来：向量点乘确实能表现两个向量的相邻程度，但是他的**容忍度**太高了，通过指数p来降低容忍度。实际效果表现为p控制高光大小，Blinn-Phong模型中p一般为100-200。

     <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213045889.png" alt="image-20230608213045889" style="zoom:50%;" />

     <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213055454.png" alt="image-20230608213055454" style="zoom:50%;" />



####  Ambient Term

渲染环境光不依赖于任何东西

- 添加一个常数颜色值去处理被忽略的照明并填充全黑的阴影
- Blinn-Phong模型中的环境光是一个近似值

即：不让阴影全黑

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213312593.png" alt="image-20230608213312593" style="zoom:50%;" />



#### This is Blinn-Phong Reflection Model

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213337319.png" alt="image-20230608213337319" style="zoom:50%;" />

Ambient + Lambert + Blin-Phong

注意用单位向量点乘表示夹角大小时，Specular才需加指数p(降低容忍度，调整高光效果)，diffuse一般不需要



### 着色频率 Shading Frequencies

所谓着色频率就是把着色应用到哪类图元上

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213720317.png" alt="image-20230608213720317" style="zoom:50%;" />

- **Flat Shading(Shade each triangle)**

  - 每个三角形是一个平面，一个三角形有一条法线，一个平面上（三角形内部）的颜色相同

  - 不适用于平滑表面

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608213934799.png" alt="image-20230608213934799" style="zoom: 80%;" />

- **Gouraud Shading(Shade each vertex)**

  - 求顶点的法线，每个顶点做一次着色，三角形内部的颜色通过插值计算出来

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608214022683.png" alt="image-20230608214022683" style="zoom:80%;" />

- **Phong Shading(Shade each pixel)**

  - 三角形内部每一个像素通过插值计算出法线

  - 利用法线计算每一个像素的颜色

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608214130071.png" alt="image-20230608214130071" style="zoom: 80%;" />



- **使用何种着色频率取决于模型三角形面数：**

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608214200761.png" alt="image-20230608214200761" style="zoom:50%;" />

- **定义逐顶点的法线：**

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022102156325.png" alt="image-20221022102156325" style="zoom: 50%;" />

  进阶：需要根据周围三角形的大小进行**加权平均**

- **定义逐顶点的法线**

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608214706559.png" alt="image-20230608214706559" style="zoom:50%;" />

利用三角形重心坐标插值计算



## 实时渲染管线 Real-time Rendering Pipline

### Pipeline

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608214837419.png" alt="image-20230608214837419" style="zoom:50%;" />

- Vertex Processing

  Model, View, Projection transforms

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221022103931571.png" alt="image-20221022103931571" style="zoom: 25%;" />

- Triangle Processing

  根据顶点信息形成三角形

- Rasteriazition

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221022103952951.png" alt="image-20221022103952951" style="zoom:25%;" />

- Fragment Processing

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221022104051402.png" alt="image-20221022104051402" style="zoom:25%;" />

- shading

  考虑不同的着色频率

  - 使用Gouraud shading时shading可以在vertex processing

    <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221022104245776.png" alt="image-20221022104245776" style="zoom:25%;" />

  - 使用Phong shading则需要等到获取像素信息后再shading



### Shader Programs

- shader是对所有顶点/像素通用的，不需要有for循环

- 分为vertex shader和fragment shader

  <img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221022105136528.png" alt="image-20221022105136528" style="zoom:50%;" />

  

### GPU

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022153032214.png" alt="image-20221022153032214" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022153207163.png" alt="image-20221022153207163" style="zoom:50%;" />



## 纹理映射 Texture Mapping

希望在物体不同位置定义不同属性——**定义任何一个点的基本属性**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022151825469.png" alt="image-20221022151825469" style="zoom: 50%;" />



**Surface are 2D**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022152031619.png" alt="image-20221022152031619" style="zoom:50%;" />



**Texture Applied to Surface**

纹理上的三角形与模型上的三角形一一映射（三角形的每个顶点对应纹理上的一个坐标）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022152159347.png" alt="image-20221022152159347" style="zoom:50%;" />

uv坐标：通常认为u、v均为0-1

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221022152445718.png" alt="image-20221022152445718" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608215134172.png" alt="image-20230608215134172" style="zoom:50%;" />



## 三角形重心坐标 Barycentric Coordinate

**目标：在三角新内部做任意属性的插值**

### Barycentirc Coordinates

A coordinate system for triangles ($\alpha ,\beta , \gamma$)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024102515064.png" alt="image-20221024102515064" style="zoom:50%;" />

A点的重心坐标

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024102557497.png" alt="image-20221024102557497" style="zoom:50%;" />

重心坐标的几何表示——面积之比

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024102756323.png" alt="image-20221024102756323" style="zoom:50%;" />

三角形的重心

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024102944584.png" alt="image-20221024102944584" style="zoom:50%;" />

任意一个点的重心坐标的计算

- 法一：算面积
- 法二：公式

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024102904788.png" alt="image-20221024102904788" style="zoom:50%;" />



### Using Barycentric Coordinates

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024103309271.png" alt="image-20221024103309271" style="zoom:50%;" />



**投影后无法保证重心坐标不变**

- 如果要插值三维空间的属性就需要在三维空间中做插值
  - 举例：深度插值在线性空间中进行



## 纹理应用 Applying Texture

### Simple Texture Mapping

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608215502239.png" alt="image-20230608215502239" style="zoom:50%;" />



###  Texture Magnification

#### Easy Case

放大后**纹理分辨率不足**的问题

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024104705021.png" alt="image-20221024104705021" style="zoom:50%;" />

**Nearest**：取最近点

**Bilinear**：双线性（取周围4个）

**Bicubic**：双向三阶（取周围16个）



**Bilinear Interpolation**

**双线性差值**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024105004314.png" alt="image-20221024105004314" style="zoom:33%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024105019778.png" alt="image-20221024105019778" style="zoom:33%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024105121151.png" alt="image-20221024105121151" style="zoom:33%;" />



#### Hard Case

**分辨率过大的问题**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024110027990.png" alt="image-20221024110027990" style="zoom:50%;" />

> **Jaggies锯齿是因为相邻频率重叠，Moire摩尔纹是因为一个采样周期内储存了原信息的多个频率周期**

Screen Pixel “Footprint” in Texture

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024110608709.png" alt="image-20221024110608709" style="zoom: 50%;" />

注释：蓝色为像素覆盖纹理的区域，灰色的框表示采样率大小

近处像素覆盖的纹理区域与采样率相适配不会出问题，但是远处像素用一个蓝色的点就取平均值表示了整个平行四边形的框，显然会有问题



使用MSAA：能得到好的结果，但是开销会很大

- 问题原因：
  - 在图形高度缩小后，一个pixel footprint里有多个texel
  - 一个像素内的信号频率过高，需要更高的采样频率

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024111409985.png" alt="image-20221024111409985" style="zoom:50%;" />

- 另一种解决思路

  采样会引起走样，如果不采样的话，只需要快速获得原采样区域的平局值（范围查询）

  

  不同像素有不同的纹理覆盖大小，那么不同像素的范围查询就应该能够查询到任意的范围大小——引入**MipMap**



### MipMap

实现（**快、近似、正方形的**）范围查询



**用一张图生成一系列的图**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024112606318.png" alt="image-20221024112606318" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024112815739.png" alt="image-20221024112815739" style="zoom:50%;" />

使用Mipmap只比原来多$\frac{1}{3}$的存储量（级数求和）



**计算不同等级的Mipmap：**

使用屏幕空间内相邻采样点对应纹素空间的坐标来估算纹理的占用空间

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024132819099.png" alt="image-20221024132819099" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024132859622.png" alt="image-20221024132859622" style="zoom:50%;" />

用相邻纹理空间坐标的最大值代表采样点覆盖纹素的正方形大小

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024132957409.png" alt="image-20221024132957409" style="zoom:50%;" />

用该正方形内的平均颜色来代表该mipmap层级内采样点的占用空间



四舍五入取mipmap层级的形象化：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024133533227.png" alt="image-20221024133533227" style="zoom:50%;" />



**Trilinear Interpolation**

Mipmap层级内部做插值（屏幕空间和纹理空间都有），Mipmap层级之间再做插值（可以理解为两次查询和一次插值）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024133700820.png" alt="image-20221024133700820" style="zoom:50%;" />

采样一张贴图的七次插值运算=Screen Space x&y(2) + Mipmap Level D x&y(2) + Mipmap Level D+1 x&y(2) + continuous D value(1)

这样处理后的mipmap层级形象化：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024134028002.png" alt="image-20221024134028002" style="zoom:50%;" />



**Mipmap的局限性**

对远处的表现过度模糊

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024135514481.png" alt="image-20221024135514481" style="zoom:50%;" />

产生问题的原因：对远处覆盖区域的采样不一定是正方形，用一个正方形的框去框住就难免会有采样精度的问题

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024140203104.png" alt="image-20221024140203104" style="zoom:50%;" />

使用**各向异性过滤(Anisotropic Filtering)**则可以部分解决Mipmap采样产生的问题（Ripmap）

各向异性过滤总共的开销是原来的四倍，mipmap只是增加了三分之一（只是对显存的开销，基本和算力无关）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024135711567.png" alt="image-20221024135711567" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024140400555.png" alt="image-20221024140400555" style="zoom:50%;" />

Mipmap正常采样图只在对角线上，其余的图对应回正常图都是矩形，所以解决了远处覆盖区域是矩形的问题（部分解决的由来），比正方形的mipmap查询要好，但是无法解决非矩形的情况

**EWA filtering**

不规则的形状可以拆成很多的圆形去覆盖这个形状，进行**多次查询**（代价大）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221024140637117.png" alt="image-20221024140637117" style="zoom:50%;" />



### Other Applications

可以把纹理理解为一块数据，可以做点查询和范围查询

texture = memory + range query（filtering）

- General method to bring data to fragment calculation

纹理的许多其他运用：

- Evironment lighting
- Store microgeometry
- Procedural textures
- Solid modeling
- Volume rendering

- …



#### Environment Map

假设只记录光源的方向信息——认为光无限远，不记录光的深度信息

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093146966.png" alt="image-20221026093146966" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093323619.png" alt="image-20221026093323619" style="zoom: 50%;" />

把光照信息存在球体上：**Spherical Environment Map**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093400107.png" alt="image-20221026093400107" style="zoom:50%;" />

问题：球体贴图展开后在靠近极点的地方会被压缩扭曲

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093459648.png" alt="image-20221026093459648" style="zoom:50%;" />

解决方法：**Cube Map**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093556148.png" alt="image-20221026093556148" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093609953.png" alt="image-20221026093609953" style="zoom:50%;" />



#### Bump Mapping

凹凸贴图：可以在不让模型复杂的基础上让物体表面的属性发生变化，人为做假的法线获得假的着色结果，让人有凹凸的感觉，但实际上模型没有凹凸

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026093804850.png" alt="image-20221026093804850" style="zoom:50%;" />



<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026094255429.png" alt="image-20221026094255429" style="zoom:50%;" />

计算方法：

（由二维类推）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026095028624.png" alt="image-20221026095028624" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026095042595.png" alt="image-20221026095042595" style="zoom:50%;" />



#### Displacement Mapping

相较于Bump Mapping，Dp贴图实际上改变了三角形顶点的位置

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026095247327.png" alt="image-20221026095247327" style="zoom:50%;" />

要求：

- 模型足够细致，三角形顶点数量足够多
  - 为此DirectX提供**动态曲面细分**，即在需要的时候动态增加三角形数量（顶点数量），以满足对顶点数量的需求



#### 3D Procedural Noise

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026095853368.png" alt="image-20221026095853368" style="zoom:50%;" />



#### Precomputed Shading

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026095931399.png" alt="image-20221026095931399" style="zoom:50%;" />



#### 3D Textures and Volume Rendering

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230608220017718.png" alt="image-20230608220017718" style="zoom:50%;" />



## 几何 Geometry

### 几何表示方法 Geometry Representation

Overview

#### Implicit

告诉你一些点满足的关系，但不告诉你这些点实际在哪儿

例如：Sphere：all points in 3D, where $x^2+y^2+Z^2 = 1$

更通用：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026101503348.png" alt="image-20221026101503348" style="zoom:50%;" />

- Sampling Can Be Hard：隐式几何难以表示

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026101643880.png" alt="image-20221026101643880" style="zoom:50%;" />

- Inside/Outside Test Easy：隐式几何便于判断点是否在几何内 

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026101749721.png" alt="image-20221026101749721" style="zoom:50%;" />



#### Explicit

所有点都被**直接给出**，或者通过**参数映射**的方法表示

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026101938890.png" alt="image-20221026101938890" style="zoom:50%;" />

- Sampling Can Be Easy：显式几何容易表示

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026102155632.png" alt="image-20221026102155632" style="zoom:50%;" />

- Inside/Outside Test Hard：显式几何难以判断点是否在几何内

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026102328575.png" alt="image-20221026102328575" style="zoom:50%;" />



==**Best Representation Depends on the Task**==



### 隐式表示 Implicit 

- Algebraic Surfaces

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026102647573.png" alt="image-20221026102647573" style="zoom:50%;" />

- Constructive Solid Geometry(CSG)

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026102904939.png" alt="image-20221026102904939" style="zoom:50%;" />

- Distance Function

  空间中的任何一个点到你想要表述的几何形体表面上任何一个点的最小距离

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026104616018.png" alt="image-20221026104616018" style="zoom:50%;" />

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026110716287.png" alt="image-20221026110716287" style="zoom:50%;" />

  在blend distance function后再恢复成原本的表面

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026104807031.png" alt="image-20221026104807031" style="zoom:50%;" />

- Level Set Methods（水平集）

  思想和距离函数相同，只不过是把距离函数表示成了带数据的格子

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026111833579.png" alt="image-20221026111833579" style="zoom:50%;" />

- Fractals（分形）

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221026112159965.png" alt="image-20221026112159965" style="zoom:50%;" />



**Implicit Representation - Pros&Cons**

Pros：

- 阐述简洁（比如通过一个函数）
- 某些特定的查询很简单（在内在外的查询、到表面的距离查询）
- 易于做光线求交
- 表示很准确，没有sampling error
- 在拓扑结构上方便做变化

Cons：

- 难以表现复杂形状



### 显式表示 Explicit

- Point Cloud（点云）

  - 最简单的表示方式：一系列的点形成

  - 可以容易地表示任何几何

  - 适用于大型数据集

  - 常转变为多边形网格

  - 不易于在采样不足的区域绘制

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027082240881.png" alt="image-20221027082240881" style="zoom:50%;" />

- Polygon Mesh（几何网格体——**用的最多**）

  - 储存顶点和多边形

  - 更易于运算、模拟和自适应采样

  - 拥有更复杂的数据结构

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027082708419.png" alt="image-20221027082708419" style="zoom:50%;" />

  - The Wavefront Object File(.obj) Format

    - 一个指明vertices, normals, texture coordinates and their connectivities的文本文件

      <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027083131340.png" alt="image-20221027083131340" style="zoom:50%;" />



### Curve

#### Bezier Curve

**定义：**

用一系列的控制点定义某条曲线，该切线满足一些性质

- 曲线始于$p_0$ 终于 $p_3$
- 曲线起始点切线为$\vec{p_0p_1}$ 终点切线为 $\vec{p_2p_3}$

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027084106100.png" alt="image-20221027084106100" style="zoom: 33%;" />



**de Casteljau Algorithm：**

给定任意的控制点绘制出贝塞尔曲线

认为$b_0 \text{到} b_2$(起点到终点)映射到时间t的0到1，绘制曲线即为求出每个t对应的位置，以此将绘制曲线转换为绘制一个个点；实际算法为在序号相邻控制点之间进行线性插值计算，并以此递归

<img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221027084608503.png" alt="image-20221027084608503" style="zoom:50%;" />

<img src="E:/Notes_GitSpace/ComputerGraphics/FromGAMES/assets/image-20221027084948760.png" alt="image-20221027084948760" style="zoom: 33%;" />

满足任意一个时刻贝塞尔曲线上的点由控制点所控制

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027085419975.png" alt="image-20221027085419975" style="zoom:50%;" />

伯恩斯坦多项式：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027085710727.png" alt="image-20221027085710727" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027085739095.png" alt="image-20221027085739095" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027085923606.png" alt="image-20221027085923606" style="zoom:50%;" />



**贝塞尔曲线的性质：**

- 起始点和终点为两个控制点
- 起始点和终点的切线
- 对控制点进行**仿射变换**后绘制出的贝塞尔曲线，和对已知贝塞尔曲线的每个顶点进行仿射变换得到的曲线相同，即对一条贝塞尔曲线进行仿射变换，不需要记录曲线上每一个点的信息，而是只需要根据控制点进行仿射变换，再用变换后的控制点进行绘制即可（注意：并不适用于其他变换，如投影就不满足此性质）
- 凸包性质：绘制出的贝塞尔曲线一定在控制点的凸包内



#### Piecewise Bezier Curves：

控制点过多的时候贝塞尔曲线就难以控制，所以可以把一条High-Order的贝塞尔曲线拆分为多条Low-Order的贝塞尔曲线（广泛运用）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027092359892.png" alt="image-20221027092359892" style="zoom:50%;" />

分段贝塞尔曲线的**连续性**：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027092553936.png" alt="image-20221027092553936" style="zoom:50%;" />

$c^0$连续：第一段的终止点=第二段的起始点

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027092640433.png" alt="image-20221027092640433" style="zoom:50%;" />

$c^1$连续：切线连续（一阶导数的连续）——共线、等长

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027092828762.png" alt="image-20221027092828762" style="zoom:50%;" />



####  Other Types of Splines

**Spline(样条线):**

由给定的控制点形成的，并且满足一定程度的连续性的一条连续的曲线

简单总结：一条可控的曲线



**B-splines(basis splines):**

- 即为基函数样条线
- 相当于是贝塞尔曲线的扩展
- 满足贝塞尔曲线的所有重要属性，并且具有局部性（分段贝塞尔也具有局部性）



###  Surfaces

#### Bezier Surface

贝塞尔曲线扩展到贝塞尔曲面

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027094136397.png" alt="image-20221027094136397" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027094205707.png" alt="image-20221027094205707" style="zoom:50%;" />

**一个方向上通过控制点获得的曲线的点再去作为另一个方向的控制点**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027094445344.png" alt="image-20221027094445344" style="zoom:50%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221027094500951.png" alt="image-20221027094500951" style="zoom:50%;" />



- Evaluating Bezier Surface —— separate 1D de Casteljau Algorithm

  - Evaluating Surace Position For Parameters (u,v) —— 相当于为两个方向提供t，需要二维控制

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028174742640.png" alt="image-20221028174742640" style="zoom:50%;" />

    在u方向上用控制点得到四个蓝点，再用这四个蓝点再v方向上作为控制点



### Mesh Operation: Geometry Processing

####  Overview

- Mesh subdivision(upsampling)

  增加分辨率，引入更多的三角形让细节更丰富

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028180607085.png" alt="image-20221028180607085" style="zoom:50%;" />

- Mesh simplification(downsampling)

  降低分辨率，并保持原有的表现

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028180555159.png" alt="image-20221028180555159" style="zoom:50%;" />

- Mesh regularization(same triangles)

  让三角形大小、形状相似

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028180539193.png" alt="image-20221028180539193" style="zoom:50%;" />



#### Subdivision

- Loop Subdivision

  ——**三角形网格**常见的细分方式

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028180923914.png" alt="image-20221028180923914" style="zoom:50%;" />

  - ①拆分出更多的三角形(增加顶点)

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028181009456.png" alt="image-20221028181009456" style="zoom:50%;" />

  - ②调整三角形位置(顶点位置)，让原本的表面更光滑

    把顶点分为旧的顶点和新的顶点，分别对他们进行调整

    （Loop是发明者的姓氏，算法里没有循环）

    <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028181020130.png" alt="image-20221028181020130" style="zoom:50%;" />

    - 对于新的顶点（白点）：

      根据旧的顶点位置对新的顶点进行加权平均

      <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028181436534.png" alt="image-20221028181436534" style="zoom:50%;" />

    - 对于旧顶点：

      一部分去“相信”周围旧顶点的位置的平均值，另一部分保留自己的位置

      <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028181800449.png" alt="image-20221028181800449" style="zoom:50%;" />

      degree：白点连了六条边，那么白点的degree就是6

      通过**加权**的方式来平均上述的两个“相信”程度

      解释：当一个顶点的degree越大，比如说有20，说明这个顶点几乎可以完全由周围的顶点来决定，就会更加根据周围的点来确定新的位置；当一个顶点的degree越小，小到2，说明这个顶点的位置很重要，就会更加根据自己原本的位置来决定新的位置

      

  Loop细分的效果：

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028182729446.png" alt="image-20221028182729446" style="zoom:50%;" />

  

- Catmull-Clark Subdivision(用于一般网格)

  定义**Non-quad face非四边形面**和**Extraordinary vertex奇异点**（度不等于4的点）

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028183137160.png" alt="image-20221028183137160" style="zoom:50%;" />

  每一条边都取中点，每一个面也取中点，并把他们连接起来

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028184313305.png" alt="image-20221028184313305" style="zoom:50%;" />

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028184420407.png" alt="image-20221028184420407" style="zoom:50%;" />

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028184435016.png" alt="image-20221028184435016" style="zoom:50%;" />

  位置更新：

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028184452793.png" alt="image-20221028184452793" style="zoom:50%;" />

- 总的来说，细分都是增加点的数量，然后再根据一定的规则去改变点的位置；

- Loop细分只能用于三角形面，Catmall-Clark细分可以用于任意的多边形

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028184734684.png" alt="image-20221028184734684" style="zoom:50%;" />



#### Simplification

**在保持大体形状的同时减少网格体数量**

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028185330982.png" alt="image-20221028185330982" style="zoom:50%;" />

Collapsing An Edge —— edge collapsing(边坍缩)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028190027668.png" alt="image-20221028190027668" style="zoom:50%;" />

- Quadric Error Metrics(二次误差度量) —— 知道哪些边不重要可以塌缩到一起

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028190432201.png" alt="image-20221028190432201" style="zoom:50%;" />

  左侧为求平均的方法，肉眼可见效果不佳

  二次误差度量：找到一个点，使得这个点到与他相关的所有边的**距离的平方和最小**的这个最小值

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028191802218.png" alt="image-20221028191802218" style="zoom:50%;" />

  知道哪些边不重要可以塌缩到一起：对所有边进行计算，取到二次度量误差**最小**的边

  但是每进行一次塌缩，会有一些相邻的边的位置发生更改，所以需要使用一种即可以**取最小值**，又可以**动态的以最小的代价对数据进行更新**的数据结构——优先队列/堆

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028192114101.png" alt="image-20221028192114101" style="zoom:50%;" />



## 阴影映射 Shadow Mapping

- 一种图像空间的做法

- 使用shadow mapping在生成阴影这一步时不需要知道场景的几何信息
- 但是会有走样现象
- 核心思想：如果一个点不在阴影里，那么说明这个点可以同时被**摄像机和光源**“看见”



### Process 

第一步：从光源看向场景，记录看到的任何点的深度

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028193324208.png" alt="image-20221028193324208" style="zoom:50%;" />

第二步：从摄像机看向场景

获取从摄像机看到的点 投影回 光源看向场景内相同点的方向上记录的深度（记为深度1），再获取摄像机看到的点到光源的实际深度（记为深度2），两个深度进行比较

如果深度1==深度2，说明这个点不在阴影里

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028193811163.png" alt="image-20221028193811163" style="zoom:50%;" />

如果深度1<深度2，说明这个点在阴影里

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028193827774.png" alt="image-20221028193827774" style="zoom:50%;" />



### Visualizing Shadow Mapping

- 有无shadows的对比

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028194209268.png" alt="image-20221028194209268" style="zoom:50%;" />

- 从光源看向场景

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028194232670.png" alt="image-20221028194232670" style="zoom:50%;" />

- 获得光源方向的深度值

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028194304932.png" alt="image-20221028194304932" style="zoom:50%;" />

- 两个深度进行比较

  球体受光面有瑕疵的原因：

  1. 数据精度（浮点误差）
  2. shadow map的分辨率不足

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028194357859.png" alt="image-20221028194357859" style="zoom:50%;" />



### Problems with shadow maps

- 硬阴影（只能用于点光源）
- 阴影质量依赖于shadow map的分辨率（基于图像的技术普遍存在的问题）
- 存在浮点深度值的相等比较、尺度、偏差、容忍度等问题



- 硬阴影和软阴影

  硬阴影边缘很锐利，软阴影边缘会有过渡（半影）

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20221028195746578.png" alt="image-20221028195746578" style="zoom:50%;" />

  点光源不会有软阴影，对于软阴影来说光源会有大小
