---
title: 技术美术百人计划学习笔记（图形2.5-2.6）
date: 2023-06-14 08:48:57
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: gamma矫正、LDR与HDR
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![【图形渲染】 2.5 - 2.6 Gamma矫正、LDR与HDR](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2600.png)

## Gamma矫正

### Gamma矫正

#### 颜色空间

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-78.png" alt="TA100-78" style="zoom:80%;" />

sRGB和Rec-709能表达的颜色空间范围是差不多的，他的三原色位置是差不多的。它们之间的区别就在于传递函数不同



#### 传递函数

把颜色显示到一个电子设备上时，需要把颜色转换成一个视频信号，这时候就需要一个传递函数

一个传递函数分为两部分：

- OETF：光转电传递函数，负责把场景线性光转到非线性视频信号值
- EOTF：电转光传递函数，负责把非线性视频信号值转换成显示光亮度

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-79.png" alt="TA100-79" style="zoom:80%;" />

#### Gamma矫正

Gamma矫正简单来说就是指**对线性三色值和非线性视频信号之间进行编码和解码的过程**，就比如说当相机拍摄到自然中的线性的光信号，传递到电脑中进行存储时，就需要进行一个Gamma的编码操作，获得非线性的视频信号；最后通过显示器显示时，又需要将非线性的视频信号还原回一个线性的光信号，这时就需要进行一个Gamma的解码操作，最后得到我们所见到的样子（线性的光信号）

![TA100-80](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-80.png)

Gamma矫正的实例：简单理解为就是**线性显示、非线性存储**。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-82.png" alt="TA100-82" style="zoom:80%;" />



#### 进行Gamma矫正的原因

这其实和人眼的特性有关，**人眼对暗部的变化要更加敏感一点**。如果要存储更多的有效信息，就需要更多的位置去存储暗部值，即在存储暗部时用更高精度的值去存储，而对于亮部则可以用相对精度较低的值去存储。

所以，非线性转换的目的主要是为了**优化存储空间和带宽**，传递函数能够更好地帮我们利用编码空间。



#### 韦伯定律

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-83.png" alt="TA100-83" style="zoom: 67%;" />

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-84.png" alt="TA100-84" style="zoom: 80%;" />

韦伯定律:即感觉的差别闽限随原来刺激量的变化而变化，而且表现为一定的规律性，用公式来表示，就是ΔΦ/Φ=c，其中Φ为原刺激量，ΔΦ为此时的差别闽限，c为常数，又称为韦柏率。

简单来说就是，当这次通过一定的刺激爽到了，下次要获得同样的爽度就需要更多的刺激。



#### 什么是中灰值？（环境对亮度的影响）

![TA100-85 纯灰色的条看上去左右部分明暗不同](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-85.png)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-86.png" alt="TA100-86" style="zoom:67%;" />

所以说**中灰值并非一个明确的值，而是取决于实际的视觉感受**



#### 小结

1. 人眼对暗部变化比亮部更加敏感
2. 我们目前所使用的真彩格式RGBA32，每个颜色通道只有8位用于记录信息，为了合理使用带宽和存储空间，需要进行非线性转换
3. 目前我们所普遍使用的sRGB颜色空间标准，他的传递函数gamma值为2.2



### 线性工作流

- 线性工作流就是在生产的各个环节中，正确使用gamma编码和gamma解码，使得最终得到的颜色数据与最初输入的物理数据保持一致
- 如果使用Gamma空间的贴图，在传给着色器前需要从Gamma空间转到线性空间，以确保获得正确的计算结果

如果不在线性空间下进行渲染工作：

Gamma空间中进行亮度计算时，会容易过曝：因为光线信息在经过Gamma编码后，他的值比实际上的要大

![TA100-87 Gamma空间中进行亮度计算容易过曝](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-87.png)

进行颜色混合时，也会让亮度过高，出现”黑边“

![TA100-88 颜色重叠区域出现“黑边”](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-88.png)

进行光照渲染时，如果把中灰色以0.5作为实际的物理光强进行计算的话，就会出现如下左图的情况；但是实际上中灰色的物理光强度应该是0.18

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-89.png" alt="TA100-89" style="zoom:80%;" />

### Unity中的颜色空间

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-90.png" alt="TA100-90" style="zoom:80%;" />

- 当选择Gamma Space时，Unity不会做任何处理
- 当选择Linear Space时，引擎的渲染流程在线性空间中计算。理想情况下项目使用线性空间的贴图颜色，不需要勾选sRGB选项；如果勾选了sRGB选项，unity则会通过硬件特性采样时进行线性转换。

#### 硬件支持

线性空间需要图形API的硬件支持，目前支持的平台有：

1. Windows，Mac OS x和Linux
2. Xbox One
3. PS4（新的PS5肯定支持）
4. Android
5. IOS
6. WebGL（新的WebGPU肯定支持）

#### 硬件特性支持

主要由两个硬件特性来支持：

- sRGB Frame Buffer
  - 将shader的计算结果输出到显示器前进行Gamma矫正
  - 作为纹理被读取时自动会把存储的颜色从sRGB空间转换到线性空间
  - 调用ReadPixels()、ReadBackImage()时，会直接返回sRGB空间下的颜色
  - sRGB Frame Buffer只支持每通道为8bit的格式，不支持浮点格式
  - HDR开启后会先把渲染结果绘制到浮点格式的Frame Buffer中，最后绘制到sRGB Frame Buffer上输出
- sRGB Sampler
  - 将sRGB的贴图进行线性采样的转换

使用硬件特性完成sRGB贴图的线性采样和shader计算结果的gamma矫正，比起在shader里对贴图采样和计算结果的矫正要快



### 资源导出问题

#### SP贴图导出

SP贴图导出时，线性的颜色值经过了gamma变换，颜色被提亮了，所以在unity中需要勾选sRGB选项，让他在采样时能还原回线性值

![TA100-91](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-91.png)



#### PS中相关设置

如果使用线性空间，一般来说PS可以什么都不改，导出的贴图只要勾上sRGB即可

如果调整PS的gamma值为1，导出的贴图在unity中也不需要勾选sRGB了

![TA100-92](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-92.png)

- PS对颜色管理特别精准，Unity中看到的颜色要经过显示器的gamma变换，而PS不会，PS会读取显示器的Color Profile，反向补偿回去。所以PS里使用的是一个真实的颜色值
- PS有第二个Color Profile（Document Color Profile）。通常它的默认值就是sRGB Color Profile，和显示器的Color Profile一致，颜色是被这个Color Profile压暗了，所以PS中看到的结果才和Unity的一样（通过一个灰度值来控制颜色的显示，并且这个灰度值和显示器的gamma值一致，让我们看着和unity一样）

Unity在进行半透明混合时，会先将它们转换到一个线性空间下，然后再进行混合。但是PS中，图层和图层之间做混合的时候，每个上层的图层都会读取他们的Color Profile，经过一个Gamma变换，再做混合，得到的结果就会偏暗一些。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-93.png" alt="TA100-93" style="zoom:80%;" />



### Unity中的颜色空间转换函数

Unity CG.cginc文件中封装的函数

其中，带有Exact后缀的的函数计算精确，但消耗高；不带有Exact后缀的函数是一种近似解法

```glsl
inline float GammaToLinearSpaceExact (float value)
{
    if (value <= 0.04045F)
        return value / 12.92F;
    else if (value < 1.0F)
        return pow((value + 0.055F)/1.055F, 2.4F);
    else
        return pow(value, 2.2F);
}

inline half3 GammaToLinearSpace (half3 sRGB)
{
    // Approximate version from http://chilliant.blogspot.com.au/2012/08/srgb-approximations-for-hlsl.html?m=1
    return sRGB * (sRGB * (sRGB * 0.305306011h + 0.682171111h) + 0.012522878h);

    // Precise version, useful for debugging.
    //return half3(GammaToLinearSpaceExact(sRGB.r), GammaToLinearSpaceExact(sRGB.g), GammaToLinearSpaceExact(sRGB.b));
}

inline float LinearToGammaSpaceExact (float value)
{
    if (value <= 0.0F)
        return 0.0F;
    else if (value <= 0.0031308F)
        return 12.92F * value;
    else if (value < 1.0F)
        return 1.055F * pow(value, 0.4166667F) - 0.055F;
    else
        return pow(value, 0.45454545F);
}

inline half3 LinearToGammaSpace (half3 linRGB)
{
    linRGB = max(linRGB, half3(0.h, 0.h, 0.h));
    // An almost-perfect approximation from http://chilliant.blogspot.com.au/2012/08/srgb-approximations-for-hlsl.html?m=1
    return max(1.055h * pow(linRGB, 0.416666667h) - 0.055h, 0.h);

    // Exact version, useful for debugging.
    //return half3(LinearToGammaSpaceExact(linRGB.r), LinearToGammaSpaceExact(linRGB.g), LinearToGammaSpaceExact(linRGB.b));
}
```





## LDR与HDR

### 基本概念

- HDR：High Dynamic Range
- LDR：Low Dynamic Range
- 动态范围 = 最高亮度 / 最低亮度

将场景中高动态范围的光线信息，转换到显示设备上可显示出来的低动态范围的视频信息，这一过程被称为Tone Mapping

Tips：电脑显示器的最高亮度是由之前经验积累下来的一个，让人眼舒适的，相对统一的亮度。

#### LDR

- 通常LDR的精度范围是8位（2的8次方）

- 0-1的单通道

- 常用的存储格式为jpg、png、tga

- 拾色器、一般的图片，电脑屏幕都是LDR

  <img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-1.png" style="zoom:50%;" />

#### HDR

- 远高于8位的精度
- 单通道的最大值可超过1
- 常用HDR图片存储格式有hdr、tif、exr、raw
- HDRI、真实世界的光都是HRD

#### 相机记录信息的过程

- 首先，将曝光值进行计算，将曝光值重新映射回相机能够感应的范围内（受到光圈、快门时间，以及传感器灵敏度等的影响）
- 再把值进行输出（线性），存储进数码相机的格式内
- 之后会经过一个线性变换（白平衡、色彩矫正、色调验色以及伽马矫正等这一系列过程），最终会得到一张LUT图
- 不同相机厂商LUT图的格式是不同的



### 为什么需要HDR

LDR是对现实颜色进行压缩并且呈现出来，具有一定局限性，在进行后期效果调整和后续加工会因为颜色精度不够而难以进行

HDR有着更好的色彩，更高的动态范围和更丰富的细节，并且能够有效地防止画面过曝，亮度值超过1的颜色也能很好的表现出来。使得像素光亮度变得正常，视觉传达更加真实。

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-2.png)

只有HDR才有超过1的数值，才会有Bloom的效果——高质量的Bloom才能更好体现画面的渲染品质

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-3.png)



### 在Unity中设置HDR

#### Camera的HDR

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-4.png)

Unity Camera基本参数中有HDR，如果开启的话，Unity会将场景渲染到HDR图像缓冲区，下一步进行后处理，在Tone mapping过程中会把HDR转换成LDR，再将LDR图像（R8G8B8）发送给显示器

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Untitled.jpeg" alt="TA100-98" style="zoom:50%;" />

#### Lightmap的HDR

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-6.png)

将Lightmap编码质量设置为High Quality，将会为Lightmap开启HDR光照贴图的支持

#### 拾色器的HDR

- 使用Intensity滑动条可以调整颜色的强度
- 滑动条每增加1，提供的光亮增加一倍

![TA100A2600-7](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-7.png)

#### 优缺点

优点：

- 画面中亮度超过1的部分不会被截为1，增加亮部的细节并减少曝光
- 减少画面较暗部分的色阶感（用更大的精度去存储暗部）
- 更好的支持Bloom效果

缺点：

- 渲染流程中多了Tone mapping这一步，渲染速度慢
- 占据更多显存
- 不支持硬件抗锯齿
- 低端手机不支持HDR

### Bloom

渲染出原图后，提取原图中较亮的部分，对提取出来的图片进行高斯模糊，然后再叠加到原图上

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-8.png)

Unity中的Bloom流程：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-9.png)

### HDR与Tone Mapping

#### Tone Mapping的概念

- 色调映射

- 将HDR转化为LDR

- 如果是线性映射的话效果极差

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-10.png)

- 所以一般使用曲线，把亮度区域和阴影区域向中等亮度方向压缩→S曲线

#### Tone Mapping映射曲线

![左图HDR，右图LDR（有点过曝的感觉）](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-11.png)

HDR的Tone Mapping映射曲线会**把暗部压得更低一些，亮部也往回缩一点，让整体效果看起来更加真实**

##### ACES映射曲线

- Academy Color Encoding System学院颜色编码系统
- 最流行、最被广泛使用的Tonemapping映射曲线
- 效果：对比度提高，很好地保留暗处和亮处的细节

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-12.png)

##### 其他类型的映射曲线

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100A2600-13.png)

### LUT

Look-Up Table，简单来说就是滤镜

与Tone mapping不同的是，LUT是对LDR图进行处理，而Tone mapping则是处理HDR图

可以调整RGB三个通达的LUT被称为3D LUT，格式有下图所示的那几种

![TA100-107](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Untitled 12.png)

Tips：可以在PS中调整LUT，导出的LUT作为滤镜去调整画面（相当于整个画面的滤镜）

LUT的效果比较：

![TA100-108 出处： [UE4 画质增强Lut使用方法_ue4 lut_Deveuper的博客-CSDN博客](https://blog.csdn.net/qq_21153225/article/details/88875382)](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100-108.png)
